+++
date = '2025-12-16T23:44:27+08:00'
draft = false
title = '[C++] Rule of Three（三法則）'
categories = ['C++ Notes']
tags = ['C++']
+++

# 目錄

- [目錄](#目錄)
  - [什麼是 Rule of Three？](#什麼是-rule-of-three)
  - [為什麼需要 Rule of Three？](#為什麼需要-rule-of-three)
  - [問題範例：違反 Rule of Three](#問題範例違反-rule-of-three)
    - [執行結果（可能崩潰）](#執行結果可能崩潰)
    - [問題分析](#問題分析)
  - [正確做法：遵守 Rule of Three](#正確做法遵守-rule-of-three)
    - [執行結果](#執行結果)
  - [深複製 vs 淺複製](#深複製-vs-淺複製)
    - [淺複製（Shallow Copy）](#淺複製shallow-copy)
    - [深複製（Deep Copy）](#深複製deep-copy)
  - [複製賦值運算子的實作細節](#複製賦值運算子的實作細節)
    - [1. 檢查自我賦值](#1-檢查自我賦值)
    - [2. Copy-and-Swap idiom（進階技巧）](#2-copy-and-swap-idiom進階技巧)
  - [何時需要 Rule of Three？](#何時需要-rule-of-three)
    - [需要的情況](#需要的情況)
    - [不需要的情況](#不需要的情況)
  - [現代 C++ 的建議](#現代-c-的建議)
    - [C++11 之後：優先使用智慧指標](#c11-之後優先使用智慧指標)
    - [Rule of Zero](#rule-of-zero)
  - [實務建議](#實務建議)
    - [DO（應該做的）](#do應該做的)
    - [DON'T（不應該做的）](#dont不應該做的)
  - [小結](#小結)

## 什麼是 Rule of Three？

**Rule of Three（三法則）** 是 C++ 中一個重要的設計原則，它指出：

> **如果一個類別需要自定義以下三者之一，那麼它很可能需要自定義所有三個：**
> 1. **解構子（Destructor）**
> 2. **複製建構子（Copy Constructor）**
> 3. **複製賦值運算子（Copy Assignment Operator）**

這個原則的核心概念是：**如果一個類別需要管理資源（如動態記憶體、檔案句柄、網路連線等），就必須明確定義這三個特殊成員函式，以確保資源的正確管理。**

## 為什麼需要 Rule of Three？

當類別擁有指標成員變數或其他需要手動管理的資源時，編譯器自動產生的複製建構子和複製賦值運算子只會進行**淺複製（Shallow Copy）**，這會導致：

1. **記憶體洩漏（Memory Leak）**
2. **重複釋放（Double Free）**
3. **懸空指標（Dangling Pointer）**

## 問題範例：違反 Rule of Three

讓我們看一個沒有遵守 Rule of Three 的例子：

```cpp
#include <iostream>
#include <cstring>

class BadString {
private:
    char* data;
    size_t length;

public:
    // 建構子：分配記憶體
    BadString(const char* str = "") {
        length = strlen(str);
        data = new char[length + 1];
        strcpy(data, str);
        std::cout << "建構 BadString: " << data << "\n";
    }

    // 解構子：釋放記憶體
    ~BadString() {
        std::cout << "解構 BadString: " << data << "\n";
        delete[] data;
    }

    // 沒有定義複製建構子！
    // 沒有定義複製賦值運算子！

    void print() const {
        std::cout << "內容: " << data << "\n";
    }
};

int main() {
    BadString str1("Hello");
    str1.print();

    std::cout << "\n--- 複製建構 ---\n";
    BadString str2 = str1;  // 使用編譯器產生的複製建構子
    str2.print();

    std::cout << "\n--- 離開作用域 ---\n";
    // 程式崩潰！str1 和 str2 指向同一塊記憶體
    // 解構時會重複釋放同一塊記憶體（Double Free）
    return 0;
}
```

### 執行結果（可能崩潰）

```sh
建構 BadString: Hello
內容: Hello

--- 複製建構 ---
內容: Hello

--- 離開作用域 ---
解構 BadString: Hello
解構 BadString: Hello    ← 重複釋放！程式崩潰
*** Error in `./program': double free or corruption
```

### 問題分析

```
初始狀態 (str1):
str1.data → [H][e][l][l][o][\0]

淺複製後 (str2 = str1):
str1.data ─┐
           ├→ [H][e][l][l][o][\0]  ← 兩個指標指向同一塊記憶體！
str2.data ─┘

解構時：
1. str2 解構：delete[] 釋放記憶體
2. str1 解構：delete[] 再次釋放已釋放的記憶體！崩潰！
```

## 正確做法：遵守 Rule of Three

```cpp
#include <iostream>
#include <cstring>

class GoodString {
private:
    char* data;
    size_t length;

public:
    // 建構子
    GoodString(const char* str = "") {
        length = strlen(str);
        data = new char[length + 1];
        strcpy(data, str);
        std::cout << "建構: " << data << " (位址: " << (void*)data << ")\n";
    }

    // 1. 解構子
    ~GoodString() {
        std::cout << "解構: " << data << " (位址: " << (void*)data << ")\n";
        delete[] data;
    }

    // 2. 複製建構子（深複製）
    GoodString(const GoodString& other) {
        length = other.length;
        data = new char[length + 1];  // 分配新記憶體
        strcpy(data, other.data);      // 複製內容
        std::cout << "複製建構: " << data << " (位址: " << (void*)data << ")\n";
    }

    // 3. 複製賦值運算子
    GoodString& operator=(const GoodString& other) {
        std::cout << "複製賦值: " << other.data << "\n";

        // 檢查自我賦值
        if (this == &other) {
            return *this;
        }

        // 釋放舊資源
        delete[] data;

        // 複製新資源（深複製）
        length = other.length;
        data = new char[length + 1];
        strcpy(data, other.data);

        return *this;
    }

    void print() const {
        std::cout << "內容: " << data << " (位址: " << (void*)data << ")\n";
    }
};

int main() {
    GoodString str1("Hello");
    str1.print();

    std::cout << "\n--- 複製建構 ---\n";
    GoodString str2 = str1;  // 呼叫複製建構子
    str2.print();

    std::cout << "\n--- 複製賦值 ---\n";
    GoodString str3("World");
    str3 = str1;  // 呼叫複製賦值運算子
    str3.print();

    std::cout << "\n--- 離開作用域 ---\n";
    // 正常解構，每個物件都有自己的記憶體
    return 0;
}
```

### 執行結果

```sh
建構: Hello (位址: 0x1234)
內容: Hello (位址: 0x1234)

--- 複製建構 ---
複製建構: Hello (位址: 0x5678)  ← 新的記憶體位址
內容: Hello (位址: 0x5678)

--- 複製賦值 ---
建構: World (位址: 0x9abc)
複製賦值: Hello
內容: Hello (位址: 0x9abc)

--- 離開作用域 ---
解構: Hello (位址: 0x9abc)     ← 每個物件有自己的記憶體
解構: Hello (位址: 0x5678)
解構: Hello (位址: 0x1234)
```

## 深複製 vs 淺複製

### 淺複製（Shallow Copy）

編譯器預設產生的複製行為，只複製指標的值（位址），不複製指標指向的內容：

```cpp
// 編譯器產生的淺複製
GoodString(const GoodString& other)
    : data(other.data),      // 只複製指標！
      length(other.length)
{
}
```

```
淺複製：
str1.data → [記憶體內容]
              ↑
str2.data ────┘  兩個指標指向同一塊記憶體
```

### 深複製（Deep Copy）

手動實作的複製行為，分配新的記憶體並複製內容：

```cpp
// 手動實作的深複製
GoodString(const GoodString& other) {
    length = other.length;
    data = new char[length + 1];  // 分配新記憶體
    strcpy(data, other.data);      // 複製內容
}
```

```
深複製：
str1.data → [記憶體內容 1]
str2.data → [記憶體內容 2]  各自擁有獨立的記憶體
```

## 複製賦值運算子的實作細節

複製賦值運算子的實作需要注意幾個重點：

### 1. 檢查自我賦值

```cpp
GoodString& operator=(const GoodString& other) {
    // 必須檢查！否則會先刪除自己的資源
    if (this == &other) {
        return *this;
    }

    // ... 賦值邏輯
}
```

**為什麼需要檢查？**

```cpp
GoodString str("Hello");
str = str;  // 自我賦值

// 如果沒有檢查，會發生：
delete[] data;           // 刪除自己的資料
strcpy(data, other.data); // other.data 已經被刪除了！懸空指標！
```

### 2. Copy-and-Swap idiom（進階技巧）

一個更安全、更簡潔的實作方式：

```cpp
class GoodString {
private:
    char* data;
    size_t length;

public:
    // ... 建構子、解構子、複製建構子

    // swap 函式
    void swap(GoodString& other) noexcept {
        std::swap(data, other.data);
        std::swap(length, other.length);
    }

    // 複製賦值運算子（使用 Copy-and-Swap）
    GoodString& operator=(const GoodString& other) {
        // 建立副本（複製建構）
        GoodString temp(other);

        // 交換內容
        swap(temp);

        // temp 解構時會自動釋放舊資源
        return *this;
    }
};
```

**優點：**
- **自動處理自我賦值**：因為先建立副本
- **例外安全**：如果複製建構失敗，原物件不受影響
- **程式碼簡潔**：邏輯清晰易懂

## 何時需要 Rule of Three？

### 需要的情況

1. **動態記憶體管理**
   ```cpp
   class DynamicArray {
       int* data;  // 需要 delete[]
   };
   ```

2. **檔案句柄**
   ```cpp
   class FileHandle {
       FILE* file;  // 需要 fclose()
   };
   ```

3. **網路連線**
   ```cpp
   class Socket {
       int sockfd;  // 需要 close()
   };
   ```

4. **互斥鎖**
   ```cpp
   class MutexGuard {
       pthread_mutex_t* mutex;  // 需要 unlock()
   };
   ```

### 不需要的情況

如果類別只包含：
- 內建型別（int, double, etc.）
- 已經妥善管理資源的類別（std::string, std::vector, std::unique_ptr）

則**不需要**自定義這三個函式，使用編譯器預設的即可：

```cpp
class Point {
    double x, y;  // 內建型別，不需要特殊處理
    // 使用編譯器預設的複製/賦值/解構
};

class Person {
    std::string name;      // std::string 會管理自己的記憶體
    std::vector<int> ages;  // std::vector 會管理自己的記憶體
    // 使用編譯器預設的複製/賦值/解構
};
```

## 現代 C++ 的建議

### C++11 之後：優先使用智慧指標

使用 RAII（Resource Acquisition Is Initialization）和智慧指標，讓編譯器自動管理資源：

```cpp
#include <memory>
#include <cstring>

class ModernString {
private:
    std::unique_ptr<char[]> data;  // 智慧指標自動管理記憶體
    size_t length;

public:
    ModernString(const char* str = "")
        : length(strlen(str)),
          data(std::make_unique<char[]>(length + 1))
    {
        strcpy(data.get(), str);
    }

    // 不需要自定義解構子！unique_ptr 會自動釋放

    // 複製建構子（仍需手動實作深複製）
    ModernString(const ModernString& other)
        : length(other.length),
          data(std::make_unique<char[]>(length + 1))
    {
        strcpy(data.get(), other.data.get());
    }

    // 複製賦值運算子
    ModernString& operator=(const ModernString& other) {
        if (this != &other) {
            length = other.length;
            data = std::make_unique<char[]>(length + 1);
            strcpy(data.get(), other.data.get());
        }
        return *this;
    }
};
```

### Rule of Zero

**Rule of Zero** 是現代 C++ 的理想目標：

> **盡可能不要自定義任何特殊成員函式，讓編譯器自動產生。**

達成方式：
- 使用 `std::unique_ptr`、`std::shared_ptr` 管理記憶體
- 使用 `std::string` 代替 `char*`
- 使用 `std::vector` 代替 raw arrays
- 使用 RAII 包裝器類別管理資源

```cpp
class BestPractice {
    std::string name;
    std::vector<int> data;
    std::unique_ptr<Resource> resource;

    // 不需要定義任何特殊成員函式！
    // 編譯器自動產生的版本已經正確處理了所有資源
};
```

## 實務建議

### DO（應該做的）

1. **有疑慮時，明確定義或明確刪除**
   ```cpp
   class MyClass {
   public:
       // 明確表示意圖
       MyClass(const MyClass&) = default;
       MyClass& operator=(const MyClass&) = default;
       ~MyClass() = default;
   };
   ```

2. **無法複製的類別，明確禁止**
   ```cpp
   class NonCopyable {
   public:
       NonCopyable() = default;
       NonCopyable(const NonCopyable&) = delete;
       NonCopyable& operator=(const NonCopyable&) = delete;
   };
   ```

### DON'T（不應該做的）

1. **不要只定義其中一個或兩個**
   ```cpp
   class Bad {
       ~Bad() { delete[] data; }
       // 錯誤！沒有定義複製建構子和賦值運算子
       // 使用預設的會造成 double free
   private:
       char* data;
   };
   ```

2. **不要忘記檢查自我賦值**
   ```cpp
   Bad& operator=(const Bad& other) {
       delete[] data; // 危險！
       data = new char[size];
       // 如果 this == &other，data 已經被刪除了
   }
   ```

3. **不要在現代 C++ 中還使用 raw pointers 管理資源**
   ```cpp
   class OldStyle {
       int* data;  // 不好！應該用 std::unique_ptr<int>
   };
   ```

## 小結

**Rule of Three 的核心概念：**

1. **定義時機**：需要管理資源時（動態記憶體、檔案、連線等）
2. **三者缺一不可**：
   - 解構子：釋放資源
   - 複製建構子：深複製資源
   - 複製賦值運算子：先釋放舊資源，再深複製新資源
3. **現代做法**：
   - 優先使用智慧指標和 RAII
   - 追求 Rule of Zero
   - 無法複製的類別使用 `=delete`
4. **安全檢查**：
   - 賦值運算子必須檢查自我賦值
   - 考慮使用 Copy-and-Swap idiom

**記住：如果你需要自定義解構子，很可能也需要自定義複製建構子和複製賦值運算子！**
