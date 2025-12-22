+++
date = '2025-12-22T14:19:45+08:00'
draft = false
title = '[C++] STL deque（雙端佇列）'
categories = ['C++ Notes']
tags = ['C++', 'STL', 'Container']
+++

# 目錄

- [目錄](#目錄)
  - [什麼是 deque？](#什麼是-deque)
  - [內部實現原理](#內部實現原理)
    - [記憶體佈局](#記憶體佈局)
    - [map 陣列與 chunks](#map-陣列與-chunks)
  - [基本使用](#基本使用)
    - [標頭檔和宣告](#標頭檔和宣告)
    - [建構方式](#建構方式)
    - [常用成員函式](#常用成員函式)
  - [性能分析](#性能分析)
    - [時間複雜度](#時間複雜度)
    - [空間複雜度](#空間複雜度)
    - [與 vector 的性能對比](#與-vector-的性能對比)
  - [常見操作示例](#常見操作示例)
    - [兩端插入/刪除](#兩端插入刪除)
    - [隨機存取](#隨機存取)
    - [中間插入/刪除](#中間插入刪除)
  - [迭代器支持](#迭代器支持)
    - [迭代器類型](#迭代器類型)
    - [迭代器失效問題](#迭代器失效問題)
  - [常見陷阱與注意事項](#常見陷阱與注意事項)
    - [1. 記憶體不連續](#1-記憶體不連續)
    - [2. 中間插入/刪除的性能](#2-中間插入刪除的性能)
    - [3. 迭代器失效規則更複雜](#3-迭代器失效規則更複雜)
    - [4. 沒有 reserve() 和 capacity()](#4-沒有-reserve-和-capacity)
  - [實際應用場景](#實際應用場景)
    - [場景 1：滑動視窗](#場景-1滑動視窗)
    - [場景 2：任務佇列](#場景-2任務佇列)
    - [場景 3：BFS 演算法](#場景-3bfs-演算法)
  - [與其他容器的選擇](#與其他容器的選擇)
  - [實務建議](#實務建議)
    - [DO（應該做的）](#do應該做的)
    - [DON'T（不應該做的）](#dont不應該做的)
  - [小結](#小結)

## 什麼是 deque？

`std::deque`（double-ended queue，雙端佇列）是一個**可以在兩端高效插入和刪除元素的序列容器**。

**核心特性**：
- 支援兩端快速插入/刪除（O(1)）
- 支援隨機存取（O(1)）
- 記憶體分段連續（不像 vector 完全連續）
- 不會因為插入而使所有迭代器失效（僅頭尾操作）

**與 vector 的主要差異**：

| 特性 | vector | deque |
|------|--------|-------|
| 頭部插入 | O(n) | **O(1)** |
| 尾部插入 | O(1) | **O(1)** |
| 記憶體佈局 | 完全連續 | 分段連續 |
| 迭代器失效 | 插入可能全部失效 | 頭尾插入不失效 |
| `reserve()` | ✓ | ✗ |

```cpp
// vector：只能高效在尾部操作
std::vector<int> vec;
vec.push_back(1);   // O(1) ✓
vec.insert(vec.begin(), 0);  // O(n) ✗

// deque：兩端都可高效操作
std::deque<int> dq;
dq.push_back(1);    // O(1) ✓
dq.push_front(0);   // O(1) ✓
```

## 內部實現原理

### 記憶體佈局

`deque` 使用**分段陣列**（segmented array）實現：

```
中央控制陣列（map）
   ↓
[ptr][ptr][ptr][ptr][ptr]
  ↓    ↓    ↓    ↓    ↓
[][][]  [][][]  [][][]  [][][]  [][][]  ← 固定大小的 chunks
        ↑                 ↑
      start             finish
```

**關鍵組件**：
1. **map**：一個指標陣列，指向各個資料區塊（chunks）
2. **chunks**：固定大小的記憶體區塊，儲存實際元素
3. **start / finish**：指向第一個和最後一個元素的迭代器

### map 陣列與 chunks

```cpp
// 簡化的 deque 實現概念
template <typename T>
class deque {
    T** map;          // 指向 chunks 的指標陣列
    size_t map_size;  // map 陣列大小
    iterator start;   // 第一個元素
    iterator finish;  // 最後一個元素的下一個位置
};
```

**優點**：
- 兩端插入/刪除只需調整指標
- 不需要移動大量元素
- 支援隨機存取（透過計算 chunk 索引 + 內部偏移）

**缺點**：
- 記憶體不連續，快取不友善
- 隨機存取比 vector 慢（需要間接定址）
- 空間開銷較大

## 基本使用

### 標頭檔和宣告

```cpp
#include <deque>

std::deque<int> dq;                // 空 deque
std::deque<int> dq2(10);           // 10 個元素，值為 0
std::deque<int> dq3(10, 42);       // 10 個元素，值為 42
std::deque<int> dq4 = {1, 2, 3};   // 初始化串列（C++11）
```

### 建構方式

```cpp
#include <deque>
#include <iostream>

int main() {
    // 1. 預設建構子
    std::deque<int> d1;

    // 2. 指定大小
    std::deque<int> d2(5);  // 5 個元素，值為 0

    // 3. 指定大小和初始值
    std::deque<int> d3(5, 100);  // 5 個元素，值為 100

    // 4. 複製建構
    std::deque<int> d4 = d3;

    // 5. 初始化串列（C++11）
    std::deque<int> d5 = {1, 2, 3, 4, 5};

    // 6. 範圍建構
    std::deque<int> d6(d5.begin(), d5.begin() + 3);  // {1, 2, 3}

    // 7. 移動建構（C++11）
    std::deque<int> d7 = std::move(d5);  // d5 變成空的

    return 0;
}
```

### 常用成員函式

```cpp
std::deque<int> dq = {1, 2, 3};

// 存取元素
dq[0];           // 不檢查邊界
dq.at(0);        // 檢查邊界
dq.front();      // 第一個元素
dq.back();       // 最後一個元素

// 容量相關
dq.size();       // 元素個數
dq.empty();      // 是否為空
// 注意：deque 沒有 capacity() 和 reserve()

// 頭部操作
dq.push_front(0);    // 頭部插入
dq.pop_front();      // 頭部刪除

// 尾部操作
dq.push_back(4);     // 尾部插入
dq.pop_back();       // 尾部刪除

// 中間操作
dq.insert(dq.begin() + 1, 99);  // 指定位置插入
dq.erase(dq.begin());           // 刪除指定位置
dq.clear();                     // 清空所有元素
```

## 性能分析

### 時間複雜度

| 操作 | 時間複雜度 | 說明 |
|------|----------|------|
| 隨機存取 `[]` / `at()` | **O(1)** | 需計算 chunk 和偏移 |
| 頭部插入 `push_front()` | **O(1)** | 只需調整指標 |
| 尾部插入 `push_back()` | **O(1)** | 只需調整指標 |
| 頭部刪除 `pop_front()` | **O(1)** | 只需調整指標 |
| 尾部刪除 `pop_back()` | **O(1)** | 只需調整指標 |
| 中間插入 `insert()` | **O(n)** | 需要移動元素 |
| 中間刪除 `erase()` | **O(n)** | 需要移動元素 |
| 查找 `find()` | **O(n)** | 線性搜尋 |

### 空間複雜度

- **儲存空間**：`sizeof(T) * size() + 額外開銷`
- **額外開銷**：
  - map 陣列：`sizeof(T*) * num_chunks`
  - 分段管理：每個 chunk 可能有未使用空間
- **通常比 vector 佔用更多記憶體**

### 與 vector 的性能對比

| 操作 | vector | deque | 勝者 |
|------|--------|-------|------|
| 隨機存取 | O(1) 極快 | O(1) 較快 | **vector** |
| 頭部插入 | O(n) | O(1) | **deque** |
| 尾部插入 | O(1) | O(1) | 相當 |
| 記憶體連續性 | 完全連續 | 分段連續 | **vector** |
| 迭代器穩定性 | 差 | 較好 | **deque** |

## 常見操作示例

### 兩端插入/刪除

```cpp
#include <deque>
#include <iostream>

int main() {
    std::deque<int> dq;

    // 兩端插入
    dq.push_back(3);    // {3}
    dq.push_front(2);   // {2, 3}
    dq.push_back(4);    // {2, 3, 4}
    dq.push_front(1);   // {1, 2, 3, 4}

    // 輸出
    for (int val : dq) {
        std::cout << val << " ";
    }
    std::cout << "\n";  // 1 2 3 4

    // 兩端刪除
    dq.pop_front();  // {2, 3, 4}
    dq.pop_back();   // {2, 3}

    std::cout << "前端: " << dq.front() << "\n";  // 2
    std::cout << "後端: " << dq.back() << "\n";   // 3

    return 0;
}
```

### 隨機存取

```cpp
#include <deque>
#include <iostream>

int main() {
    std::deque<int> dq = {10, 20, 30, 40, 50};

    // 使用 [] 運算子
    std::cout << dq[2] << "\n";  // 30

    // 使用 at()（有邊界檢查）
    try {
        std::cout << dq.at(10) << "\n";  // 拋出例外
    } catch (const std::out_of_range& e) {
        std::cerr << "錯誤: " << e.what() << "\n";
    }

    // 修改元素
    dq[0] = 100;
    dq[4] = 500;

    for (int val : dq) {
        std::cout << val << " ";
    }
    std::cout << "\n";  // 100 20 30 40 500

    return 0;
}
```

### 中間插入/刪除

```cpp
#include <deque>
#include <iostream>

int main() {
    std::deque<int> dq = {1, 2, 3, 4, 5};

    // 中間插入
    dq.insert(dq.begin() + 2, 99);  // {1, 2, 99, 3, 4, 5}

    // 中間刪除
    dq.erase(dq.begin() + 1);  // {1, 99, 3, 4, 5}

    // 刪除範圍
    dq.erase(dq.begin() + 2, dq.begin() + 4);  // {1, 99, 5}

    for (int val : dq) {
        std::cout << val << " ";
    }
    std::cout << "\n";

    return 0;
}
```

## 迭代器支持

### 迭代器類型

`deque` 提供**隨機存取迭代器**（Random Access Iterator）：

```cpp
std::deque<int> dq = {1, 2, 3, 4, 5};

auto it = dq.begin();
it + 3;     // ✓ 跳躍存取
it += 2;    // ✓ 複合賦值
it - it2;   // ✓ 迭代器相減
it < it2;   // ✓ 比較運算
it[2];      // ✓ 下標運算
```

### 迭代器失效問題

**deque 的迭代器失效規則比 vector 複雜**：

1. **頭部或尾部插入/刪除**：
   - 迭代器**不失效**（除了 `end()`）
   - 但參考和指標**可能失效**

   ```cpp
   std::deque<int> dq = {1, 2, 3};
   auto it = dq.begin() + 1;  // 指向 2

   dq.push_front(0);  // 頭部插入
   // it 仍然有效，但現在指向的位置可能改變

   std::cout << *it << "\n";  // 仍然是 2（但在記憶體中的位置可能變了）
   ```

2. **中間插入/刪除**：
   - **所有迭代器失效**
   - 參考和指標也失效

   ```cpp
   std::deque<int> dq = {1, 2, 3, 4};
   auto it = dq.begin() + 2;  // 指向 3

   dq.insert(dq.begin() + 1, 99);  // 中間插入
   // it 現在失效！
   ```

## 常見陷阱與注意事項

### 1. 記憶體不連續

```cpp
// ❌ 錯誤：假設記憶體連續
std::deque<int> dq = {1, 2, 3, 4, 5};
int* ptr = &dq[0];
// ptr + 1 不保證指向 dq[1]（可能跨 chunk）

// ✅ 正確：需要連續記憶體時使用 vector
std::vector<int> vec = {1, 2, 3, 4, 5};
int* ptr2 = &vec[0];
// ptr2 + 1 保證指向 vec[1]

// ✅ 或使用 data() 取得指標（但 deque 沒有 data()）
// int* ptr = vec.data();
```

### 2. 中間插入/刪除的性能

```cpp
// ❌ 錯誤：頻繁在中間插入
std::deque<int> dq;
for (int i = 0; i < 10000; ++i) {
    dq.insert(dq.begin() + dq.size() / 2, i);  // O(n) 每次
}

// ✅ 正確：頻繁中間操作用 list
std::list<int> lst;
auto it = lst.begin();
for (int i = 0; i < 10000; ++i) {
    lst.insert(it, i);  // O(1)
}
```

### 3. 迭代器失效規則更複雜

```cpp
// ❌ 錯誤：假設只有頭尾操作不會失效迭代器
std::deque<int> dq = {1, 2, 3};
auto it = dq.begin();

dq.push_back(4);  // it 仍然有效
dq.push_front(0); // it 仍然有效

dq.insert(dq.begin() + 2, 99);  // it 失效！
std::cout << *it << "\n";  // 未定義行為
```

### 4. 沒有 reserve() 和 capacity()

```cpp
// ❌ 錯誤：deque 沒有這些函式
std::deque<int> dq;
// dq.reserve(1000);  // 編譯錯誤
// dq.capacity();     // 編譯錯誤

// ✅ 正確：需要預留容量時使用 vector
std::vector<int> vec;
vec.reserve(1000);
```

## 實際應用場景

### 場景 1：滑動視窗

```cpp
#include <deque>
#include <iostream>
#include <vector>

// 計算滑動視窗的最大值
std::vector<int> maxSlidingWindow(const std::vector<int>& nums, int k) {
    std::deque<int> dq;  // 儲存索引
    std::vector<int> result;

    for (int i = 0; i < nums.size(); ++i) {
        // 移除超出視窗的元素
        if (!dq.empty() && dq.front() == i - k) {
            dq.pop_front();
        }

        // 維護遞減佇列
        while (!dq.empty() && nums[dq.back()] < nums[i]) {
            dq.pop_back();
        }

        dq.push_back(i);

        // 記錄結果
        if (i >= k - 1) {
            result.push_back(nums[dq.front()]);
        }
    }

    return result;
}

int main() {
    std::vector<int> nums = {1, 3, -1, -3, 5, 3, 6, 7};
    int k = 3;

    auto result = maxSlidingWindow(nums, k);

    for (int val : result) {
        std::cout << val << " ";
    }
    std::cout << "\n";  // 3 3 5 5 6 7

    return 0;
}
```

### 場景 2：任務佇列

```cpp
#include <deque>
#include <iostream>
#include <string>

struct Task {
    std::string name;
    int priority;  // 1=高, 2=中, 3=低
};

int main() {
    std::deque<Task> taskQueue;

    // 加入任務
    taskQueue.push_back({"任務 A", 2});
    taskQueue.push_back({"任務 B", 3});

    // 高優先級任務插入到前面
    taskQueue.push_front({"緊急任務", 1});

    // 處理任務（從前面取出）
    while (!taskQueue.empty()) {
        Task task = taskQueue.front();
        taskQueue.pop_front();

        std::cout << "處理: " << task.name
                  << " (優先級 " << task.priority << ")\n";
    }

    return 0;
}
```

**輸出**：
```
處理: 緊急任務 (優先級 1)
處理: 任務 A (優先級 2)
處理: 任務 B (優先級 3)
```

### 場景 3：BFS 演算法

```cpp
#include <deque>
#include <iostream>
#include <vector>

// 圖的 BFS 遍歷
void BFS(const std::vector<std::vector<int>>& graph, int start) {
    std::deque<int> queue;
    std::vector<bool> visited(graph.size(), false);

    queue.push_back(start);
    visited[start] = true;

    while (!queue.empty()) {
        int node = queue.front();
        queue.pop_front();

        std::cout << node << " ";

        // 訪問所有鄰居
        for (int neighbor : graph[node]) {
            if (!visited[neighbor]) {
                visited[neighbor] = true;
                queue.push_back(neighbor);
            }
        }
    }

    std::cout << "\n";
}

int main() {
    // 鄰接串列表示的圖
    std::vector<std::vector<int>> graph = {
        {1, 2},     // 0 的鄰居
        {0, 3, 4},  // 1 的鄰居
        {0, 4},     // 2 的鄰居
        {1},        // 3 的鄰居
        {1, 2}      // 4 的鄰居
    };

    std::cout << "BFS 從節點 0 開始: ";
    BFS(graph, 0);

    return 0;
}
```

**輸出**：
```
BFS 從節點 0 開始: 0 1 2 3 4
```

## 與其他容器的選擇

| 需求 | 推薦容器 | 理由 |
|------|---------|------|
| 頻繁兩端插入/刪除 | **deque** | 兩端操作 O(1) |
| 只需尾部操作 | **vector** | 記憶體連續，快取友善 |
| 頻繁隨機存取 | **vector** | 隨機存取更快 |
| 需要穩定的迭代器 | **list** | 插入/刪除不使迭代器失效 |
| 實作 queue / stack | **deque** | STL 預設底層容器 |

**選擇建議**：
- **預設使用 `vector`**：除非有明確理由
- **需要頻繁頭部操作時用 `deque`**
- **不要為了「可能」的頭部操作而選 `deque`**

## 實務建議

### DO（應該做的）

1. **實作 queue 或 stack 時使用 deque 作為底層容器**
   ```cpp
   std::queue<int> q;            // 預設使用 deque
   std::stack<int> s;            // 預設使用 deque
   std::priority_queue<int> pq;  // 預設使用 vector
   ```

2. **需要頻繁兩端操作時使用 deque**
   ```cpp
   std::deque<int> dq;
   dq.push_front(1);  // O(1)
   dq.push_back(2);   // O(1)
   dq.pop_front();    // O(1)
   dq.pop_back();     // O(1)
   ```

3. **滑動視窗問題使用 deque**
   ```cpp
   std::deque<int> window;
   window.push_back(new_element);
   window.pop_front();  // 移除最舊元素
   ```

4. **使用範圍 for 遍歷**
   ```cpp
   for (const auto& val : dq) {
       std::cout << val << "\n";
   }
   ```

5. **傳遞給函式時使用 const 引用**
   ```cpp
   void printDeque(const std::deque<int>& dq) {
       for (int val : dq) {
           std::cout << val << " ";
       }
   }
   ```

### DON'T（不應該做的）

1. **不要假設記憶體連續**
   ```cpp
   // ❌ 錯誤
   std::deque<int> dq = {1, 2, 3};
   int* ptr = &dq[0];
   int val = *(ptr + 1);  // 未定義行為

   // ✅ 正確
   int val = dq[1];
   ```

2. **不要頻繁在中間插入/刪除**
   ```cpp
   // ❌ 錯誤
   for (int i = 0; i < 10000; ++i) {
       dq.insert(dq.begin() + dq.size() / 2, i);
   }

   // ✅ 正確：用 list
   std::list<int> lst;
   ```

3. **不要嘗試使用 reserve()**
   ```cpp
   // ❌ 錯誤
   // dq.reserve(1000);  // 編譯錯誤

   // ✅ 正確：如需 reserve，用 vector
   std::vector<int> vec;
   vec.reserve(1000);
   ```

4. **不要在只需尾部操作時使用 deque**
   ```cpp
   // ❌ 不推薦
   std::deque<int> dq;
   dq.push_back(1);  // 雖然可以，但 vector 更好

   // ✅ 推薦
   std::vector<int> vec;
   vec.push_back(1);  // 記憶體連續，快取友善
   ```

5. **不要忽略迭代器失效規則**
   ```cpp
   // ❌ 錯誤
   for (auto it = dq.begin(); it != dq.end(); ++it) {
       dq.insert(dq.begin(), 0);  // 迭代器失效
   }

   // ✅ 正確
   for (auto it = dq.begin(); it != dq.end(); ) {
       // 處理迭代器失效
   }
   ```

## 小結

**deque 的核心特性：**

1. **雙端佇列**：頭尾插入/刪除都是 O(1)
2. **分段連續記憶體**：不像 vector 完全連續
3. **隨機存取支援**：但比 vector 稍慢
4. **迭代器穩定性較好**：頭尾操作不使迭代器失效

**適用場景：**
- 需要頻繁頭部操作（`push_front`, `pop_front`）
- 實作 `queue` 和 `stack` 的底層容器
- 滑動視窗演算法
- BFS 演算法

**與 vector 的對比：**
- **優勢**：頭部操作 O(1)
- **劣勢**：記憶體不連續、隨機存取較慢、空間開銷較大

**記住：`deque` 不是萬能的！只有在真正需要頻繁頭部操作時才使用，否則 `vector` 通常是更好的選擇。**
