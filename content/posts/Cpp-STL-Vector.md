+++
date = '2025-12-22T14:17:12+08:00'
draft = false
title = '[C++] STL vector（動態陣列）'
categories = ['C++ Notes']
tags = ['C++', 'STL', 'Container']
+++

# 目錄

- [目錄](#目錄)
  - [什麼是 vector？](#什麼是-vector)
  - [內部實現原理](#內部實現原理)
    - [記憶體佈局](#記憶體佈局)
    - [容量增長策略](#容量增長策略)
  - [基本使用](#基本使用)
    - [標頭檔和宣告](#標頭檔和宣告)
    - [建構方式](#建構方式)
    - [常用成員函式](#常用成員函式)
  - [性能分析](#性能分析)
    - [時間複雜度](#時間複雜度)
    - [空間複雜度](#空間複雜度)
    - [與其他容器的性能對比](#與其他容器的性能對比)
  - [常見操作示例](#常見操作示例)
    - [插入元素](#插入元素)
    - [刪除元素](#刪除元素)
    - [存取元素](#存取元素)
    - [遍歷元素](#遍歷元素)
  - [迭代器支持](#迭代器支持)
    - [迭代器類型](#迭代器類型)
    - [迭代器失效問題](#迭代器失效問題)
  - [常見陷阱與注意事項](#常見陷阱與注意事項)
    - [1. 重新配置導致的迭代器失效](#1-重新配置導致的迭代器失效)
    - [2. 容量浪費](#2-容量浪費)
    - [3. 中間插入/刪除的性能問題](#3-中間插入刪除的性能問題)
    - [4. 儲存 bool 的特殊情況](#4-儲存-bool-的特殊情況)
  - [實際應用場景](#實際應用場景)
    - [場景 1：動態增長的資料集合](#場景-1動態增長的資料集合)
    - [場景 2：需要隨機存取的情境](#場景-2需要隨機存取的情境)
    - [場景 3：與演算法配合使用](#場景-3與演算法配合使用)
  - [與其他容器的選擇](#與其他容器的選擇)
  - [實務建議](#實務建議)
    - [DO（應該做的）](#do應該做的)
    - [DON'T（不應該做的）](#dont不應該做的)
  - [小結](#小結)

## 什麼是 vector？

`std::vector` 是 C++ STL 中最常用的序列容器，它是一個**動態大小的陣列**，能夠在執行期自動調整大小。

**核心特性**：
- 底層使用連續記憶體儲存元素
- 支援隨機存取（透過 `[]` 或 `at()`）
- 尾部插入/刪除效率高
- 自動管理記憶體（RAII）

**與 C 語言陣列的比較**：

```cpp
// C 語言陣列
int arr[10];  // 固定大小，無法動態增長
arr[5] = 42;

// C++ vector
std::vector<int> vec;  // 初始為空
vec.push_back(42);     // 動態增長
vec.push_back(100);    // 繼續增長
```

## 內部實現原理

### 記憶體佈局

`vector` 內部維護三個指標：

```
[開始指標] [結束指標] [容量結束指標]
    ↓          ↓            ↓
  [1][2][3][4][5][ ][ ][ ]
   ←─── size ──→
   ←──── capacity ────→
```

- **開始指標**：指向第一個元素
- **結束指標**：指向最後一個元素的下一個位置
- **容量結束指標**：指向已配置記憶體的結束位置

```cpp
std::vector<int> vec = {1, 2, 3, 4, 5};
std::cout << "大小: " << vec.size() << "\n";      // 5
std::cout << "容量: " << vec.capacity() << "\n";  // 可能是 8 或更大
```

### 容量增長策略

當 `size == capacity` 時，繼續插入元素會觸發**重新配置**：

1. 配置一塊更大的記憶體（通常是當前容量的 1.5 倍或 2 倍）
2. 將舊元素複製/移動到新記憶體
3. 釋放舊記憶體
4. 更新指標

```cpp
std::vector<int> vec;
std::cout << "初始容量: " << vec.capacity() << "\n";  // 0

for (int i = 0; i < 10; ++i) {
    vec.push_back(i);
    std::cout << "大小: " << vec.size()
              << ", 容量: " << vec.capacity() << "\n";
}
// 輸出可能是：
// 大小: 1, 容量: 1
// 大小: 2, 容量: 2
// 大小: 3, 容量: 4
// 大小: 4, 容量: 4
// 大小: 5, 容量: 8
// ...
```

**不同編譯器的增長策略**：
- **GCC/Clang**：通常是 2 倍增長
- **MSVC**：通常是 1.5 倍增長

## 基本使用

### 標頭檔和宣告

```cpp
#include <vector>

std::vector<int> vec;              // 空 vector
std::vector<int> vec2(10);         // 10 個元素，值為 0
std::vector<int> vec3(10, 42);     // 10 個元素，值為 42
std::vector<int> vec4 = {1, 2, 3}; // 初始化串列（C++11）
```

### 建構方式

```cpp
#include <vector>
#include <iostream>

int main() {
    // 1. 預設建構子
    std::vector<int> v1;

    // 2. 指定大小
    std::vector<int> v2(5);  // 5 個元素，值為 0

    // 3. 指定大小和初始值
    std::vector<int> v3(5, 100);  // 5 個元素，值為 100

    // 4. 複製建構
    std::vector<int> v4 = v3;

    // 5. 初始化串列（C++11）
    std::vector<int> v5 = {1, 2, 3, 4, 5};

    // 6. 範圍建構
    std::vector<int> v6(v5.begin(), v5.begin() + 3);  // {1, 2, 3}

    // 7. 移動建構（C++11）
    std::vector<int> v7 = std::move(v5);  // v5 變成空的

    return 0;
}
```

### 常用成員函式

```cpp
std::vector<int> vec = {1, 2, 3};

// 存取元素
vec[0];           // 不檢查邊界
vec.at(0);        // 檢查邊界，超出範圍拋出例外
vec.front();      // 第一個元素
vec.back();       // 最後一個元素
vec.data();       // 取得底層陣列指標

// 容量相關
vec.size();       // 元素個數
vec.capacity();   // 已配置容量
vec.empty();      // 是否為空
vec.reserve(100); // 預留容量（不改變 size）
vec.shrink_to_fit();  // 釋放多餘容量

// 修改操作
vec.push_back(4);       // 尾部加入
vec.pop_back();         // 移除最後一個
vec.insert(vec.begin(), 0);  // 指定位置插入
vec.erase(vec.begin());      // 刪除指定位置
vec.clear();            // 清空所有元素
vec.resize(10);         // 改變大小
```

## 性能分析

### 時間複雜度

| 操作 | 時間複雜度 | 說明 |
|------|----------|------|
| 隨機存取 `[]` / `at()` | **O(1)** | 連續記憶體，直接計算位址 |
| 尾部插入 `push_back()` | **O(1) 均攤** | 不需重新配置時 O(1)，需要時 O(n) |
| 尾部刪除 `pop_back()` | **O(1)** | 只需移動 size 指標 |
| 頭部/中間插入 `insert()` | **O(n)** | 需要移動後續所有元素 |
| 頭部/中間刪除 `erase()` | **O(n)** | 需要移動後續所有元素 |
| 查找 `find()` | **O(n)** | 需要線性搜尋 |

### 空間複雜度

- **儲存空間**：`sizeof(T) * capacity()`
- **額外開銷**：3 個指標（24 bytes，64-bit 系統）
- **記憶體浪費**：`(capacity - size) * sizeof(T)`

```cpp
std::vector<int> vec;
vec.reserve(1000);  // 配置 1000 個 int 的空間
vec.push_back(1);   // 只使用 1 個
// 浪費：(1000 - 1) * 4 bytes = 3996 bytes
```

### 與其他容器的性能對比

| 容器 | 隨機存取 | 尾部插入 | 中間插入 | 記憶體開銷 |
|------|---------|---------|---------|-----------|
| **vector** | O(1) | O(1)* | O(n) | 低 |
| **deque** | O(1) | O(1) | O(n) | 中 |
| **list** | O(n) | O(1) | O(1) | 高 |

## 常見操作示例

### 插入元素

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> vec = {1, 2, 3};

    // 尾部插入（最常用）
    vec.push_back(4);  // {1, 2, 3, 4}

    // 頭部插入（效率低）
    vec.insert(vec.begin(), 0);  // {0, 1, 2, 3, 4}

    // 中間插入
    vec.insert(vec.begin() + 2, 99);  // {0, 1, 99, 2, 3, 4}

    // 批量插入
    vec.insert(vec.end(), {10, 20, 30});  // 尾部加入 3 個元素

    // C++11: emplace_back（直接建構）
    vec.emplace_back(100);  // 比 push_back 效率更高（避免複製）

    // 輸出
    for (int val : vec) {
        std::cout << val << " ";
    }
    std::cout << "\n";

    return 0;
}
```

**輸出**：
```
0 1 99 2 3 4 10 20 30 100
```

### 刪除元素

```cpp
#include <vector>
#include <algorithm>
#include <iostream>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5, 3, 6};

    // 刪除最後一個
    vec.pop_back();  // {1, 2, 3, 4, 5, 3}

    // 刪除指定位置
    vec.erase(vec.begin());  // {2, 3, 4, 5, 3}

    // 刪除範圍
    vec.erase(vec.begin() + 1, vec.begin() + 3);  // {2, 5, 3}

    // 刪除所有值為 3 的元素（erase-remove idiom）
    vec.erase(std::remove(vec.begin(), vec.end(), 3), vec.end());
    // {2, 5}

    // 清空
    vec.clear();  // {}

    std::cout << "大小: " << vec.size() << "\n";  // 0
    std::cout << "容量: " << vec.capacity() << "\n";  // 容量不變！

    return 0;
}
```

### 存取元素

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> vec = {10, 20, 30, 40, 50};

    // 方法 1：[] 運算子（不檢查邊界）
    std::cout << vec[0] << "\n";  // 10
    // vec[10];  // 未定義行為！可能當機

    // 方法 2：at()（檢查邊界）
    std::cout << vec.at(1) << "\n";  // 20
    try {
        std::cout << vec.at(10) << "\n";  // 拋出 std::out_of_range
    } catch (const std::out_of_range& e) {
        std::cerr << "錯誤: " << e.what() << "\n";
    }

    // 方法 3：front() 和 back()
    std::cout << "第一個: " << vec.front() << "\n";  // 10
    std::cout << "最後一個: " << vec.back() << "\n";  // 50

    // 方法 4：取得底層陣列指標
    int* ptr = vec.data();
    std::cout << ptr[2] << "\n";  // 30

    return 0;
}
```

### 遍歷元素

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};

    // 方法 1：範圍 for（C++11，最推薦）
    for (int val : vec) {
        std::cout << val << " ";
    }
    std::cout << "\n";

    // 方法 2：迭代器
    for (auto it = vec.begin(); it != vec.end(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << "\n";

    // 方法 3：索引
    for (size_t i = 0; i < vec.size(); ++i) {
        std::cout << vec[i] << " ";
    }
    std::cout << "\n";

    // 方法 4：反向遍歷
    for (auto it = vec.rbegin(); it != vec.rend(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << "\n";

    return 0;
}
```

## 迭代器支持

### 迭代器類型

`vector` 提供**隨機存取迭代器**（Random Access Iterator），支援所有迭代器操作：

```cpp
std::vector<int> vec = {1, 2, 3, 4, 5};

auto it = vec.begin();
it + 3;     // ✓ 跳躍存取
it += 2;    // ✓ 複合賦值
it - it2;   // ✓ 迭代器相減
it < it2;   // ✓ 比較運算
it[2];      // ✓ 下標運算
```

### 迭代器失效問題

**什麼情況下迭代器會失效？**

1. **重新配置**（`push_back`, `insert` 導致 `capacity` 增加）
   ```cpp
   std::vector<int> vec = {1, 2, 3};
   auto it = vec.begin();

   vec.push_back(4);  // 可能觸發重新配置
   // it 現在失效！使用 it 是未定義行為
   ```

2. **插入元素**（插入位置之後的迭代器失效）
   ```cpp
   std::vector<int> vec = {1, 2, 3};
   auto it = vec.begin() + 1;  // 指向 2

   vec.insert(vec.begin(), 0);  // 在開頭插入
   // it 現在失效！
   ```

3. **刪除元素**（刪除位置及之後的迭代器失效）
   ```cpp
   std::vector<int> vec = {1, 2, 3, 4};
   auto it = vec.begin() + 2;  // 指向 3

   vec.erase(vec.begin() + 1);  // 刪除 2
   // it 現在失效！
   ```

**安全做法**：

```cpp
std::vector<int> vec = {1, 2, 3, 4, 5};

// 正確：使用 erase 的回傳值
for (auto it = vec.begin(); it != vec.end(); ) {
    if (*it % 2 == 0) {
        it = vec.erase(it);  // erase 回傳下一個有效迭代器
    } else {
        ++it;
    }
}
```

## 常見陷阱與注意事項

### 1. 重新配置導致的迭代器失效

```cpp
// ❌ 錯誤：迭代器失效
std::vector<int> vec = {1, 2, 3};
auto it = vec.begin();

for (int i = 0; i < 1000; ++i) {
    vec.push_back(i);  // 可能觸發重新配置
}

std::cout << *it << "\n";  // 未定義行為！

// ✅ 正確：預留足夠容量
std::vector<int> vec2 = {1, 2, 3};
vec2.reserve(1003);  // 確保不會重新配置
auto it2 = vec2.begin();

for (int i = 0; i < 1000; ++i) {
    vec2.push_back(i);
}

std::cout << *it2 << "\n";  // 安全
```

### 2. 容量浪費

```cpp
// ❌ 錯誤：大量容量浪費
std::vector<int> vec;
vec.reserve(1000000);  // 預留 100 萬
vec.push_back(1);      // 只用 1 個
// 浪費：(1000000 - 1) * 4 bytes ≈ 4 MB

// ✅ 正確：使用 shrink_to_fit 釋放多餘容量
vec.shrink_to_fit();  // 釋放未使用的容量
```

### 3. 中間插入/刪除的性能問題

```cpp
// ❌ 錯誤：頻繁在頭部插入
std::vector<int> vec;
for (int i = 0; i < 10000; ++i) {
    vec.insert(vec.begin(), i);  // O(n) × 10000 次 = O(n²)
}

// ✅ 正確：使用 deque 或先插入再反轉
std::deque<int> dq;  // 兩端插入都是 O(1)
for (int i = 0; i < 10000; ++i) {
    dq.push_front(i);
}

// 或者
std::vector<int> vec2;
for (int i = 0; i < 10000; ++i) {
    vec2.push_back(i);  // O(1)
}
std::reverse(vec2.begin(), vec2.end());  // O(n)，總共 O(n)
```

### 4. 儲存 bool 的特殊情況

```cpp
// ⚠️ 注意：vector<bool> 是特殊化版本
std::vector<bool> vec = {true, false, true};

// 問題：operator[] 回傳的不是 bool&
auto val = vec[0];  // val 的型別是 std::vector<bool>::reference，不是 bool&

// 解決方案：使用 std::vector<char> 或 std::deque<bool>
std::vector<char> vec2 = {1, 0, 1};  // 用 char 代替 bool
```

## 實際應用場景

### 場景 1：動態增長的資料集合

```cpp
#include <vector>
#include <iostream>
#include <string>

struct Student {
    std::string name;
    int score;
};

int main() {
    std::vector<Student> students;
    students.reserve(100);  // 預估會有 100 個學生

    // 動態加入學生資料
    students.push_back({"Alice", 95});
    students.push_back({"Bob", 87});
    students.push_back({"Charlie", 92});

    // 計算平均分數
    int total = 0;
    for (const auto& student : students) {
        total += student.score;
    }

    double average = static_cast<double>(total) / students.size();
    std::cout << "平均分數: " << average << "\n";

    return 0;
}
```

### 場景 2：需要隨機存取的情境

```cpp
#include <vector>
#include <iostream>
#include <cstdlib>
#include <ctime>

int main() {
    std::srand(std::time(nullptr));

    // 模擬遊戲中的隨機存取
    std::vector<int> scores(100);

    // 初始化分數
    for (size_t i = 0; i < scores.size(); ++i) {
        scores[i] = std::rand() % 100;
    }

    // 隨機存取某個玩家的分數
    size_t playerId = std::rand() % 100;
    std::cout << "玩家 " << playerId << " 的分數: "
              << scores[playerId] << "\n";

    return 0;
}
```

### 場景 3：與演算法配合使用

```cpp
#include <vector>
#include <algorithm>
#include <iostream>

int main() {
    std::vector<int> vec = {5, 2, 8, 1, 9, 3, 7};

    // 排序
    std::sort(vec.begin(), vec.end());

    // 二分查找（必須先排序）
    bool found = std::binary_search(vec.begin(), vec.end(), 7);
    std::cout << "找到 7: " << (found ? "是" : "否") << "\n";

    // 查找範圍
    auto lower = std::lower_bound(vec.begin(), vec.end(), 5);
    auto upper = std::upper_bound(vec.begin(), vec.end(), 8);

    std::cout << "5 到 8 之間的元素: ";
    for (auto it = lower; it != upper; ++it) {
        std::cout << *it << " ";
    }
    std::cout << "\n";

    // 移除重複元素
    auto last = std::unique(vec.begin(), vec.end());
    vec.erase(last, vec.end());

    return 0;
}
```

## 與其他容器的選擇

| 需求 | 推薦容器 | 理由 |
|------|---------|------|
| 需要頻繁隨機存取 | `vector` | O(1) 隨機存取 |
| 需要頻繁尾部插入/刪除 | `vector` | O(1) 均攤時間 |
| 需要頻繁頭部插入/刪除 | `deque` | `vector` 的 O(n) vs `deque` 的 O(1) |
| 需要頻繁中間插入/刪除 | `list` | `vector` 的 O(n) vs `list` 的 O(1) |
| 大小固定，不需動態配置 | `array` | 無額外開銷 |
| 需要保持元素有序並快速查找 | `set` | 自動排序，O(log n) 查找 |

**一般原則**：
- **預設使用 `vector`**：90% 的情況下它是最佳選擇
- **記憶體連續性**帶來的快取友善性，通常勝過理論上的時間複雜度優勢

## 實務建議

### DO（應該做的）

1. **預留容量以避免重新配置**
   ```cpp
   std::vector<int> vec;
   vec.reserve(1000);  // 如果知道大概大小，先 reserve
   for (int i = 0; i < 1000; ++i) {
       vec.push_back(i);
   }
   ```

2. **使用 `emplace_back` 代替 `push_back`（C++11）**
   ```cpp
   struct Point {
       int x, y;
       Point(int x, int y) : x(x), y(y) {}
   };

   std::vector<Point> points;
   points.emplace_back(1, 2);  // 直接建構，避免複製
   // points.push_back(Point(1, 2));  // 需要先建構臨時物件
   ```

3. **使用範圍 for 遍歷**
   ```cpp
   std::vector<int> vec = {1, 2, 3, 4, 5};
   for (const auto& val : vec) {  // const auto& 避免複製
       std::cout << val << "\n";
   }
   ```

4. **刪除元素時使用 erase-remove idiom**
   ```cpp
   vec.erase(std::remove(vec.begin(), vec.end(), 3), vec.end());
   ```

5. **傳遞 vector 給函式時使用 const 引用**
   ```cpp
   void printVector(const std::vector<int>& vec) {  // 避免複製
       for (int val : vec) {
           std::cout << val << " ";
       }
   }
   ```

### DON'T（不應該做的）

1. **不要忘記檢查 `empty()` 再使用 `front()` 或 `back()`**
   ```cpp
   // ❌ 錯誤
   std::vector<int> vec;
   int first = vec.front();  // 未定義行為！

   // ✅ 正確
   if (!vec.empty()) {
       int first = vec.front();
   }
   ```

2. **不要在迭代時修改容器大小**
   ```cpp
   // ❌ 錯誤
   for (auto it = vec.begin(); it != vec.end(); ++it) {
       if (*it % 2 == 0) {
           vec.erase(it);  // 迭代器失效！
       }
   }

   // ✅ 正確
   for (auto it = vec.begin(); it != vec.end(); ) {
       if (*it % 2 == 0) {
           it = vec.erase(it);
       } else {
           ++it;
       }
   }
   ```

3. **不要過度使用 `shrink_to_fit()`**
   ```cpp
   // ❌ 錯誤：每次操作後都 shrink
   vec.push_back(1);
   vec.shrink_to_fit();  // 可能觸發重新配置
   vec.push_back(2);
   vec.shrink_to_fit();  // 又重新配置

   // ✅ 正確：只在確定不再增長時才 shrink
   // ... 所有操作完成後 ...
   vec.shrink_to_fit();
   ```

4. **不要儲存指向 vector 元素的指標或引用（長期）**
   ```cpp
   // ❌ 錯誤
   std::vector<int> vec = {1, 2, 3};
   int* ptr = &vec[0];
   vec.push_back(4);  // 可能重新配置
   std::cout << *ptr;  // 未定義行為！
   ```

5. **不要使用 `vector<bool>`（除非真的需要節省空間）**
   ```cpp
   // ❌ 不推薦
   std::vector<bool> flags;

   // ✅ 推薦
   std::vector<char> flags;  // 或 std::vector<int>
   std::deque<bool> flags2;  // 如果需要 bool 語意
   ```

## 小結

**vector 的核心特性：**

1. **動態陣列**：自動管理記憶體，可動態增長
2. **連續記憶體**：快取友善，隨機存取 O(1)
3. **尾部操作高效**：`push_back` 和 `pop_back` 均攤 O(1)
4. **容量策略**：`capacity >= size`，重新配置時成倍增長

**適用場景：**
- 需要隨機存取的場景（90% 的情況）
- 元素數量未知但持續增長
- 需要與 STL 演算法配合使用
- 需要記憶體連續性的場景

**注意事項：**
- 重新配置會使迭代器失效
- 中間插入/刪除效率低
- 使用 `reserve()` 避免不必要的重新配置
- 儲存 `bool` 時考慮使用 `vector<char>` 代替

**記住：`vector` 是 C++ 中最常用的容器，當不確定用哪個容器時，優先選擇 `vector`！**
