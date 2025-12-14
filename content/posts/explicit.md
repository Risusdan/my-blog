+++
date = '2025-12-14T10:17:22+08:00'
draft = false
title = '[C++] explicit: Avoiding the pitfalls of implicit conversion'
categories = ['C++ Notes']
tags = ['C++']
+++

# [C++] explicit: Avoiding the pitfalls of implicit conversion

## 前言

在 C++ 程式設計中，型別安全（Type Safety）是一個非常重要的概念。編譯器會在某些情況下自動進行型別轉換，這種「貼心」的功能有時候反而會造成難以察覺的 bug。`explicit` 關鍵字就是為了解決這個問題而存在的，它可以防止編譯器進行不必要的隱式轉換（Implicit Conversion），讓程式碼的意圖更加明確。

本文將介紹 `explicit` 關鍵字的使用時機、不使用它可能遇到的問題，以及如何正確運用它來提升程式碼品質。

## 什麼是 explicit

`explicit` 是 C++ 的關鍵字，用來修飾建構子（Constructor）或轉換運算子（Conversion Operator），目的是**禁止編譯器進行隱式型別轉換**。

### C++98/03：單參數建構子

在 C++98/03 中，`explicit` 主要用於單參數建構子（或多參數但其他參數都有預設值的建構子），防止編譯器自動使用該建構子進行隱式轉換。

```cpp
class MyString {
public:
    // 單參數建構子，沒有 explicit：允許隱式轉換
    MyString(int size) { /* ... */ }
};

class SafeString {
public:
    // 單參數建構子，有 explicit：禁止隱式轉換
    explicit SafeString(int size) { /* ... */ }
};
```

### C++11：轉換運算子

C++11 擴展了 `explicit` 的使用範圍，可以用於轉換運算子（Conversion Operator）：

```cpp
class SmartPointer {
    int* ptr;
public:
    // explicit 轉換運算子
    explicit operator bool() const {
        return ptr != nullptr;
    }
};
```

### C++20：條件式 explicit

C++20 引入了條件式 `explicit`，可以根據條件決定是否要求明確轉換：

```cpp
template<typename T>
class Optional {
public:
    // 只有在 T 可以隱式轉換時，才允許隱式轉換
    template<typename U>
    explicit(!std::is_convertible_v<U, T>)
    Optional(U&& value) : data(std::forward<U>(value)) {}

private:
    T data;
};
```

## 不用它的缺點

不使用 `explicit` 可能會導致編譯器在你不預期的時候進行隱式轉換，造成邏輯錯誤或效能問題。

### 範例 1：意外的型別轉換

```cpp
class Array {
    int* data;
    size_t size;
public:
    // 危險！沒有 explicit
    Array(size_t n) : size(n) {
        data = new int[n];
    }

    ~Array() { delete[] data; }

    size_t getSize() const { return size; }
};

void processArray(const Array& arr) {
    std::cout << "Array size: " << arr.getSize() << std::endl;
}

int main() {
    Array arr1(10);
    processArray(arr1);       // OK，傳入的是大小為 10 的陣列

    // 危險！因為 3 會被隱式轉換成 Array(3)
    processArray(3);          // 編譯通過，但這很可能不是你想要的

    // 更糟的情況：這會建立一個臨時的 Array，然後立刻銷毀
    if (arr1 == 10) {         // 10 被轉換成 Array(10)，這完全沒有意義！
        // ...
    }

    return 0;
}
```

這個例子中，`processArray(3)` 會編譯通過，因為編譯器會自動把 `3` 轉換成 `Array(3)`，創建一個臨時物件。這很可能不是程式設計師的本意，而且會造成不必要的效能開銷。

### 範例 2：bool 轉換的陷阱

```cpp
class FileHandle {
    FILE* file;
public:
    FileHandle(const char* filename) {
        file = fopen(filename, "r");
    }

    ~FileHandle() {
        if (file) fclose(file);
    }

    // 危險！沒有 explicit
    operator bool() const {
        return file != nullptr;
    }
};

int main() {
    FileHandle handle("data.txt");

    // 原本想檢查檔案是否開啟成功
    if (handle) {  // OK
        std::cout << "File opened\n";
    }

    // 但這樣也能編譯通過，卻完全沒意義
    int x = handle + 5;  // handle 被轉成 bool (0 或 1)，再轉成 int

    // 更危險的是可以這樣
    FileHandle h1("file1.txt");
    FileHandle h2("file2.txt");

    // 這會編譯通過！兩個 FileHandle 被轉成 bool 再相加
    int result = h1 + h2;  // 完全不合理的操作

    return 0;
}
```

## 有用它的優點

使用 `explicit` 可以讓編譯器幫你把關，確保型別轉換都是明確且有意圖的。

### 範例 1：避免意外轉換

```cpp
class Array {
    int* data;
    size_t size;
public:
    // 安全！有 explicit
    explicit Array(size_t n) : size(n) {
        data = new int[n];
    }

    ~Array() { delete[] data; }

    size_t getSize() const { return size; }
};

void processArray(const Array& arr) {
    std::cout << "Array size: " << arr.getSize() << std::endl;
}

int main() {
    Array arr1(10);
    processArray(arr1);       // OK

    // processArray(3);       // 編譯錯誤！不允許隱式轉換

    // 如果真的需要，必須明確轉換
    processArray(Array(3));   // OK：明確表達意圖

    return 0;
}
```

使用 `explicit` 後，`processArray(3)` 會產生編譯錯誤，這樣可以避免不小心傳入錯誤的參數。如果真的需要這樣做，必須寫成 `processArray(Array(3))`，讓意圖更明確。

### 範例 2：安全的 bool 轉換

```cpp
class FileHandle {
    FILE* file;
public:
    FileHandle(const char* filename) {
        file = fopen(filename, "r");
    }

    ~FileHandle() {
        if (file) fclose(file);
    }

    // 安全！有 explicit
    explicit operator bool() const {
        return file != nullptr;
    }
};

int main() {
    FileHandle handle("data.txt");

    // 在條件判斷中仍然可以使用（這是允許的）
    if (handle) {  // OK
        std::cout << "File opened\n";
    }

    // int x = handle + 5;  // 編譯錯誤！不允許隱式轉成 bool

    FileHandle h1("file1.txt");
    FileHandle h2("file2.txt");

    // int result = h1 + h2;  // 編譯錯誤！防止不合理的操作

    // 如果真的需要，必須明確轉換
    bool isOpen = static_cast<bool>(handle);  // OK：明確轉換

    return 0;
}
```

### 範例 3：標準函式庫的最佳實踐

C++ 標準函式庫大量使用 `explicit`，例如：

```cpp
// std::vector 的建構子
template<typename T>
class vector {
public:
    // explicit！避免 size_t 隱式轉換成 vector
    explicit vector(size_t n);
};

// std::unique_ptr 的 bool 轉換
template<typename T>
class unique_ptr {
public:
    // explicit！只能在條件判斷中使用
    explicit operator bool() const noexcept;
};

// 使用範例
std::vector<int> v1(10);     // OK
// std::vector<int> v2 = 10; // 編譯錯誤！

std::unique_ptr<int> ptr = std::make_unique<int>(42);
if (ptr) {                   // OK：條件判斷
    std::cout << *ptr;
}
// int x = ptr;              // 編譯錯誤！
```

## 小結

`explicit` 關鍵字是 C++ 中非常重要的型別安全機制：

### 使用時機

1. **單參數建構子**：除非你明確希望允許隱式轉換，否則都應該加上 `explicit`
2. **轉換運算子**：特別是轉換成 `bool` 的運算子，建議都加上 `explicit`
3. **多參數建構子**：如果其他參數都有預設值，也應該考慮加上 `explicit`

### 核心原則

- **明確優於隱式**：使用 `explicit` 可以讓程式碼的意圖更清楚
- **型別安全**：防止意外的型別轉換，減少 bug
- **遵循慣例**：參考標準函式庫的做法，建立良好的程式設計習慣

### 經驗法則

- 當不確定是否該使用 `explicit` 時，**預設加上它**。如果之後發現確實需要隱式轉換，再移除也不遲。這樣可以避免很多潛在的問題。
- 記住編譯器的「貼心」有時候反而會造成問題，適時地使用 `explicit` 關鍵字，讓編譯器在你需要的時候提供協助，而不是幫倒忙！
