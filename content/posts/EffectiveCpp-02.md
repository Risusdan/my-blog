+++
draft = false
date = 2025-12-15T22:43:46+08:00
title = "[Effective C++] 02: Prefer consts, enums, and inlines to #defines"
categories = ['Effective C++']
tags = ['C++']
+++

# 目錄

- [目錄](#目錄)
  - [前言](#前言)
  - [核心概念](#核心概念)
  - [問題：使用 #define 的缺點](#問題使用-define-的缺點)
    - [問題 1：缺乏型別檢查](#問題-1缺乏型別檢查)
    - [問題 2：無法封裝](#問題-2無法封裝)
    - [問題 3：macro 的陷阱](#問題-3macro-的陷阱)
  - [解決方案 1：用 const 取代 #define 常數](#解決方案-1用-const-取代-define-常數)
    - [基本用法](#基本用法)
    - [解決方案 1 的特殊情況 1：定義常數指標](#解決方案-1-的特殊情況-1定義常數指標)
    - [解決方案 1 的特殊情況 2：class 專屬常數](#解決方案-1-的特殊情況-2class-專屬常數)
  - [解決方案 2：enum hack](#解決方案-2enum-hack)
  - [解決方案 3：用 inline 函式取代 macro](#解決方案-3用-inline-函式取代-macro)
    - [問題回顧](#問題回顧)
    - [解決方案：template inline 函式](#解決方案template-inline-函式)
    - [inline 函式的特性](#inline-函式的特性)
  - [現代 C++ 的補充（C++11 及之後）](#現代-c-的補充c11-及之後)
    - [constexpr（C++11）](#constexprc11)
    - [inline 變數（C++17）](#inline-變數c17)
  - [實務建議](#實務建議)
    - [取代 #define 常數的決策樹](#取代-define-常數的決策樹)
    - [取代 #define macro 的決策樹](#取代-define-macro-的決策樹)
  - [#define 的合理使用時機](#define-的合理使用時機)
    - [1. Include Guards](#1-include-guards)
    - [2. 條件編譯](#2-條件編譯)
  - [關鍵重點總結](#關鍵重點總結)
    - [為什麼要避免 #define？](#為什麼要避免-define)
    - [替代方案](#替代方案)
    - [核心原則](#核心原則)
  - [實例比較](#實例比較)
    - [Before：使用 #define](#before使用-define)
    - [After：使用 const/enum/inline](#after使用-constenuminline)
  - [結語](#結語)

## 前言

本文是 Scott Meyers 所著《Effective C++》的閱讀筆記。

## 核心概念

**寧可以編譯器替換預處理器（Prefer the compiler to the preprocessor）**

這個建議的核心在於：`#define` 不被視為語言的一部分，預處理器只是簡單的文字替換，這會導致許多問題。而使用 `const`、`enum` 和 `inline` 這些語言特性，可以獲得型別安全、除錯支援和更好的封裝性。

## 問題：使用 #define 的缺點

### 問題 1：缺乏型別檢查

```cpp
// 使用 #define
#define ASPECT_RATIO 1.653

// 預處理器會把所有 ASPECT_RATIO 替換成 1.653
double width = 10;
double height = width / ASPECT_RATIO;
```

**問題：**
- `ASPECT_RATIO` 這個名稱可能不會進入符號表（symbol table）
- 如果編譯錯誤提到 `1.653`，你可能會困惑這個數字從哪來
- 除錯器中看不到 `ASPECT_RATIO`，只看到 `1.653`
- 沒有型別檢查，可能被誤用

### 問題 2：無法封裝

```cpp
// 無法在 class 內定義 #define 常數
class GamePlayer {
private:
    #define NUM_TURNS 5  // 這不會限制在 class 內！
    int scores[NUM_TURNS];
};
```

**問題：**
- `#define` 不遵守作用域規則
- 無法創建 class 專屬的常數

### 問題 3：macro 的陷阱

```cpp
// 使用 macro 實作函式
#define CALL_WITH_MAX(a, b) f((a) > (b) ? (a) : (b))

int a = 5, b = 0;
CALL_WITH_MAX(++a, b);      // a 被累加兩次！
CALL_WITH_MAX(++a, b + 10); // a 被累加一次
```

**問題：**
- 參數可能被多次求值（multiple evaluation）
- 必須記得加上所有括號
- 不是真正的函式呼叫，無法取得地址
- 除錯困難

## 解決方案 1：用 const 取代 #define 常數

### 基本用法

```cpp
// 使用 const
const double AspectRatio = 1.653;

double width = 10;
double height = width / AspectRatio;
```

**優點：**
- `AspectRatio` 會進入符號表
- 編譯器可以看到這個名稱，除錯更容易
- 有型別檢查
- 可能產生較小的目的碼（只有一份常數，而不像用 #define 時出現多個 1.653）

### 解決方案 1 的特殊情況 1：定義常數指標

```cpp
// 定義 const char* 字串
const char* const authorName = "Scott Meyers";
// 第一個 const：指向的內容（char 型別的某個對象）不可變
// 第二個 const：指標本身不可變

// 更好的做法：使用 string
const std::string authorName("Scott Meyers");
```

### 解決方案 1 的特殊情況 2：class 專屬常數

```cpp
class GamePlayer {
private:
    // static const 整數型別可以直接在 class 內初始化
    static const int NumTurns = 5;
    int scores[NumTurns];
};

// 如果需要取得這個常數的地址，需要提供定義
// 在實作檔中：
const int GamePlayer::NumTurns;  // 定義即可，不需再給初值，因為在宣告處給過了
```

**注意事項：**
- 只有 static const 整數型別（int, char, bool 等）可以在 class 內初始化
- 其他型別需要在實作檔中定義

```cpp
class CostEstimate {
private:
    static const double FudgeFactor;  // 宣告
    // 不能寫成 static const double FudgeFactor = 1.35; // 錯誤
};

// 在實作檔中：
const double CostEstimate::FudgeFactor = 1.35;  // 定義並初始化
```

## 解決方案 2：enum hack

當編譯器不允許 static const 成員在宣告時獲得初值時，可以使用「enum hack」技巧：

```cpp
class GamePlayer {
private:
    // enum hack：以 enum 取代 #define
    enum { NumTurns = 5 };
    int scores[NumTurns];
};
```

**enum hack 的特性：**

1. **行為像 #define**
   - 不能取地址（像 #define）
   - 不會導致不必要的記憶體配置

2. **是語言的一部分**
   - 有型別檢查
   - 作用域規則適用

3. **實務應用**
   - 許多程式碼使用這個技巧
   - 認識它有助於閱讀他人程式碼
   - 模板元程式設計（TMP）的基礎技術之一

```cpp
// enum hack 的比較
class Example {
private:
    static const int x = 5;  // 可能配置記憶體
    enum { y = 5 };          // 絕對不配置記憶體
};

// const int* p1 = &Example::x;  // 可能可以
// const int* p2 = &Example::y;  // 錯誤，不能取 enum 的地址
```

## 解決方案 3：用 inline 函式取代 macro

### 問題回顧

```cpp
// macro 的問題
#define CALL_WITH_MAX(a, b) f((a) > (b) ? (a) : (b))

int a = 5, b = 0;
CALL_WITH_MAX(++a, b);      // a 被累加兩次！展開後：
// f((++a) > (b) ? (++a) : (b))
//    ^^^           ^^^
//    第一次        第二次（如果條件為真）
```

### 解決方案：template inline 函式

```cpp
// 使用 template inline 函式
template<typename T>
inline void callWithMax(const T& a, const T& b) {
    f(a > b ? a : b);
}

int a = 5, b = 0;
callWithMax(++a, b);  // 正確：a 只被累加一次
```

**優點：**
- 真正的函式，遵守作用域和型別規則
- 參數只被求值一次
- 可以取得函式地址
- 型別安全：編譯器會檢查型別

### inline 函式的特性

```cpp
// 可以獲得 macro 的效率，又有函式的安全性
inline int max(int a, int b) {
    return a > b ? a : b;
}

// 編譯器可能會將函式呼叫展開，避免函式呼叫開銷
int result = max(10, 20);  // 可能被展開為：int result = 10 > 20 ? 10 : 20;
```

**注意：**
- `inline` 只是對編譯器的建議，不是命令
- 編譯器可能忽略 inline 要求
- 複雜的函式通常不會被 inline

## 現代 C++ 的補充（C++11 及之後）

### constexpr（C++11）

```cpp
// C++11: 使用 constexpr
constexpr double AspectRatio = 1.653;

// constexpr 函式
constexpr int square(int x) {
    return x * x;
}

// 可以在編譯期求值
int arr[square(5)];  // 相當於 int arr[25];
```

**constexpr 的優勢：**
- 保證編譯期求值
- 比 const 更強的保證
- 可以用於需要常數表達式的地方

### inline 變數（C++17）

```cpp
// C++17: inline 變數
class GamePlayer {
private:
    inline static const int NumTurns = 5;  // C++17
    // 不需要在 .cpp 檔案中定義
};
```

## 實務建議

### 取代 #define 常數的決策樹

```
需要定義常數？
│
├─ 簡單常數（如數字、字串）
│  └─ 使用 const 或 constexpr
│
├─ class 專屬常數
│  ├─ C++17: inline static const
│  ├─ 整數型別: static const（class 內初始化）
│  ├─ 其他型別: static const（實作檔初始化）
│  └─ 老式編譯器: enum hack
│
└─ 需要在模板元程式設計中使用
   └─ 使用 enum hack
```

### 取代 #define macro 的決策樹

```
需要類似函式的 macro？
│
├─ 簡單函式
│  └─ 使用 inline 函式
│
├─ 需要型別通用性
│  └─ 使用 template inline 函式
│
└─ 需要編譯期求值（C++11+）
   └─ 使用 constexpr 函式
```

## #define 的合理使用時機

雖然建議避免使用 `#define`，但以下情況仍然合理：

### 1. Include Guards

```cpp
// 合理使用
#ifndef MYCLASS_H
#define MYCLASS_H

class MyClass {
    // ...
};

#endif
```

或使用 `#pragma once`（非標準但廣泛支援）：

```cpp
// 更簡潔（但非標準）
#pragma once

class MyClass {
    // ...
};
```

### 2. 條件編譯

```cpp
// 合理使用
#ifdef DEBUG
    std::cout << "Debug info" << std::endl;
#endif

#ifdef _WIN32
    // Windows 特定程式碼
#elif defined(__linux__)
    // Linux 特定程式碼
#endif
```

## 關鍵重點總結

### 為什麼要避免 #define？

1. **除錯困難**：名稱不會進入符號表
2. **缺乏型別安全**：純文字替換，沒有型別檢查
3. **作用域問題**：不遵守 C++ 的作用域規則
4. **無法封裝**：無法創建 class 專屬常數
5. **macro 陷阱**：參數可能被多次求值

### 替代方案

| 情境 | #define | 推薦替代 |
|------|---------|----------|
| 常數 | `#define PI 3.14159` | `const double Pi = 3.14159;` 或 `constexpr` |
| class 常數 | `#define SIZE 10` | `static const int Size = 10;` 或 `enum` |
| 函式 macro | `#define MAX(a,b) ...` | `template<typename T> inline T max(...)` |
| 編譯期常數 | `#define SIZE 100` | `constexpr int Size = 100;` (C++11+) |

### 核心原則

- **對於常數，寧可用 const 物件或 enum，不要用 #define**
- **對於形似函式的 macro，寧可用 inline 函式，不要用 #define**
- **讓編譯器幫你把關**：型別檢查、作用域規則、除錯支援

## 實例比較

### Before：使用 #define

```cpp
#define ASPECT_RATIO 1.653
#define MAX_SIZE 100
#define SQUARE(x) ((x) * (x))

class Widget {
    #define BUFFER_SIZE 1024  // 不會限制在 class 內
    char buffer[BUFFER_SIZE];
};

double width = 10;
double height = width / ASPECT_RATIO;

int arr[MAX_SIZE];
int x = 5;
int result = SQUARE(x++);  // x 被累加兩次！
```

### After：使用 const/enum/inline

```cpp
const double AspectRatio = 1.653;
constexpr int MaxSize = 100;

template<typename T>
inline T square(T x) { return x * x; }

class Widget {
    static const int BufferSize = 1024;  // class 專屬常數
    // 或使用：enum { BufferSize = 1024 };
    char buffer[BufferSize];
};

double width = 10;
double height = width / AspectRatio;

int arr[MaxSize];
int x = 5;
int result = square(x++);  // x 只被累加一次
```

## 結語

記住這個簡單的口訣：

> **對於常數，用 const 或 enum，不要用 #define；**</br>
> **對於 macro 函式，用 inline，不要用 #define。**

這樣做可以：
- 讓程式碼更容易除錯（名稱會進入符號表）
- 提供型別安全（編譯器會檢查）
- 遵守作用域規則（更好的封裝）
- 避免 macro 的陷阱（參數只被求值一次）
