+++
date = '2025-12-22T14:14:16+08:00'
draft = false
title = '[C++] STL 標準模板庫：完整指南'
categories = ['C++ Notes']
tags = ['C++', 'STL']
+++

# 目錄

- [目錄](#目錄)
  - [什麼是 STL？](#什麼是-stl)
  - [STL 的歷史演進](#stl-的歷史演進)
  - [STL 的六大組件](#stl-的六大組件)
    - [1. 容器（Containers）](#1-容器containers)
    - [2. 迭代器（Iterators）](#2-迭代器iterators)
    - [3. 演算法（Algorithms）](#3-演算法algorithms)
    - [4. 函式物件（Function Objects / Functors）](#4-函式物件function-objects--functors)
    - [5. 適配器（Adapters）](#5-適配器adapters)
    - [6. 分配器（Allocators）](#6-分配器allocators)
  - [容器分類體系](#容器分類體系)
    - [序列容器（Sequence Containers）](#序列容器sequence-containers)
    - [關聯容器（Associative Containers）](#關聯容器associative-containers)
    - [無序容器（Unordered Containers）](#無序容器unordered-containers)
    - [容器適配器（Container Adapters）](#容器適配器container-adapters)
  - [如何選擇合適的容器？](#如何選擇合適的容器)
    - [決策流程圖](#決策流程圖)
    - [常見使用場景](#常見使用場景)
  - [容器性能對比](#容器性能對比)
    - [時間複雜度總覽](#時間複雜度總覽)
    - [空間使用特性](#空間使用特性)
  - [小結](#小結)

## 什麼是 STL？

**STL（Standard Template Library，標準模板庫）** 是 C++ 標準函式庫的核心部分，提供了一組通用的、高效的、可重用的模板類別和函式。

STL 的核心理念是**將資料結構和演算法分離**：
- **容器**負責儲存資料
- **演算法**負責處理資料
- **迭代器**作為兩者之間的橋梁

這種設計讓你可以：
```cpp
std::vector<int> vec = {1, 2, 3, 4, 5};
std::sort(vec.begin(), vec.end());  // 任何容器都可使用 sort
```

## STL 的歷史演進

| 年份 | 重要事件 |
|------|---------|
| **1979** | Alexander Stepanov 開始研究泛型程式設計 |
| **1992** | Stepanov 和 Meng Lee 在 HP 實驗室開發 HP STL |
| **1994** | STL 被提議納入 C++ 標準（ANSI/ISO） |
| **1998** | **C++98** - STL 正式成為 C++ 標準的一部分 |
| **2011** | **C++11** - 引入新容器（`array`, `forward_list`, `unordered_*`）、右值引用、移動語意 |
| **2014** | **C++14** - 改進泛型 lambda、`std::make_unique` |
| **2017** | **C++17** - 引入 `std::optional`, `std::variant`, `std::any`、平行演算法 |
| **2020** | **C++20** - Ranges 函式庫、Concepts、`std::span` |

**關鍵影響**：
- STL 讓 C++ 程式設計從「重新發明輪子」轉向「組合現有組件」
- 促進了泛型程式設計的普及
- 影響了其他程式語言的標準函式庫設計（如 Java Collections、.NET LINQ）

## STL 的六大組件

### 1. 容器（Containers）

容器是儲存資料的模板類別，負責管理記憶體和提供存取介面。

**分類**：
- **序列容器**：`vector`, `deque`, `list`, `array`, `forward_list`
- **關聯容器**：`set`, `map`, `multiset`, `multimap`
- **無序容器**：`unordered_set`, `unordered_map`, `unordered_multiset`, `unordered_multimap`
- **容器適配器**：`stack`, `queue`, `priority_queue`

### 2. 迭代器（Iterators）

迭代器是連接容器和演算法的橋梁，類似於指標的行為。

**分類**：
- **輸入迭代器**（Input Iterator）- 唯讀、單向
- **輸出迭代器**（Output Iterator）- 唯寫、單向
- **前向迭代器**（Forward Iterator）- 可讀寫、單向
- **雙向迭代器**（Bidirectional Iterator）- 可讀寫、雙向（`++`, `--`）
- **隨機存取迭代器**（Random Access Iterator）- 可讀寫、任意跳躍（`+`, `-`, `[]`）

```cpp
std::vector<int> vec = {1, 2, 3, 4, 5};

// 迭代器遍歷
for (auto it = vec.begin(); it != vec.end(); ++it) {
    std::cout << *it << " ";
}

// 範圍 for（背後使用迭代器）
for (int val : vec) {
    std::cout << val << " ";
}
```

### 3. 演算法（Algorithms）

STL 提供了約 100 個常用演算法，涵蓋排序、查找、修改、數值運算等。

**常見分類**：
- **非修改演算法**：`find`, `count`, `equal`, `search`
- **修改演算法**：`copy`, `transform`, `replace`, `remove`
- **排序演算法**：`sort`, `stable_sort`, `partial_sort`
- **數值演算法**：`accumulate`, `inner_product`

```cpp
std::vector<int> vec = {3, 1, 4, 1, 5, 9};

// 排序
std::sort(vec.begin(), vec.end());

// 查找
auto it = std::find(vec.begin(), vec.end(), 4);

// 累加
int sum = std::accumulate(vec.begin(), vec.end(), 0);
```

### 4. 函式物件（Function Objects / Functors）

函式物件是重載了 `operator()` 的類別，可以像函式一樣被呼叫。

```cpp
// 內建函式物件
std::vector<int> vec = {3, 1, 4, 1, 5};
std::sort(vec.begin(), vec.end(), std::greater<int>());  // 降序排序

// 自訂函式物件
struct MultiplyBy {
    int factor;
    MultiplyBy(int f) : factor(f) {}
    int operator()(int x) const { return x * factor; }
};

std::vector<int> nums = {1, 2, 3, 4, 5};
std::transform(nums.begin(), nums.end(), nums.begin(), MultiplyBy(2));
// nums 變成 {2, 4, 6, 8, 10}
```

### 5. 適配器（Adapters）

適配器將一個介面轉換為另一個介面。

**容器適配器**：
- `stack` - 後進先出（LIFO），基於 `deque` 實作
- `queue` - 先進先出（FIFO），基於 `deque` 實作
- `priority_queue` - 優先權佇列，基於 `vector` + heap 實作

**迭代器適配器**：
- `reverse_iterator` - 反向遍歷
- `insert_iterator` - 插入元素
- `stream_iterator` - 串流輸入/輸出

### 6. 分配器（Allocators）

分配器負責記憶體的配置和釋放，通常使用預設分配器 `std::allocator`。

```cpp
// 預設分配器
std::vector<int> vec;  // 使用 std::allocator<int>

// 自訂分配器（進階用法）
template <typename T>
class MyAllocator { /* ... */ };

std::vector<int, MyAllocator<int>> vec;
```

## 容器分類體系

### 序列容器（Sequence Containers）

**特性**：元素按插入順序排列，可以指定位置插入/刪除。

| 容器 | 底層結構 | 特點 | C++ 版本 | 詳細文章 |
|------|---------|------|----------|---------|
| `vector` | 動態陣列 | 隨機存取快、尾部插入快 | C++98 | [[C++] STL Vector（動態陣列）](Cpp-STL-Vector.md) |
| `deque` | 分段陣列 | 兩端插入/刪除快 | C++98 | [[C++] STL Deque（雙端佇列）](Cpp-STL-Deque.md) |
| `list` | 雙向鏈結串列 | 任意位置插入/刪除快 | C++98 | [[C++] STL List（雙向鏈表）](Cpp-STL-List.md) |
| `array` | 固定大小陣列 | 無動態配置、效率最高 | C++11 | [[C++] STL Array（固定大小陣列）](Cpp-STL-Array.md) |
| `forward_list` | 單向鏈結串列 | 記憶體使用最小 | C++11 | [[C++] STL Forward_list（單向鏈表）](Cpp-STL-ForwardList.md) |

### 關聯容器（Associative Containers）

**特性**：元素自動排序（通常使用紅黑樹實作），查找效率高 O(log n)。

| 容器 | 儲存內容 | 唯一性 | C++ 版本 | 詳細文章 |
|------|---------|--------|----------|---------|
| `set` | 鍵值 | 鍵值唯一 | C++98 | [[C++] STL Set（集合容器）](Cpp-STL-Set.md) |
| `multiset` | 鍵值 | 鍵值可重複 | C++98 | [[C++] STL Multiset（多重集合容器）](Cpp-STL-Multiset.md) |
| `map` | 鍵-值對 | 鍵唯一 | C++98 | [[C++] STL Map（映射容器）](Cpp-STL-Map.md) |
| `multimap` | 鍵-值對 | 鍵可重複 | C++98 | [[C++] STL Multimap（多重映射容器）](Cpp-STL-Multimap.md) |

### 無序容器（Unordered Containers）

**特性**：使用哈希表實作，查找平均 O(1)，但元素無序。

| 容器 | 儲存內容 | 唯一性 | C++ 版本 | 詳細文章 |
|------|---------|--------|----------|---------|
| `unordered_set` | 鍵值 | 鍵值唯一 | C++11 | [[C++] STL Unordered_set（無序集合容器）](Cpp-STL-UnorderedSet.md) |
| `unordered_multiset` | 鍵值 | 鍵值可重複 | C++11 | [[C++] STL Unordered_multiset（無序多重集合）](Cpp-STL-UnorderedMultiset.md) |
| `unordered_map` | 鍵-值對 | 鍵唯一 | C++11 | [[C++] STL Unordered_map（無序映射容器）](Cpp-STL-UnorderedMap.md) |
| `unordered_multimap` | 鍵-值對 | 鍵可重複 | C++11 | [[C++] STL Unordered_multimap（無序多重映射）](Cpp-STL-UnorderedMultimap.md) |

### 容器適配器（Container Adapters）

**特性**：基於其他容器實作，提供特定介面。

| 適配器 | 特性 | 預設底層容器 | C++ 版本 | 詳細文章 |
|--------|------|-------------|----------|---------|
| `stack` | 後進先出（LIFO） | `deque` | C++98 | [[C++] STL Stack（棧容器適配器）](Cpp-STL-Stack.md) |
| `queue` | 先進先出（FIFO） | `deque` | C++98 | [[C++] STL Queue（佇列容器適配器）](Cpp-STL-Queue.md) |
| `priority_queue` | 優先權佇列（Heap） | `vector` | C++98 | [[C++] STL Priority_queue（優先佇列容器適配器）](Cpp-STL-PriorityQueue.md) |

## 如何選擇合適的容器？

### 決策流程圖

```
需要儲存元素嗎？
│
├─ 是 → 需要排序嗎？
│       │
│       ├─ 是 → 需要快速查找（O(log n)）嗎？
│       │       │
│       │       ├─ 是 → 需要儲存鍵值對嗎？
│       │       │       ├─ 是 → 鍵唯一？
│       │       │       │       ├─ 是 → map
│       │       │       │       └─ 否 → multimap
│       │       │       └─ 否 → 鍵唯一？
│       │       │               ├─ 是 → set
│       │       │               └─ 否 → multiset
│       │       │
│       │       └─ 否 → 需要最快查找（O(1)）嗎？
│       │               ├─ 是 → 需要鍵值對？
│       │               │       ├─ 是 → 鍵唯一？
│       │               │       │       ├─ 是 → unordered_map
│       │               │       │       └─ 否 → unordered_multimap
│       │               │       └─ 否 → 鍵唯一？
│       │               │               ├─ 是 → unordered_set
│       │               │               └─ 否 → unordered_multiset
│       │               └─ 否 → 使用 vector + sort
│       │
│       └─ 否 → 需要特定存取方式嗎？
│               ├─ LIFO（後進先出）→ stack
│               ├─ FIFO（先進先出）→ queue
│               ├─ 優先權 → priority_queue
│               └─ 一般 → 需要隨機存取嗎？
│                         ├─ 是 → 大小固定？
│                         │       ├─ 是 → array
│                         │       └─ 否 → vector
│                         └─ 否 → 頻繁插入/刪除於？
│                                 ├─ 兩端 → deque
│                                 ├─ 任意位置 → list
│                                 └─ 只需單向 → forward_list
│
└─ 否 → （不需要容器）
```

### 常見使用場景

| 場景 | 推薦容器 | 理由 |
|------|---------|------|
| 動態陣列、頻繁尾部插入 | `vector` | 記憶體連續、快取友善、隨機存取 O(1) |
| 兩端頻繁插入/刪除 | `deque` | 兩端操作 O(1)、支援隨機存取 |
| 頻繁在中間插入/刪除 | `list` | 任意位置插入/刪除 O(1)（已知位置） |
| 固定大小、不需動態配置 | `array` | 無額外開銷、效能最佳 |
| 需要排序且快速查找 | `set` / `map` | 自動排序、查找 O(log n) |
| 需要最快查找、不在意順序 | `unordered_set` / `unordered_map` | 平均查找 O(1) |
| 實作 DFS、undo 功能 | `stack` | LIFO 特性 |
| 實作 BFS、任務佇列 | `queue` | FIFO 特性 |
| 需要取得最大/最小值 | `priority_queue` | 堆積結構、取極值 O(1) |

## 容器性能對比

### 時間複雜度總覽

| 操作 | vector | deque | list | set/map | unordered_set/map |
|------|--------|-------|------|---------|-------------------|
| **隨機存取** | O(1) | O(1) | O(n) | - | - |
| **頭部插入** | O(n) | O(1) | O(1) | - | - |
| **尾部插入** | O(1)* | O(1) | O(1) | - | - |
| **中間插入** | O(n) | O(n) | O(1)** | - | - |
| **查找** | O(n) | O(n) | O(n) | O(log n) | O(1)*** |
| **刪除（已知位置）** | O(n) | O(n) | O(1) | O(log n) | O(1)*** |

- \* `vector` 尾部插入：均攤 O(1)，最壞情況 O(n)（需要重新配置）
- \*\* `list` 中間插入：O(1) 假設已有迭代器指向該位置
- \*\*\* `unordered` 容器：平均 O(1)，最壞情況 O(n)（哈希衝突）

### 空間使用特性

| 容器 | 額外開銷 | 記憶體連續性 | 備註 |
|------|---------|------------|------|
| `vector` | 低（容量 > 大小時有浪費） | 連續 | 重新配置時可能有大量複製 |
| `deque` | 中等 | 分段連續 | 兩端插入不會使迭代器失效 |
| `list` | 高（每個節點有指標） | 不連續 | 插入/刪除不會使迭代器失效 |
| `set`/`map` | 中高（紅黑樹節點） | 不連續 | 有序性保證需要額外開銷 |
| `unordered_*` | 中等（哈希桶 + 鏈結） | 不連續 | load factor 影響性能 |

## 小結

**STL 的核心價值：**

1. **高度重用性**：容器、演算法、迭代器可任意組合
2. **效率保證**：每個操作的時間複雜度有明確規範
3. **型別安全**：模板提供編譯期型別檢查
4. **易於維護**：使用標準元件，減少自訂實作的 bug

**選擇容器的黃金法則：**

1. **預設使用 `vector`**：90% 情況下它是最佳選擇
2. **需要兩端操作時用 `deque`**
3. **需要頻繁中間插入/刪除時用 `list`**
4. **需要有序查找時用 `set`/`map`**
5. **需要最快查找時用 `unordered_set`/`unordered_map`**

**記住：沒有「完美」的容器，只有「最適合」的容器。選擇容器時要根據實際需求的操作模式來決定！**

---

## 系列文章導覽

### 序列容器
1. [[C++] STL Vector（動態陣列）](Cpp-STL-Vector.md) - 最常用的容器
2. [[C++] STL Deque（雙端佇列）](Cpp-STL-Deque.md) - 兩端操作高效
3. [[C++] STL List（雙向鏈表）](Cpp-STL-List.md) - 中間插入刪除快
4. [[C++] STL Array（固定大小陣列）](Cpp-STL-Array.md) - 零開銷封裝
5. [[C++] STL Forward_list（單向鏈表）](Cpp-STL-ForwardList.md) - 最小記憶體開銷

### 關聯容器
6. [[C++] STL Map（映射容器）](Cpp-STL-Map.md) - 有序鍵值對
7. [[C++] STL Set（集合容器）](Cpp-STL-Set.md) - 有序唯一元素
8. [[C++] STL Multimap（多重映射容器）](Cpp-STL-Multimap.md) - 允許重複鍵
9. [[C++] STL Multiset（多重集合容器）](Cpp-STL-Multiset.md) - 允許重複元素

### 無序容器
10. [[C++] STL Unordered_map（無序映射容器）](Cpp-STL-UnorderedMap.md) - O(1) 查找鍵值對
11. [[C++] STL Unordered_set（無序集合容器）](Cpp-STL-UnorderedSet.md) - O(1) 查找元素
12. [[C++] STL Unordered_multimap（無序多重映射）](Cpp-STL-UnorderedMultimap.md)
13. [[C++] STL Unordered_multiset（無序多重集合）](Cpp-STL-UnorderedMultiset.md)

### 容器適配器
14. [[C++] STL Stack（棧容器適配器）](Cpp-STL-Stack.md) - LIFO 結構
15. [[C++] STL Queue（佇列容器適配器）](Cpp-STL-Queue.md) - FIFO 結構
16. [[C++] STL Priority_queue（優先佇列容器適配器）](Cpp-STL-PriorityQueue.md) - 堆結構

---

**學習建議：**
- 建議從 **Vector** 開始學習（最常用）
- 理解迭代器失效問題
- 練習使用 STL 演算法
- 學習 C++11/14/17 的新特性
