+++
date = '2025-12-22T12:30:00+08:00'
draft = false
title = '[C++] STL Set（集合容器）'
categories = ['C++ Notes']
tags = ['C++', 'STL', 'Container']
+++

## 什麼是 Set？

`std::set` 是 C++ STL 中的**關聯容器**，儲存**唯一且自動排序**的元素集合。

**與其他容器的主要區別**：
- 與 `map`：set 只儲存鍵，map 儲存鍵值對
- 與 `multiset`：set 的元素唯一，multiset 允許重複元素
- 與 `unordered_set`：set 有序（紅黑樹），unordered_set 無序（哈希表）
- 與 `vector`：set 自動去重和排序，vector 保持插入順序

**適用場景**：
- 需要儲存唯一元素
- 需要自動排序
- 需要快速查找、插入、刪除（O(log n)）
- 需要範圍查詢
- 需要集合運算（交集、聯集、差集）

## 內部實現原理

### 底層數據結構

`std::set` 使用**紅黑樹**（Red-Black Tree）實現，與 `std::map` 相同，但只儲存鍵。

### 記憶體布局示意圖

```
std::set<int> mySet = {3, 1, 5, 2, 4};

紅黑樹結構（示意）：
                ┌───┐
                │ 3 │ (黑)
                └───┘
               /     \
          ┌───┐     ┌───┐
          │ 1 │     │ 5 │ (黑)
          └───┘     └───┘
              \     /
          ┌───┐ ┌───┐
          │ 2 │ │ 4 │ (紅)
          └───┘ └───┘

中序遍歷結果（自動排序）：1, 2, 3, 4, 5

每個節點包含：
- 元素值
- 左子節點指標
- 右子節點指標
- 父節點指標
- 顏色標記（紅/黑）
```

**特點**：
1. **自動排序**：元素按照 `<` 運算子排序
2. **自動去重**：重複插入相同元素會失敗
3. **平衡性**：樹高保持在 O(log n)
4. **非連續記憶體**：節點散布在堆記憶體中

## 基本使用

### 頭文件和宣告

```cpp
#include <set>
#include <iostream>

std::set<int> mySet;                    // 空 set
std::set<int> mySet2 = {1, 2, 3, 4, 5}; // 初始化列表
std::set<std::string> mySet3;           // 字串 set
```

### 構造方法

```cpp
#include <set>
#include <iostream>

int main() {
    // 預設構造
    std::set<int> set1;

    // 初始化列表
    std::set<int> set2 = {5, 2, 8, 1, 9, 2};  // 重複的 2 會被忽略
    // set2: {1, 2, 5, 8, 9}

    // 拷貝構造
    std::set<int> set3(set2);

    // 移動構造
    std::set<int> set4(std::move(set3));

    // 範圍構造
    int arr[] = {3, 1, 4, 1, 5};
    std::set<int> set5(arr, arr + 5);
    // set5: {1, 3, 4, 5}（重複的 1 被去除）

    // 自訂比較函式（降序）
    std::set<int, std::greater<int>> set6 = {1, 2, 3, 4, 5};
    // set6: {5, 4, 3, 2, 1}

    return 0;
}
```

### 常用成員函式

| 函式 | 功能 | 時間複雜度 |
|------|------|-----------|
| `insert(val)` | 插入元素 | O(log n) |
| `emplace(args...)` | 原地構造並插入 | O(log n) |
| `erase(val)` | 刪除元素 | O(log n) |
| `find(val)` | 查找元素，返回迭代器 | O(log n) |
| `count(val)` | 計算元素出現次數（0 或 1） | O(log n) |
| `lower_bound(val)` | 返回 >= val 的第一個元素 | O(log n) |
| `upper_bound(val)` | 返回 > val 的第一個元素 | O(log n) |
| `equal_range(val)` | 返回等於 val 的範圍 | O(log n) |
| `size()` | 返回元素個數 | O(1) |
| `empty()` | 判斷是否為空 | O(1) |
| `clear()` | 清空所有元素 | O(n) |

## 性能分析

### 時間複雜度表格

| 操作 | set | unordered_set | vector |
|------|-----|---------------|--------|
| 查找 | **O(log n)** | O(1) 平均 | O(n) |
| 插入 | **O(log n)** | O(1) 平均 | O(n) |
| 刪除 | **O(log n)** | O(1) 平均 | O(n) |
| 有序遍歷 | **O(n)** | ❌ | 需排序 O(n log n) |
| 範圍查詢 | **O(log n + k)** | ❌ | O(n) |

### 空間複雜度分析

**記憶體開銷**：
```cpp
// 64 位元系統
sizeof(set<int>::node):
  value:  4 bytes
  left:   8 bytes (指標)
  right:  8 bytes (指標)
  parent: 8 bytes (指標)
  color:  1 byte
  總計：  ~32 bytes（含對齊）

100 個元素的 set<int> ≈ 3200 bytes
100 個元素的 vector<int> ≈ 400 bytes

set 記憶體開銷約為 vector 的 8 倍
```

## 常見操作示例

### 插入元素

```cpp
#include <set>
#include <iostream>

int main() {
    std::set<int> mySet;

    // 方法1：使用 insert()
    auto result1 = mySet.insert(10);
    std::cout << "插入 10: " << (result1.second ? "成功" : "失敗") << std::endl;

    auto result2 = mySet.insert(10);  // 重複插入
    std::cout << "再次插入 10: " << (result2.second ? "成功" : "失敗") << std::endl;

    // 方法2：批次插入
    mySet.insert({20, 30, 40, 20});  // 20 重複，只會插入一次

    // 方法3：使用 emplace()
    mySet.emplace(50);

    // 輸出（自動排序）
    std::cout << "mySet: ";
    for (int val : mySet) {
        std::cout << val << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

**輸出**：
```
插入 10: 成功
再次插入 10: 失敗
mySet: 10 20 30 40 50
```

### 刪除元素

```cpp
#include <set>
#include <iostream>

int main() {
    std::set<int> mySet = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

    // 方法1：通過值刪除
    size_t count = mySet.erase(5);
    std::cout << "刪除了 " << count << " 個元素" << std::endl;

    // 方法2：通過迭代器刪除
    auto it = mySet.find(3);
    if (it != mySet.end()) {
        mySet.erase(it);
    }

    // 方法3：刪除範圍
    auto first = mySet.find(6);
    auto last = mySet.find(9);
    mySet.erase(first, last);  // 刪除 [6, 9)，即 6, 7, 8

    std::cout << "剩餘元素: ";
    for (int val : mySet) {
        std::cout << val << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

**輸出**：
```
刪除了 1 個元素
剩餘元素: 1 2 4 9 10
```

### 查找元素

```cpp
#include <set>
#include <iostream>

int main() {
    std::set<int> mySet = {10, 20, 30, 40, 50};

    // 方法1：使用 find()
    auto it = mySet.find(30);
    if (it != mySet.end()) {
        std::cout << "找到: " << *it << std::endl;
    }

    // 方法2：使用 count()
    if (mySet.count(40) > 0) {
        std::cout << "40 存在" << std::endl;
    }

    if (mySet.count(100) == 0) {
        std::cout << "100 不存在" << std::endl;
    }

    // C++20: contains()
    // if (mySet.contains(30)) {
    //     std::cout << "30 存在" << std::endl;
    // }

    return 0;
}
```

**輸出**：
```
找到: 30
40 存在
100 不存在
```

### 遍歷容器

```cpp
#include <set>
#include <iostream>

int main() {
    std::set<int> mySet = {5, 2, 8, 1, 9};

    // 方法1：範圍 for（自動排序）
    std::cout << "正向遍歷: ";
    for (int val : mySet) {
        std::cout << val << " ";
    }
    std::cout << std::endl;

    // 方法2：迭代器
    std::cout << "迭代器: ";
    for (auto it = mySet.begin(); it != mySet.end(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    // 方法3：反向遍歷
    std::cout << "反向遍歷: ";
    for (auto it = mySet.rbegin(); it != mySet.rend(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

**輸出**：
```
正向遍歷: 1 2 5 8 9
迭代器: 1 2 5 8 9
反向遍歷: 9 8 5 2 1
```

### 範圍查詢

```cpp
#include <set>
#include <iostream>

int main() {
    std::set<int> mySet = {10, 20, 30, 40, 50, 60, 70, 80};

    // lower_bound：>= val 的第一個元素
    auto lb = mySet.lower_bound(35);
    if (lb != mySet.end()) {
        std::cout << "lower_bound(35): " << *lb << std::endl;  // 40
    }

    // upper_bound：> val 的第一個元素
    auto ub = mySet.upper_bound(50);
    if (ub != mySet.end()) {
        std::cout << "upper_bound(50): " << *ub << std::endl;  // 60
    }

    // 查詢範圍 [30, 60)
    std::cout << "\n範圍 [30, 60) 的元素: ";
    auto start = mySet.lower_bound(30);
    auto end = mySet.lower_bound(60);
    for (auto it = start; it != end; ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

**輸出**：
```
lower_bound(35): 40
upper_bound(50): 60

範圍 [30, 60) 的元素: 30 40 50
```

### 集合運算

```cpp
#include <set>
#include <algorithm>
#include <iostream>
#include <iterator>

int main() {
    std::set<int> set1 = {1, 2, 3, 4, 5};
    std::set<int> set2 = {4, 5, 6, 7, 8};

    // 聯集（Union）
    std::set<int> unionSet;
    std::set_union(set1.begin(), set1.end(),
                   set2.begin(), set2.end(),
                   std::inserter(unionSet, unionSet.begin()));

    std::cout << "聯集: ";
    for (int val : unionSet) {
        std::cout << val << " ";
    }
    std::cout << std::endl;

    // 交集（Intersection）
    std::set<int> intersectionSet;
    std::set_intersection(set1.begin(), set1.end(),
                          set2.begin(), set2.end(),
                          std::inserter(intersectionSet, intersectionSet.begin()));

    std::cout << "交集: ";
    for (int val : intersectionSet) {
        std::cout << val << " ";
    }
    std::cout << std::endl;

    // 差集（Difference）
    std::set<int> differenceSet;
    std::set_difference(set1.begin(), set1.end(),
                        set2.begin(), set2.end(),
                        std::inserter(differenceSet, differenceSet.begin()));

    std::cout << "差集 (set1 - set2): ";
    for (int val : differenceSet) {
        std::cout << val << " ";
    }
    std::cout << std::endl;

    // 對稱差集
    std::set<int> symDifferenceSet;
    std::set_symmetric_difference(set1.begin(), set1.end(),
                                  set2.begin(), set2.end(),
                                  std::inserter(symDifferenceSet, symDifferenceSet.begin()));

    std::cout << "對稱差集: ";
    for (int val : symDifferenceSet) {
        std::cout << val << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

**輸出**：
```
聯集: 1 2 3 4 5 6 7 8
交集: 4 5
差集 (set1 - set2): 1 2 3
對稱差集: 1 2 3 6 7 8
```

## 迭代器支持

### 迭代器失效規則

| 操作 | 失效範圍 |
|------|----------|
| `insert/emplace` | **無失效** |
| `erase(val)` | **僅被刪除元素的迭代器失效** |
| `clear()` | **所有迭代器失效** |

## 常見陷阱與注意事項

### 1. 不能修改 set 中的元素

```cpp
std::set<int> mySet = {1, 2, 3, 4, 5};

auto it = mySet.find(3);
if (it != mySet.end()) {
    // ❌ 錯誤：不能修改 set 中的元素
    // *it = 10;  // 編譯錯誤！set 的元素是 const

    // ✅ 正確：刪除後重新插入
    mySet.erase(it);
    mySet.insert(10);
}
```

### 2. 重複插入會失敗

```cpp
std::set<int> mySet = {1, 2, 3};

auto result = mySet.insert(2);  // 2 已存在
if (!result.second) {
    std::cout << "插入失敗，元素已存在" << std::endl;
}
```

### 3. 自訂型別必須定義比較運算

```cpp
struct Point {
    int x, y;
};

// ❌ 錯誤：Point 沒有定義 operator<
// std::set<Point> mySet;  // 編譯錯誤！

// ✅ 解決方案：定義 operator<
struct Point {
    int x, y;
    bool operator<(const Point& other) const {
        if (x != other.x) return x < other.x;
        return y < other.y;
    }
};
```

## 實際應用場景

### 場景 1：去重並排序

```cpp
#include <set>
#include <vector>
#include <iostream>

std::vector<int> removeDuplicatesAndSort(const std::vector<int>& input) {
    std::set<int> uniqueSet(input.begin(), input.end());
    return std::vector<int>(uniqueSet.begin(), uniqueSet.end());
}

int main() {
    std::vector<int> data = {5, 2, 8, 2, 9, 1, 5, 3};

    std::cout << "原始資料: ";
    for (int val : data) {
        std::cout << val << " ";
    }
    std::cout << std::endl;

    auto result = removeDuplicatesAndSort(data);

    std::cout << "去重排序後: ";
    for (int val : result) {
        std::cout << val << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

**輸出**：
```
原始資料: 5 2 8 2 9 1 5 3
去重排序後: 1 2 3 5 8 9
```

### 場景 2：檢查子集關係

```cpp
#include <set>
#include <algorithm>
#include <iostream>

bool isSubset(const std::set<int>& subset, const std::set<int>& superset) {
    return std::includes(superset.begin(), superset.end(),
                        subset.begin(), subset.end());
}

int main() {
    std::set<int> setA = {1, 2, 3};
    std::set<int> setB = {1, 2, 3, 4, 5};
    std::set<int> setC = {1, 6};

    std::cout << "A 是 B 的子集: " << (isSubset(setA, setB) ? "是" : "否") << std::endl;
    std::cout << "C 是 B 的子集: " << (isSubset(setC, setB) ? "是" : "否") << std::endl;

    return 0;
}
```

**輸出**：
```
A 是 B 的子集: 是
C 是 B 的子集: 否
```

### 場景 3：訪客記錄系統

```cpp
#include <set>
#include <string>
#include <iostream>

class VisitorLog {
private:
    std::set<std::string> visitors;

public:
    bool checkIn(const std::string& name) {
        auto [it, inserted] = visitors.insert(name);
        if (inserted) {
            std::cout << name << " 第一次訪問" << std::endl;
            return true;
        } else {
            std::cout << name << " 已訪問過" << std::endl;
            return false;
        }
    }

    int getTotalVisitors() const {
        return visitors.size();
    }

    void printAllVisitors() const {
        std::cout << "所有訪客（按字母順序）:" << std::endl;
        for (const auto& name : visitors) {
            std::cout << "  " << name << std::endl;
        }
    }
};

int main() {
    VisitorLog log;

    log.checkIn("Alice");
    log.checkIn("Bob");
    log.checkIn("Alice");  // 重複訪問
    log.checkIn("Charlie");

    std::cout << "\n總訪客數: " << log.getTotalVisitors() << std::endl;
    std::cout << std::endl;

    log.printAllVisitors();

    return 0;
}
```

**輸出**：
```
Alice 第一次訪問
Bob 第一次訪問
Alice 已訪問過
Charlie 第一次訪問

總訪客數: 3

所有訪客（按字母順序）:
  Alice
  Bob
  Charlie
```

## 與其他容器的選擇

### 對比表格

| 特性 | set | unordered_set | multiset | vector |
|------|-----|---------------|----------|--------|
| 有序性 | ✅ 有序 | ❌ 無序 | ✅ 有序 | 插入順序 |
| 唯一性 | ✅ 唯一 | ✅ 唯一 | ❌ 可重複 | ❌ 可重複 |
| 查找 | O(log n) | **O(1) 平均** | O(log n) | O(n) |
| 插入 | O(log n) | **O(1) 平均** | O(log n) | O(n) |
| 範圍查詢 | ✅ | ❌ | ✅ | ❌ |

### 選擇建議

**選擇 set 當**：
- ✅ 需要唯一元素
- ✅ 需要自動排序
- ✅ 需要範圍查詢
- ✅ 需要集合運算

**選擇 unordered_set 當**：
- ✅ 只需要去重（不需要排序）
- ✅ 查找是主要操作

**選擇 multiset 當**：
- ✅ 需要自動排序
- ✅ 允許重複元素

**選擇 vector 當**：
- ✅ 需要保持插入順序
- ✅ 需要隨機訪問

## 實務建議

### DO（應該做的）

#### 1. ✅ 使用 set 自動去重和排序

```cpp
std::vector<int> data = {5, 2, 8, 2, 1, 5};

// ✅ 推薦：使用 set 自動去重和排序
std::set<int> uniqueSorted(data.begin(), data.end());

// ❌ 繁瑣：手動排序和去重
// std::sort(data.begin(), data.end());
// data.erase(std::unique(data.begin(), data.end()), data.end());
```

#### 2. ✅ 檢查 insert 返回值

```cpp
std::set<int> mySet = {1, 2, 3};

// ✅ 推薦：檢查是否插入成功
auto [it, inserted] = mySet.insert(2);
if (!inserted) {
    std::cout << "元素已存在" << std::endl;
}
```

#### 3. ✅ 使用集合運算算法

```cpp
std::set<int> set1 = {1, 2, 3};
std::set<int> set2 = {2, 3, 4};

// ✅ 推薦：使用 STL 算法
std::set<int> result;
std::set_union(set1.begin(), set1.end(),
               set2.begin(), set2.end(),
               std::inserter(result, result.begin()));
```

### DON'T（不應該做的）

#### 1. ❌ 不要嘗試修改元素

```cpp
std::set<int> mySet = {1, 2, 3};

auto it = mySet.find(2);
// ❌ 錯誤：不能修改
// *it = 10;  // 編譯錯誤！
```

#### 2. ❌ 不要在不需要排序時使用 set

```cpp
// ❌ 浪費：只需要去重，不需要排序
std::set<int> unique;  // O(log n) 插入

// ✅ 推薦：使用 unordered_set
std::unordered_set<int> unique;  // O(1) 平均插入
```

## 小結

### 核心概念總結

1. **唯一且有序**：set 自動去重和排序
2. **紅黑樹實現**：O(log n) 操作
3. **元素不可修改**：保證有序性
4. **範圍查詢**：支援 lower_bound、upper_bound
5. **集合運算**：支援交集、聯集、差集

### 一句話總結

**`std::set` 是基於紅黑樹的有序集合容器，自動去重和排序，提供 O(log n) 的插入、刪除、查找性能，適合需要唯一有序元素和集合運算的場景。**
