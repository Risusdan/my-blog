+++
date = '2025-12-16T23:44:36+08:00'
draft = false
title = '[C++] Rule of Five（五法則）'
categories = ['C++ Notes']
tags = ['C++']
+++

# 目錄

- [目錄](#目錄)
  - [什麼是 Rule of Five？](#什麼是-rule-of-five)
  - [為什麼需要移動語意？](#為什麼需要移動語意)
    - [沒有移動語意的問題](#沒有移動語意的問題)
    - [問題分析](#問題分析)
  - [移動語意：從複製到轉移](#移動語意從複製到轉移)
    - [右值參考（Rvalue Reference）](#右值參考rvalue-reference)
  - [Rule of Five 的完整實作](#rule-of-five-的完整實作)
    - [執行結果](#執行結果)
  - [移動操作的核心概念](#移動操作的核心概念)
    - [移動建構子](#移動建構子)
    - [為什麼要加 noexcept？](#為什麼要加-noexcept)
  - [std::move 的作用](#stdmove-的作用)
    - [使用 std::move 的時機](#使用-stdmove-的時機)
    - [std::move 的陷阱](#stdmove-的陷阱)
  - [移動建構子 vs 複製建構子的選擇](#移動建構子-vs-複製建構子的選擇)
  - [Rule of Five 的實作選項](#rule-of-five-的實作選項)
    - [選項 1：全部自定義](#選項-1全部自定義)
    - [選項 2：使用 =default](#選項-2使用-default)
    - [選項 3：只定義需要的](#選項-3只定義需要的)
  - [移動語意的效能優勢](#移動語意的效能優勢)
  - [何時需要 Rule of Five？](#何時需要-rule-of-five)
    - [需要的情況（同 Rule of Three）](#需要的情況同-rule-of-three)
    - [不需要的情況](#不需要的情況)
  - [實務建議](#實務建議)
    - [DO（應該做的）](#do應該做的)
    - [DON'T（不應該做的）](#dont不應該做的)
  - [Rule of Five vs Rule of Zero](#rule-of-five-vs-rule-of-zero)
    - [Rule of Five：手動管理](#rule-of-five手動管理)
    - [Rule of Zero：自動管理](#rule-of-zero自動管理)
  - [小結](#小結)

## 什麼是 Rule of Five？

**Rule of Five（五法則）** 是 C++11 引入**移動語意（Move Semantics）** 後，對 Rule of Three 的擴展。它指出：

> **如果一個類別需要自定義以下五者之一，那麼它很可能需要明確定義所有五個：**
> 1. **解構子（Destructor）**
> 2. **複製建構子（Copy Constructor）**
> 3. **複製賦值運算子（Copy Assignment Operator）**
> 4. **移動建構子（Move Constructor）**  ← C++11 新增
> 5. **移動賦值運算子（Move Assignment Operator）**  ← C++11 新增

Rule of Five 的核心在於：**當你管理資源時，除了要正確處理複製操作（Rule of Three），還應該提供高效的移動操作。**

## 為什麼需要移動語意？

在 C++11 之前，所有的值傳遞都涉及複製，即使是臨時物件也需要完整的深複製，這會造成不必要的效能開銷。

### 沒有移動語意的問題

```cpp
#include <iostream>
#include <cstring>

class OldString {
private:
    char* data;
    size_t length;

public:
    // 建構子
    OldString(const char* str = "") {
        length = strlen(str);
        data = new char[length + 1];
        strcpy(data, str);
        std::cout << "建構: " << data << "\n";
    }

    // 解構子
    ~OldString() {
        std::cout << "解構: " << data << "\n";
        delete[] data;
    }

    // 複製建構子
    OldString(const OldString& other) {
        length = other.length;
        data = new char[length + 1];
        strcpy(data, other.data);
        std::cout << "複製建構: " << data << " (昂貴的深複製！)\n";
    }

    // 複製賦值運算子
    OldString& operator=(const OldString& other) {
        if (this != &other) {
            delete[] data;
            length = other.length;
            data = new char[length + 1];
            strcpy(data, other.data);
            std::cout << "複製賦值: " << data << " (昂貴的深複製！)\n";
        }
        return *this;
    }
};

// 回傳臨時物件
OldString createString() {
    OldString temp("Temporary String");
    return temp;  // 回傳時會複製！
}

int main() {
    std::cout << "--- 建立臨時物件 ---\n";
    OldString str = createString();  // 又一次複製！

    std::cout << "\n--- 程式結束 ---\n";
    return 0;
}
```

### 問題分析

```
createString() 內部：
1. 建構 temp
2. 複製 temp 到回傳值（深複製！）
3. 解構 temp

main() 內部：
4. 複製回傳值到 str（又一次深複製！）
5. 解構回傳值

結果：為了一個字串，進行了 2 次深複製！
```

## 移動語意：從複製到轉移

移動語意的核心概念是：**不複製資源，而是轉移資源的所有權。**

### 右值參考（Rvalue Reference）

C++11 引入了右值參考 `&&`，用於識別臨時物件：

```cpp
void process(const MyClass& obj);  // 左值參考：接受持久物件
void process(MyClass&& obj);        // 右值參考：接受臨時物件
```

- **左值（Lvalue）**：有名字的物件，存在於記憶體中，可以取地址
- **右值（Rvalue）**：臨時物件，即將被銷毀，無法取地址

```cpp
int x = 10;           // x 是左值
int y = x + 5;        // x + 5 是右值（臨時結果）

MyClass obj;          // obj 是左值
MyClass temp();       // temp() 的回傳值是右值
```

## Rule of Five 的完整實作

```cpp
#include <iostream>
#include <cstring>
#include <utility>  // for std::move

class ModernString {
private:
    char* data;
    size_t length;

public:
    // 建構子
    ModernString(const char* str = "") {
        length = strlen(str);
        data = new char[length + 1];
        strcpy(data, str);
        std::cout << "建構: " << data << " (位址: " << (void*)data << ")\n";
    }

    // 1. 解構子
    ~ModernString() {
        std::cout << "解構: ";
        if (data) {
            std::cout << data << " (位址: " << (void*)data << ")";
        } else {
            std::cout << "(nullptr)";
        }
        std::cout << "\n";
        delete[] data;
    }

    // 2. 複製建構子（深複製）
    ModernString(const ModernString& other) {
        length = other.length;
        data = new char[length + 1];
        strcpy(data, other.data);
        std::cout << "複製建構: " << data << " (深複製)\n";
    }

    // 3. 複製賦值運算子
    ModernString& operator=(const ModernString& other) {
        std::cout << "複製賦值: " << other.data << " (深複製)\n";
        if (this != &other) {
            delete[] data;
            length = other.length;
            data = new char[length + 1];
            strcpy(data, other.data);
        }
        return *this;
    }

    // 4. 移動建構子（轉移所有權）
    ModernString(ModernString&& other) noexcept {
        std::cout << "移動建構: " << other.data << " (轉移所有權)\n";

        // 轉移資源
        data = other.data;
        length = other.length;

        // 清空來源物件
        other.data = nullptr;
        other.length = 0;
    }

    // 5. 移動賦值運算子（轉移所有權）
    ModernString& operator=(ModernString&& other) noexcept {
        std::cout << "移動賦值: " << other.data << " (轉移所有權)\n";

        // 檢查自我賦值
        if (this != &other) {
            // 釋放現有資源
            delete[] data;

            // 轉移資源
            data = other.data;
            length = other.length;

            // 清空來源物件
            other.data = nullptr;
            other.length = 0;
        }
        return *this;
    }

    void print() const {
        if (data) {
            std::cout << "內容: " << data << "\n";
        } else {
            std::cout << "內容: (已移動)\n";
        }
    }
};

// 回傳臨時物件
ModernString createString() {
    ModernString temp("Temporary String");
    return temp;  // 自動使用移動建構子！
}

int main() {
    std::cout << "--- 1. 從函式回傳值（自動移動）---\n";
    ModernString str1 = createString();
    str1.print();

    std::cout << "\n--- 2. 複製建構（左值）---\n";
    ModernString str2 = str1;  // str1 是左值，使用複製建構子
    str2.print();

    std::cout << "\n--- 3. 移動建構（使用 std::move）---\n";
    ModernString str3 = std::move(str2);  // 明確告訴編譯器：可以移動
    str3.print();
    str2.print();  // str2 已經被移動，內容為空

    std::cout << "\n--- 4. 移動賦值 ---\n";
    ModernString str4("Another String");
    str4 = std::move(str3);  // 移動賦值
    str4.print();
    str3.print();

    std::cout << "\n--- 程式結束 ---\n";
    return 0;
}
```

### 執行結果

```sh
--- 1. 從函式回傳值（自動移動）---
建構: Temporary String (位址: 0x1234)
移動建構: Temporary String (轉移所有權)
解構: (nullptr)
內容: Temporary String

--- 2. 複製建構（左值）---
複製建構: Temporary String (深複製)
內容: Temporary String

--- 3. 移動建構（使用 std::move）---
移動建構: Temporary String (轉移所有權)
內容: Temporary String
內容: (已移動)

--- 4. 移動賦值 ---
建構: Another String (位址: 0x5678)
移動賦值: Temporary String (轉移所有權)
內容: Temporary String
內容: (已移動)

--- 程式結束 ---
解構: Temporary String (位址: 0x1234)
解構: (nullptr)
解構: Another String (位址: 0x5678)
解構: (nullptr)
```

## 移動操作的核心概念

### 移動建構子

```cpp
ModernString(ModernString&& other) noexcept {
    // 1. 偷取來源物件的資源
    data = other.data;
    length = other.length;

    // 2. 將來源物件設為有效但未定義的狀態
    other.data = nullptr;
    other.length = 0;
}
```

**記憶體變化：**

```
移動前：
source.data → [H][e][l][l][o][\0]
target.data → (未初始化)

移動後：
source.data → nullptr         ← 不再擁有資源
target.data → [H][e][l][l][o][\0]  ← 接管資源
```

### 為什麼要加 noexcept？

移動操作應該標記為 `noexcept`，原因有：

1. **效能優化**：STL 容器在擴展時，如果移動建構子有 `noexcept`，會優先使用移動；否則為了例外安全，會使用複製
2. **例外安全**：移動操作通常只是指標交換，不應該拋出例外
3. **語意正確**：移動失敗會導致資源處於不一致狀態

```cpp
std::vector<ModernString> vec;
vec.push_back(ModernString("Test"));

// 如果移動建構子有 noexcept：
// → vector 擴展時使用移動，效率高

// 如果移動建構子沒有 noexcept：
// → vector 擴展時使用複製，效率低
```

## std::move 的作用

`std::move` 並不真的「移動」任何東西，它只是**將左值轉換為右值參考**，告訴編譯器「這個物件可以被移動」。

```cpp
ModernString str1("Hello");

// 錯誤理解：std::move 會移動 str1
ModernString str2 = std::move(str1);

// 正確理解：
// 1. std::move(str1) 將 str1 轉換為右值參考
// 2. 編譯器看到右值參考，呼叫移動建構子
// 3. 移動建構子真正執行移動操作
```

### 使用 std::move 的時機

```cpp
// 1. 明確表示不再使用某個物件
ModernString temp("Data");
ModernString permanent = std::move(temp);
// temp 不應該再被使用（雖然還能用，但內容已被掏空）

// 2. 從容器中移出元素
std::vector<ModernString> vec;
// ...
ModernString extracted = std::move(vec[0]);

// 3. 完美轉發（進階用法）
template<typename T>
void wrapper(T&& arg) {
    process(std::forward<T>(arg));
}
```

### std::move 的陷阱

```cpp
ModernString str1("Hello");
ModernString str2 = std::move(str1);

// 陷阱 1：str1 仍然是有效物件，但內容未定義
str1.print();  // 可能輸出: (已移動)

// 陷阱 2：不要在移動後繼續使用物件（除了賦值或解構）
// str1.someMethod();  // 危險！未定義行為

// 陷阱 3：const 物件無法被移動
const ModernString str3("World");
ModernString str4 = std::move(str3);  // 呼叫複製建構子！不是移動！
```

## 移動建構子 vs 複製建構子的選擇

編譯器如何決定呼叫哪個建構子？

```cpp
ModernString str1("Hello");

// 1. 複製建構（左值）
ModernString str2 = str1; // str1 是左值 → 複製建構子

// 2. 移動建構（右值）
ModernString str3 = ModernString("World"); // 臨時物件 → 移動建構子

// 3. 明確移動（左值 → 右值）
ModernString str4 = std::move(str1); // std::move → 移動建構子

// 4. 從函式回傳（自動移動）
ModernString str5 = createString(); // 回傳值 → 移動建構子
```

## Rule of Five 的實作選項

### 選項 1：全部自定義

如同上面的 `ModernString` 範例，完整實作所有五個函式。

**適用情況：**
- 需要細緻控制資源管理
- 有特殊的效能需求
- 複雜的自定義資源

### 選項 2：使用 =default

讓編譯器自動產生移動操作：

```cpp
class MyClass {
public:
    MyClass(const MyClass&) = default;
    MyClass& operator=(const MyClass&) = default;
    MyClass(MyClass&&) = default;  // 編譯器產生的版本通常就夠了
    MyClass& operator=(MyClass&&) = default;
    ~MyClass() = default;
};
```

### 選項 3：只定義需要的

如果只需要禁止某些操作：

```cpp
class NonMovable {
public:
    NonMovable() = default;
    ~NonMovable() = default;

    // 允許複製
    NonMovable(const NonMovable&) = default;
    NonMovable& operator=(const NonMovable&) = default;

    // 禁止移動
    NonMovable(NonMovable&&) = delete;
    NonMovable& operator=(NonMovable&&) = delete;
};
```

## 移動語意的效能優勢

讓我們比較有無移動語意的效能差異：

```cpp
#include <vector>
#include <chrono>

// 測量時間
template<typename Func>
void benchmark(const char* name, Func func) {
    auto start = std::chrono::high_resolution_clock::now();
    func();
    auto end = std::chrono::high_resolution_clock::now();
    auto duration = std::chrono::duration_cast<std::chrono::milliseconds>(end - start);
    std::cout << name << ": " << duration.count() << " ms\n";
}

int main() {
    const int SIZE = 1000000;

    // 有移動語意
    benchmark("使用移動語意", [&]() {
        std::vector<ModernString> vec;
        for (int i = 0; i < SIZE; ++i) {
            vec.push_back(ModernString("String"));  // 移動建構
        }
    });

    // 只有複製（假設沒有移動建構子）
    benchmark("只有複製", [&]() {
        std::vector<OldString> vec;
        for (int i = 0; i < SIZE; ++i) {
            vec.push_back(OldString("String"));  // 複製建構
        }
    });

    return 0;
}
```

**典型結果：**
```
使用移動語意: 45 ms
只有複製: 1250 ms

效能提升: ~27 倍！
```

## 何時需要 Rule of Five？

### 需要的情況（同 Rule of Three）

1. **管理動態記憶體**
2. **管理系統資源（檔案、網路、鎖等）**
3. **需要高效的移動操作**

### 不需要的情況

如果你的類別：
- 只使用標準函式庫容器和智慧指標
- 不直接管理資源

則應該追求 **Rule of Zero**：

```cpp
class Modern {
    std::string name;
    std::vector<int> data;
    std::unique_ptr<Resource> resource;

    // 什麼都不用寫！
    // 編譯器自動產生的移動/複製操作已經最優化
};
```

## 實務建議

### DO（應該做的）

1. **移動操作標記 noexcept**
   ```cpp
   MyClass(MyClass&&) noexcept;
   MyClass& operator=(MyClass&&) noexcept;
   ```

2. **移動後的物件必須處於有效狀態**
   ```cpp
   MyClass(MyClass&& other) noexcept {
       data = other.data;
       other.data = nullptr;  // 確保來源物件可安全解構
   }
   ```

3. **優先使用 std::move 轉移局部物件**
   ```cpp
   std::vector<MyClass> createVector() {
       std::vector<MyClass> result;
       // ... 填充 result
       return result;  // 編譯器自動移動，不需要 std::move
   }
   ```

4. **在容器操作中善用移動**
   ```cpp
   std::vector<MyClass> vec;
   MyClass obj("Data");
   vec.push_back(std::move(obj));  // 移動而非複製
   ```

### DON'T（不應該做的）

1. **不要移動 const 物件**
   ```cpp
   const MyClass obj;
   MyClass other = std::move(obj);  // 無效！呼叫複製建構子
   ```

2. **不要在 return 語句中使用 std::move**
   ```cpp
   // 錯誤
   MyClass createObject() {
       MyClass obj;
       return std::move(obj);  // 錯誤！阻礙 RVO（Return Value Optimization）
   }

   // 正確
   MyClass createObject() {
       MyClass obj;
       return obj;  // 編譯器自動優化（RVO 或移動）
   }
   ```

3. **不要假設移動後的物件為空**
   ```cpp
   MyClass obj1("Data");
   MyClass obj2 = std::move(obj1);
   // obj1 仍然是有效物件，但狀態未定義
   // 不要假設 obj1 == nullptr 或 obj1.empty()
   ```

4. **不要遺漏 noexcept**
   ```cpp
   // 不好
   MyClass(MyClass&& other);  // STL 容器會使用複製而非移動

   // 好
   MyClass(MyClass&& other) noexcept;  // STL 容器會優先使用移動
   ```

## Rule of Five vs Rule of Zero

### Rule of Five：手動管理

```cpp
class ManualResource {
    char* data;

public:
    ~ManualResource() { delete[] data; }
    ManualResource(const ManualResource&);           // 複製建構
    ManualResource& operator=(const ManualResource&); // 複製賦值
    ManualResource(ManualResource&&) noexcept;       // 移動建構
    ManualResource& operator=(ManualResource&&) noexcept; // 移動賦值
};
```

- **優點：** 完全控制
- **缺點：** 容易出錯，維護成本高

### Rule of Zero：自動管理

```cpp
class AutoResource {
    std::unique_ptr<char[]> data;  // 智慧指標自動管理一切

    // 不需要寫任何特殊成員函式！
};
```

- **優點：** 簡潔、不易出錯、自動最優化
- **缺點：** 對特殊需求的控制較少

## 小結

**Rule of Five 的核心概念：**

1. **Rule of Three + 移動語意 = Rule of Five**
   - 解構子、複製建構子、複製賦值運算子（來自 Rule of Three）
   - 移動建構子、移動賦值運算子（C++11 新增）

2. **移動語意的本質**：
   - 轉移資源所有權，而非複製
   - 使用右值參考 `&&` 識別臨時物件
   - 透過 `std::move` 明確表達移動意圖

3. **實作要點**：
   - 移動操作必須標記 `noexcept`
   - 移動後的物件必須處於有效狀態
   - 檢查自我賦值（移動賦值運算子）

4. **現代最佳實踐**：
   - 優先追求 **Rule of Zero**
   - 使用智慧指標和標準函式庫容器
   - 只在必要時實作 Rule of Five

5. **效能優勢**：
   - 避免不必要的深複製
   - 容器擴展時更高效
   - 函式回傳大物件時無額外開銷

**記住：在現代 C++ 中，優先使用智慧指標和 RAII，讓編譯器自動處理資源管理。只有在真正需要時，才手動實作 Rule of Five！**
