+++ 
draft = false
date = 2025-12-14T16:44:34+08:00
title = "Effective C++ 01: View C++ as a federation of languages"
categories = ['Effective C++']
tags = ['C++']
+++

## 前言

本文是 Scott Meyers 所著《Effective C++》的閱讀筆記。

## 核心概念

理解 C++ 內容最簡單的方法是將它視為 **由多個「子語言（sublanguages）」組成的聯邦**，而非單一語言。每個子語言都有自己的慣例和最佳實踐，理解這點可以幫助我們在不同情境下選擇最適當的程式設計策略。

## 四個主要子語言

### 1. C

C++ 最初就是建立在 C 語言之上，這個子語言包括：

- **基本語法**：語句、預處理器、區塊結構
- **內建資料型別**：`int`、`float`、`char` 等
- **陣列和指標**：C 風格的記憶體操作
- **限制**：沒有模板、例外處理、函式多載等進階功能

這部分的規則很簡單，主要關注效能和資源控制。

### 2. Object-Oriented C++

這是 C with Classes 的精神所在：

- **類別（Classes）**：封裝資料和函式
- **建構子/解構子**：物件生命週期管理
- **封裝（Encapsulation）**：`public`、`private`、`protected`
- **繼承（Inheritance）**：程式碼重用和階層設計
- **多型（Polymorphism）**：虛擬函式、動態綁定

這部分強調物件導向設計原則，如封裝、繼承、多型。

### 3. Template C++

C++ 的泛型（generic）程式設計部分：

- **函式模板和類別模板**：型別參數化
- **模板元程式設計（template metaprogramming, TMP）**：編譯期運算
- **型別推導**：`auto`、`decltype` 等

這部分的規則有時與其他子語言差異很大，有自己獨特的慣例：
- 例如模板程式碼通常放在標頭檔
- 編譯期多型 vs 執行期多型的取捨

### 4. STL

標準模板函式庫有自己特定的使用方式：

- **容器（Containers）**：`vector`、`list`、`map` 等
- **迭代器（Iterators）**：統一的遍歷介面
- **演算法（Algorithms）**：`sort`、`find`、`transform` 等
- **函式物件（Function Objects）**：仿函式

STL 基於特定的設計哲學，有自己的慣例和最佳實踐。

## 實際影響：參數傳遞策略

不同子語言對於參數傳遞有不同的最佳實踐：

### C 子語言：內建型別用 pass-by-value

```cpp
// 對於小型內建型別，直接傳值最有效率
void processInt(int value) {
    // 直接複製一個 int 很快
}

void processDouble(double value) {
    // double 也是小型型別，傳值即可
}
```

**原因**：內建型別很小，複製成本低，且編譯器可以把值放在暫存器中。

### Object-Oriented C++：自定義物件用 pass-by-reference-to-const

```cpp
class Widget {
    // 假設有很多成員變數
    std::string name;
    std::vector<int> data;
    // ...
};

// 傳參考避免複製整個物件
void processWidget(const Widget& w) {
    // 只傳遞參考（基本上是指標），避免昂貴的複製
}
```

**原因**：
- 避免呼叫複製建構子（可能很昂貴）
- 避免[物件切割（slicing）問題](/posts/cpp-object-slicing/)
- `const` 保證不會修改原物件

### STL：迭代器和函式物件常用 pass-by-value

```cpp
// 迭代器本質上是輕量級物件（類似指標）
void processRange(std::vector<int>::iterator begin,
                  std::vector<int>::iterator end) {
    // 迭代器傳值沒問題，它們設計上就很小
}

// 函式物件也通常設計為可複製的輕量級物件
template<typename T>
void applyOperation(std::vector<T>& vec, std::function<void(T&)> op) {
    for (auto& elem : vec) {
        op(elem);
    }
}
```

**原因**：STL 的設計基於 C 指標模型，迭代器和函式物件刻意設計為輕量、可複製。

## 為什麼這很重要？

### 1. 選擇適當的策略

理解你正在使用哪個子語言，可以幫助你選擇最適當的程式設計方法。

```cpp
// 錯誤示範：混淆子語言的規則
void badExample(std::string str) {  // 不必要的複製！
    // 應該用 const std::string&
}

// 正確做法
void goodExample(const std::string& str) {  // 避免複製
    // ...
}
```

### 2. 理解規則的適用性

某些規則在某個子語言中是好的，在另一個子語言中可能不適用。

```cpp
// 在 C 子語言中，指標操作很正常
void cStyle(int* arr, size_t len) {
    for (size_t i = 0; i < len; ++i) {
        arr[i] = 0;
    }
}

// 在 STL 子語言中，應該用容器和演算法
void stlStyle(std::vector<int>& vec) {
    std::fill(vec.begin(), vec.end(), 0);
}
```

### 3. 更有效率的學習

把 C++ 視為聯邦語言，可以幫助你：
- 分門別類地學習不同特性
- 理解為什麼有些建議看起來互相矛盾
- 知道何時該打破「規則」

## 關鍵重點總結

1. **C++ 是多範式語言**：支援過程式、物件導向、函式式、泛型程式設計
2. **沒有一體適用的規則**：不同子語言有不同的最佳實踐
3. **效率考量因子語言而異**：
   - C 子語言：直接記憶體操作最快
   - OO C++：注意物件生命週期和虛擬函式開銷
   - Template C++：善用編譯期運算
   - STL：了解容器和演算法的複雜度保證
4. **靈活運用**：根據情境選擇最適合的子語言特性

## 實務建議

- **寫程式碼時問自己**：我現在用的是哪個子語言？
- **不要死守單一規則**：理解規則背後的原因，知道何時該打破它
- **善用各子語言的優勢**：
  - 需要效能時用 C 風格
  - 需要抽象時用 OO 特性
  - 需要型別安全的泛用性時用模板
  - 需要標準演算法時用 STL
