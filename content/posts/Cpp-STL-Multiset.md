+++
date = '2025-12-22T13:30:00+08:00'
draft = false
title = '[C++] STL Multiset（多重集合容器）'
categories = ['C++ Notes']
tags = ['C++', 'STL', 'Container']
+++

## 什麼是 Multiset？

`std::multiset` 是 C++ STL 中的**關聯容器**，儲存**允許重複且自動排序**的元素集合。

**與其他容器的主要區別**：
- 與 `set`：multiset 允許重複元素，set 的元素唯一
- 與 `multimap`：multiset 只儲存鍵，multimap 儲存鍵值對
- 與 `unordered_multiset`：multiset 有序（紅黑樹），unordered_multiset 無序（哈希表）

**適用場景**：
- 需要儲存重複元素並自動排序
- 需要快速查找、插入、刪除（O(log n)）
- 需要統計元素出現次數
- 需要範圍查詢

## 內部實現原理

### 底層數據結構

`std::multiset` 使用**紅黑樹**實現，允許重複元素。

### 記憶體布局示意圖

```
std::multiset<int> mySet = {5, 2, 8, 2, 9, 5, 1};

紅黑樹結構（示意）：
                ┌───┐
                │ 5 │ (黑)
                └───┘
               /     \
          ┌───┐     ┌───┐
          │ 2 │     │ 8 │
          └───┘     └───┘
         /   \         \
    ┌───┐ ┌───┐     ┌───┐
    │ 1 │ │ 2 │     │ 9 │
    └───┘ └───┘     └───┘
              \
          ┌───┐
          │ 5 │
          └───┘

中序遍歷（自動排序，保留重複）：1, 2, 2, 5, 5, 8, 9
```

## 基本使用

### 頭文件和宣告

```cpp
#include <set>
#include <iostream>

std::multiset<int> mySet;
std::multiset<int> mySet2 = {1, 2, 2, 3, 3, 3};
```

### 構造方法

```cpp
#include <set>
#include <iostream>

int main() {
    // 預設構造
    std::multiset<int> set1;

    // 初始化列表（允許重複）
    std::multiset<int> set2 = {5, 2, 8, 2, 9, 5, 1};
    // set2: {1, 2, 2, 5, 5, 8, 9}

    // 拷貝構造
    std::multiset<int> set3(set2);

    // 範圍構造
    int arr[] = {3, 1, 4, 1, 5, 9, 2, 6, 5};
    std::multiset<int> set4(arr, arr + 9);

    // 自訂比較函式（降序）
    std::multiset<int, std::greater<int>> set5 = {1, 2, 2, 3};
    // set5: {3, 2, 2, 1}

    return 0;
}
```

### 常用成員函式

| 函式 | 功能 | 時間複雜度 |
|------|------|-----------|
| `insert(val)` | 插入元素（總是成功） | O(log n) |
| `emplace(args...)` | 原地構造並插入 | O(log n) |
| `erase(val)` | 刪除所有指定值 | O(log n + k) |
| `find(val)` | 查找元素，返回第一個匹配 | O(log n) |
| `count(val)` | 計算元素出現次數 | O(log n + k) |
| `equal_range(val)` | 返回等於 val 的範圍 | O(log n) |
| `lower_bound(val)` | 返回 >= val 的第一個元素 | O(log n) |
| `upper_bound(val)` | 返回 > val 的第一個元素 | O(log n) |
| `size()` | 返回元素個數 | O(1) |
| `clear()` | 清空所有元素 | O(n) |

## 常見操作示例

### 插入元素

```cpp
#include <set>
#include <iostream>

int main() {
    std::multiset<int> mySet;

    // 插入元素（允許重複）
    mySet.insert(10);
    mySet.insert(10);  // 重複插入，成功
    mySet.insert(20);
    mySet.insert(10);  // 再次插入

    // 批次插入
    mySet.insert({30, 30, 40});

    // 輸出（自動排序）
    std::cout << "mySet: ";
    for (int val : mySet) {
        std::cout << val << " ";
    }
    std::cout << std::endl;

    std::cout << "大小: " << mySet.size() << std::endl;

    return 0;
}
```

**輸出**：
```
mySet: 10 10 10 20 30 30 40
大小: 7
```

### 統計元素出現次數

```cpp
#include <set>
#include <iostream>

int main() {
    std::multiset<int> mySet = {1, 2, 2, 3, 3, 3, 4, 4, 4, 4};

    // 統計出現次數
    std::cout << "元素出現次數:" << std::endl;
    std::cout << "1 出現 " << mySet.count(1) << " 次" << std::endl;
    std::cout << "2 出現 " << mySet.count(2) << " 次" << std::endl;
    std::cout << "3 出現 " << mySet.count(3) << " 次" << std::endl;
    std::cout << "4 出現 " << mySet.count(4) << " 次" << std::endl;
    std::cout << "5 出現 " << mySet.count(5) << " 次" << std::endl;

    return 0;
}
```

**輸出**：
```
元素出現次數:
1 出現 1 次
2 出現 2 次
3 出現 3 次
4 出現 4 次
5 出現 0 次
```

### 刪除元素

```cpp
#include <set>
#include <iostream>

int main() {
    std::multiset<int> mySet = {1, 2, 2, 3, 3, 3, 4};

    std::cout << "初始: ";
    for (int val : mySet) {
        std::cout << val << " ";
    }
    std::cout << std::endl;

    // 方法1：刪除所有指定值
    size_t removed = mySet.erase(3);
    std::cout << "刪除了 " << removed << " 個 3" << std::endl;

    // 方法2：通過迭代器刪除單個元素
    auto it = mySet.find(2);
    if (it != mySet.end()) {
        mySet.erase(it);  // 只刪除一個 2
    }

    std::cout << "刪除後: ";
    for (int val : mySet) {
        std::cout << val << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

**輸出**：
```
初始: 1 2 2 3 3 3 4
刪除了 3 個 3
刪除後: 1 2 4
```

### 使用 equal_range

```cpp
#include <set>
#include <iostream>

int main() {
    std::multiset<int> mySet = {10, 20, 20, 20, 30, 40};

    // 查找所有值為 20 的元素
    auto range = mySet.equal_range(20);

    std::cout << "值為 20 的元素個數: "
              << std::distance(range.first, range.second) << std::endl;

    std::cout << "這些元素: ";
    for (auto it = range.first; it != range.second; ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

**輸出**：
```
值為 20 的元素個數: 3
這些元素: 20 20 20
```

## 實際應用場景

### 場景 1：成績排序（允許同分）

```cpp
#include <set>
#include <iostream>

int main() {
    std::multiset<int, std::greater<int>> scores;  // 降序排列

    // 錄入成績
    scores.insert({95, 87, 92, 87, 95, 88, 92, 90});

    std::cout << "成績排名（從高到低）:" << std::endl;
    int rank = 1;
    for (int score : scores) {
        std::cout << "第 " << rank++ << " 名: " << score << " 分" << std::endl;
    }

    std::cout << "\n統計:" << std::endl;
    std::cout << "95 分有 " << scores.count(95) << " 人" << std::endl;
    std::cout << "87 分有 " << scores.count(87) << " 人" << std::endl;

    return 0;
}
```

**輸出**：
```
成績排名（從高到低）:
第 1 名: 95 分
第 2 名: 95 分
第 3 名: 92 分
第 4 名: 92 分
第 5 名: 90 分
第 6 名: 88 分
第 7 名: 87 分
第 8 名: 87 分

統計:
95 分有 2 人
87 分有 2 人
```

### 場景 2：中位數查詢

```cpp
#include <set>
#include <iostream>

class MedianFinder {
private:
    std::multiset<int> data;

public:
    void addNum(int num) {
        data.insert(num);
    }

    double findMedian() {
        if (data.empty()) return 0.0;

        size_t size = data.size();
        auto it = data.begin();
        std::advance(it, size / 2);

        if (size % 2 == 1) {
            return *it;
        } else {
            double val1 = *it;
            --it;
            double val2 = *it;
            return (val1 + val2) / 2.0;
        }
    }

    void print() const {
        std::cout << "資料: ";
        for (int val : data) {
            std::cout << val << " ";
        }
        std::cout << std::endl;
    }
};

int main() {
    MedianFinder mf;

    mf.addNum(5);
    mf.addNum(2);
    mf.addNum(8);
    mf.addNum(2);
    mf.addNum(9);

    mf.print();
    std::cout << "中位數: " << mf.findMedian() << std::endl;

    return 0;
}
```

**輸出**：
```
資料: 2 2 5 8 9
中位數: 5
```

### 場景 3：滑動視窗中位數

```cpp
#include <set>
#include <vector>
#include <iostream>

std::vector<double> medianSlidingWindow(const std::vector<int>& nums, int k) {
    std::vector<double> result;
    std::multiset<int> window;

    for (int i = 0; i < nums.size(); ++i) {
        window.insert(nums[i]);

        // 維持視窗大小
        if (window.size() > k) {
            window.erase(window.find(nums[i - k]));
        }

        // 計算中位數
        if (window.size() == k) {
            auto it = window.begin();
            std::advance(it, k / 2);

            if (k % 2 == 1) {
                result.push_back(*it);
            } else {
                double val1 = *it;
                --it;
                double val2 = *it;
                result.push_back((val1 + val2) / 2.0);
            }
        }
    }

    return result;
}

int main() {
    std::vector<int> nums = {1, 3, -1, -3, 5, 3, 6, 7};
    int k = 3;

    auto medians = medianSlidingWindow(nums, k);

    std::cout << "滑動視窗中位數 (k=" << k << "):" << std::endl;
    for (double median : medians) {
        std::cout << median << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

**輸出**：
```
滑動視窗中位數 (k=3):
1 -1 -1 3 5 6
```

## 與其他容器的選擇

### 對比表格

| 特性 | multiset | set | unordered_multiset | vector |
|------|----------|-----|-------------------|--------|
| 唯一性 | ❌ 可重複 | ✅ 唯一 | ❌ 可重複 | ❌ 可重複 |
| 有序性 | ✅ 有序 | ✅ 有序 | ❌ 無序 | 插入順序 |
| 查找 | O(log n) | O(log n) | **O(1) 平均** | O(n) |
| 統計次數 | O(log n + k) | O(log n) | **O(1) 平均** | O(n) |

### 選擇建議

**選擇 multiset 當**：
- ✅ 需要儲存重複元素
- ✅ 需要自動排序
- ✅ 需要統計元素出現次數

**選擇 set 當**：
- ✅ 元素必須唯一
- ✅ 需要自動排序

**選擇 unordered_multiset 當**：
- ✅ 只需要快速查找（不需要排序）

**選擇 vector 當**：
- ✅ 需要保持插入順序
- ✅ 需要隨機訪問

## 實務建議

### DO（應該做的）

#### 1. ✅ 使用 multiset 統計頻率並排序

```cpp
std::vector<int> data = {5, 2, 8, 2, 1, 5, 3};

// ✅ 推薦：使用 multiset 自動排序
std::multiset<int> sorted(data.begin(), data.end());

// 輸出排序結果和頻率
for (auto it = sorted.begin(); it != sorted.end(); ) {
    int val = *it;
    int count = sorted.count(val);
    std::cout << val << " 出現 " << count << " 次" << std::endl;
    std::advance(it, count);  // 跳過重複元素
}
```

#### 2. ✅ 使用 equal_range 遍歷相同元素

```cpp
std::multiset<int> mySet = {1, 2, 2, 2, 3};

// ✅ 推薦：使用 equal_range
auto range = mySet.equal_range(2);
for (auto it = range.first; it != range.second; ++it) {
    // 處理所有 2
}
```

### DON'T（不應該做的）

#### 1. ❌ 不要忘記 erase(val) 會刪除所有相同值

```cpp
std::multiset<int> mySet = {1, 2, 2, 2, 3};

// ❌ 注意：刪除所有 2
mySet.erase(2);  // 全部刪除！

// ✅ 如果只想刪除一個，使用迭代器
auto it = mySet.find(2);
if (it != mySet.end()) {
    mySet.erase(it);  // 只刪除一個
}
```

#### 2. ❌ 不要在元素唯一時使用 multiset

```cpp
// ❌ 浪費：元素唯一卻用 multiset
std::multiset<int> unique;  // 多餘的開銷

// ✅ 推薦：使用 set
std::set<int> unique;
```

## 小結

### 核心概念總結

1. **允許重複**：相同元素可以多次插入
2. **自動排序**：元素按升序排列
3. **統計頻率**：count() 可統計出現次數
4. **範圍查詢**：equal_range() 查找所有相同元素
5. **適合場景**：需要排序且允許重複的集合

### 一句話總結

**`std::multiset` 是基於紅黑樹的有序集合容器，允許重複元素並自動排序，提供 O(log n) 性能和頻率統計能力，適合需要排序且允許重複的場景。**
