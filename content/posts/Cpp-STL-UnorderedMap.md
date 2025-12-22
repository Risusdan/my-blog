+++
date = '2025-12-22T14:00:00+08:00'
draft = false
title = '[C++] STL Unordered_map（無序映射容器）'
categories = ['C++ Notes']
tags = ['C++', 'STL', 'Container']
+++

## 什麼是 Unordered_map？

`std::unordered_map` 是 C++11 引入的**無序關聯容器**，使用**哈希表**實現，提供平均 O(1) 的查找性能。

**與其他容器的主要區別**：
- 與 `map`：unordered_map 無序（哈希表），map 有序（紅黑樹）
- 與 `unordered_multimap`：unordered_map 鍵唯一，unordered_multimap 允許重複鍵

**適用場景**：
- 需要快速查找、插入、刪除（O(1) 平均）
- 不需要有序性
- 鍵必須唯一
- 鍵類型可以哈希

## 內部實現原理

### 底層數據結構

使用**哈希表**（Hash Table）實現，採用**鏈式法**（chaining）解決哈希衝突。

```
哈希表結構：
buckets:
[0] -> nullptr
[1] -> {key1, val1} -> {key7, val7} -> nullptr
[2] -> nullptr
[3] -> {key3, val3} -> nullptr
[4] -> {key4, val4} -> {key11, val11} -> nullptr
...

當 load_factor 超過閾值時，會進行 rehash（重新分配更大的桶陣列）
```

**關鍵概念**：
- **哈希函式**：將鍵轉換為桶索引
- **負載因子**：元素數量 / 桶數量
- **Rehash**：當負載因子過高時，擴展桶陣列

## 基本使用

```cpp
#include <unordered_map>
#include <iostream>
#include <string>

int main() {
    std::unordered_map<std::string, int> myMap;

    // 插入
    myMap["apple"] = 100;
    myMap.insert({"banana", 200});
    myMap.emplace("cherry", 300);

    // 訪問
    std::cout << "apple: " << myMap["apple"] << std::endl;
    std::cout << "banana: " << myMap.at("banana") << std::endl;

    // 查找
    if (myMap.find("cherry") != myMap.end()) {
        std::cout << "找到 cherry" << std::endl;
    }

    // 遍歷（無序）
    for (const auto& [key, value] : myMap) {
        std::cout << key << ": " << value << std::endl;
    }

    return 0;
}
```

## 性能分析

| 操作 | unordered_map | map |
|------|---------------|-----|
| 查找 | **O(1) 平均**, O(n) 最壞 | O(log n) |
| 插入 | **O(1) 平均**, O(n) 最壞 | O(log n) |
| 刪除 | **O(1) 平均**, O(n) 最壞 | O(log n) |
| 有序遍歷 | ❌ | ✅ |

## 常見陷阱

### 1. 自訂型別需要哈希函式

```cpp
struct Point {
    int x, y;
};

// ❌ 錯誤：Point 沒有哈希函式
// std::unordered_map<Point, int> myMap;

// ✅ 解決方案：提供哈希函式
struct PointHash {
    size_t operator()(const Point& p) const {
        return std::hash<int>()(p.x) ^ (std::hash<int>()(p.y) << 1);
    }
};

struct PointEqual {
    bool operator()(const Point& a, const Point& b) const {
        return a.x == b.x && a.y == b.y;
    }
};

std::unordered_map<Point, int, PointHash, PointEqual> myMap;
```

### 2. 迭代器可能在 rehash 時失效

```cpp
std::unordered_map<int, int> myMap = {{1, 1}, {2, 2}};
auto it = myMap.begin();

// 大量插入可能觸發 rehash
for (int i = 0; i < 1000; ++i) {
    myMap[i] = i;  // rehash 後 it 可能失效
}

// ⚠️ it 可能已失效
```

## 實際應用場景

### 字元頻率統計

```cpp
#include <unordered_map>
#include <string>
#include <iostream>

std::unordered_map<char, int> countFrequency(const std::string& str) {
    std::unordered_map<char, int> freq;
    for (char c : str) {
        freq[c]++;
    }
    return freq;
}

int main() {
    std::string text = "hello world";
    auto freq = countFrequency(text);

    for (const auto& [ch, count] : freq) {
        std::cout << ch << ": " << count << std::endl;
    }

    return 0;
}
```

## 實務建議

### DO（應該做的）

✅ **使用 unordered_map 進行快速查找**
```cpp
// 快取查找
std::unordered_map<std::string, Result> cache;
```

✅ **使用 reserve 預留空間**
```cpp
std::unordered_map<int, int> myMap;
myMap.reserve(1000);  // 避免多次 rehash
```

### DON'T（不應該做的）

❌ **不要在需要有序遍歷時使用**
```cpp
// ❌ 錯誤：unordered_map 是無序的
std::unordered_map<int, int> myMap;
// 遍歷順序不確定

// ✅ 正確：使用 map
std::map<int, int> myMap;
```

## 小結

**`std::unordered_map` 是基於哈希表的無序映射容器，提供平均 O(1) 的插入、刪除、查找性能，適合需要快速查找且不需要有序性的場景。**
