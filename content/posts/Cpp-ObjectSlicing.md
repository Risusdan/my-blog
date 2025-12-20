+++
date = '2025-12-14T00:00:00+08:00'
draft = false
title = '[C++] 物件切割（Object Slicing）問題'
categories = ['C++ Notes']
tags = ['C++']
+++

# 目錄

- [目錄](#目錄)
  - [什麼是物件切割？](#什麼是物件切割)
  - [問題範例](#問題範例)
    - [輸出結果](#輸出結果)
  - [發生了什麼事？](#發生了什麼事)
    - [1. 資料遺失](#1-資料遺失)
    - [2. 多型失效](#2-多型失效)
    - [3. 記憶體佈局](#3-記憶體佈局)
  - [容易發生切割的場景](#容易發生切割的場景)
    - [場景 1：函式參數傳遞（最常見）](#場景-1函式參數傳遞最常見)
    - [場景 2：容器儲存](#場景-2容器儲存)
    - [場景 3：賦值操作](#場景-3賦值操作)
    - [場景 4：函式回傳值](#場景-4函式回傳值)
  - [正確做法：避免切割](#正確做法避免切割)
    - [方法 1：使用參考（Reference）](#方法-1使用參考reference)
    - [方法 2：使用指標（Pointer）](#方法-2使用指標pointer)
    - [方法 3：容器儲存指標](#方法-3容器儲存指標)
    - [方法 4：回傳指標或參考](#方法-4回傳指標或參考)
  - [為什麼這在 Effective C++ 中很重要？](#為什麼這在-effective-c-中很重要)
    - [1. 避免切割問題](#1-避免切割問題)
    - [2. 保持多型行為](#2-保持多型行為)
    - [3. 避免昂貴的複製操作](#3-避免昂貴的複製操作)
  - [實務建議](#實務建議)
    - [DO（應該做的）](#do應該做的)
    - [DON'T（不應該做的）](#dont不應該做的)
  - [偵測切割問題](#偵測切割問題)
    - [編譯期偵測（C++11 起）](#編譯期偵測c11-起)
  - [小結](#小結)

## 什麼是物件切割？

**物件切割（Object Slicing）** 是 C++ 中一個常見且危險的問題，發生在**當派生類（derived class）物件被賦值給基礎類（base class）物件時，派生類特有的部分會被「切掉」**。

簡單來說，當你用 pass-by-value 的方式將子類物件傳給父類物件時，子類獨有的資料和行為都會消失。

## 問題範例

讓我們看一個具體的例子：

```cpp
#include <iostream>
#include <string>

class Animal {
public:
    std::string name;

    virtual void makeSound() const {
        std::cout << "Some generic animal sound\n";
    }

    virtual ~Animal() = default;
};

class Dog : public Animal {
public:
    std::string breed;  // Dog 特有的成員變數

    virtual void makeSound() const override {
        std::cout << "Woof! Woof!\n";
    }
};

int main() {
    Dog myDog;
    myDog.name = "Buddy";
    myDog.breed = "Golden Retriever";

    std::cout << "原始 Dog 物件:\n";
    std::cout << "Name: " << myDog.name << "\n";
    std::cout << "Breed: " << myDog.breed << "\n";
    myDog.makeSound();  // 輸出: Woof! Woof!

    std::cout << "\n--- 物件切割發生 ---\n\n";

    // 物件切割發生了！
    Animal animal = myDog;  // pass-by-value，呼叫 Animal 的複製建構子

    std::cout << "切割後的 Animal 物件:\n";
    std::cout << "Name: " << animal.name << "\n";
    // std::cout << "Breed: " << animal.breed << "\n";  // 編譯錯誤！breed 不存在
    animal.makeSound();  // 輸出: Some generic animal sound
                        // 不是 "Woof!"！多型失效了！

    return 0;
}
```

### 輸出結果

```sh
原始 Dog 物件:
Name: Buddy
Breed: Golden Retriever
Woof! Woof!

--- 物件切割發生 ---

切割後的 Animal 物件:
Name: Buddy
Some generic animal sound
```

## 發生了什麼事？

當執行 `Animal animal = myDog;` 時，發生了以下情況：

### 1. 資料遺失

- `Dog` 的 `breed` 成員變數**完全消失**了
- `animal` 物件只包含 `Animal` 部分的資料
- 這是因為複製建構時，只複製了基礎類的大小

### 2. 多型失效

- 雖然 `makeSound()` 是虛擬函式
- 但因為 `animal` 是 `Animal` **型別**（不是指標或參考）
- 所以呼叫的是 `Animal::makeSound()`，而非 `Dog::makeSound()`
- 虛擬函式的多型機制失效

### 3. 記憶體佈局

```
myDog 的記憶體佈局:
┌─────────────────┐
│ vptr (virtual)  │  ← 指向 Dog 的 vtable
├─────────────────┤
│ name (Animal)   │
├─────────────────┤
│ breed (Dog)     │  ← Dog 特有的資料
└─────────────────┘

animal 的記憶體佈局（切割後）:
┌─────────────────┐
│ vptr (virtual)  │  ← 指向 Animal 的 vtable（不是 Dog 的！）
├─────────────────┤
│ name (Animal)   │
└─────────────────┘
                     ← breed 被切掉了！
```

## 容易發生切割的場景

### 場景 1：函式參數傳遞（最常見）

```cpp
// 危險：會發生切割
void processAnimal(Animal animal) {  // pass-by-value
    animal.makeSound();  // 總是呼叫 Animal::makeSound()
}

int main() {
    Dog myDog;
    processAnimal(myDog);  // 切割發生！傳入時就切掉了
}
```

### 場景 2：容器儲存

```cpp
// 危險：容器儲存物件本身
std::vector<Animal> animals;

Dog dog;
dog.name = "Buddy";
dog.breed = "Golden Retriever";

animals.push_back(dog);  // 切割！breed 資料遺失
animals[0].makeSound();  // 輸出: Some generic animal sound
```

### 場景 3：賦值操作

```cpp
Dog myDog;
Animal animal;

animal = myDog;  // 切割！只賦值 Animal 部分
```

### 場景 4：函式回傳值

```cpp
// 危險：回傳值會被切割
Animal createDog() {
    Dog dog;
    dog.name = "Max";
    dog.breed = "Labrador";
    return dog;  // 回傳時發生切割！
}

int main() {
    Animal animal = createDog();
    animal.makeSound();  // 輸出: Some generic animal sound
}
```

## 正確做法：避免切割

### 方法 1：使用參考（Reference）

```cpp
// 正確：使用參考，不會切割
void processAnimal(const Animal& animal) {  // pass-by-reference-to-const
    animal.makeSound();  // 會呼叫實際物件的 makeSound()（多型正常）
}

int main() {
    Dog myDog;
    processAnimal(myDog);  // 輸出: Woof! Woof!（多型運作正常）
}
```

**為什麼參考不會切割？**
- 參考只是別名，指向原始的完整物件
- 不會產生新的物件副本
- 保留完整的多型資訊（vptr 指向正確的 vtable）

### 方法 2：使用指標（Pointer）

```cpp
// 正確：使用指標，不會切割
void processAnimal(const Animal* animal) {
    if (animal) {
        animal->makeSound();  // 多型正常運作
    }
}

int main() {
    Dog myDog;
    processAnimal(&myDog);  // 輸出: Woof! Woof!
}
```

### 方法 3：容器儲存指標

```cpp
// 正確：儲存指標（使用智慧指標更好）
std::vector<std::unique_ptr<Animal>> animals;

animals.push_back(std::make_unique<Dog>());
animals[0]->name = "Buddy";

// 需要向下轉型才能存取 breed
if (Dog* dog = dynamic_cast<Dog*>(animals[0].get())) {
    dog->breed = "Golden Retriever";
}

animals[0]->makeSound();  // 輸出: Woof! Woof!（多型正常）
```

### 方法 4：回傳指標或參考

```cpp
// 正確：回傳指標
std::unique_ptr<Animal> createDog() {
    auto dog = std::make_unique<Dog>();
    dog->name = "Max";
    // 注意：需要向下轉型才能設定 breed
    static_cast<Dog*>(dog.get())->breed = "Labrador";
    return dog;
}

int main() {
    auto animal = createDog();
    animal->makeSound();  // 輸出: Woof! Woof!
}
```

## 為什麼這在 Effective C++ 中很重要？

這就是 **Effective C++ 建議「以 pass-by-reference-to-const 取代 pass-by-value」** 的原因之一：

### 1. 避免切割問題
```cpp
// 在 Object-Oriented C++ 子語言中
void processWidget(const Widget& w);  // 推薦！不會切割
```

### 2. 保持多型行為
```cpp
void displayShape(const Shape& shape) {
    shape.draw();  // 會呼叫正確的 draw() 版本
}

Circle circle;
Rectangle rect;

displayShape(circle);  // 呼叫 Circle::draw()
displayShape(rect);    // 呼叫 Rectangle::draw()
```

### 3. 避免昂貴的複製操作
```cpp
class HugeObject {
    std::vector<int> data(1000000);  // 很大的資料
    // ...
};

// 不好：複製 1000000 個 int
void process(HugeObject obj);

// 好：只傳遞參考（一個指標的大小）
void process(const HugeObject& obj);
```

## 實務建議

### DO（應該做的）

1. **預設使用 const 參考傳遞自定義類別**
   ```cpp
   void func(const MyClass& obj);
   ```

2. **容器儲存多型物件時使用智慧指標**
   ```cpp
   std::vector<std::unique_ptr<Base>> objects;
   ```

3. **理解何時該用指標、參考或值**
   - 內建型別（int, double）：pass-by-value
   - 小型物件且不需多型：pass-by-value
   - 大型物件或需要多型：pass-by-reference 或 pointer

### DON'T（不應該做的）

1. **不要用 pass-by-value 傳遞多型物件**
   ```cpp
   void bad(Base obj);  // 會切割
   ```

2. **不要在容器中直接儲存多型物件**
   ```cpp
   std::vector<Base> vec;  // 會切割
   ```

3. **不要回傳多型物件本身**
   ```cpp
   Base createDerived();  // 回傳時會切割
   ```

## 偵測切割問題

### 編譯期偵測（C++11 起）

可以將基礎類的複製建構子和賦值運算子刪除，強制使用指標或參考：

```cpp
class Animal {
public:
    // 禁止複製，避免切割
    Animal(const Animal&) = delete;
    Animal& operator=(const Animal&) = delete;

    // 但允許移動（如果需要）
    Animal(Animal&&) = default;
    Animal& operator=(Animal&&) = default;

    virtual void makeSound() const = 0;
    virtual ~Animal() = default;

protected:
    Animal() = default;  // 只能被派生類呼叫
};
```

現在如果嘗試切割，會得到編譯錯誤：

```cpp
Dog myDog;
Animal animal = myDog;  // 編譯錯誤！複製建構子被刪除
```

## 小結

**物件切割的關鍵點：**

1. **發生時機**：用 pass-by-value 傳遞派生類物件給基礎類
2. **後果**：
   - 派生類特有資料遺失
   - 多型行為失效
   - 可能造成難以察覺的 bug
3. **解決方法**：
   - 使用 pass-by-reference-to-const
   - 使用指標（最好是智慧指標）
   - 考慮刪除基礎類的複製建構子
4. **適用規則**：
   - 內建型別：pass-by-value 沒問題
   - 自定義類別（尤其是多型）：pass-by-reference-to-const
