+++
date = '2025-12-22T14:30:00+08:00'
draft = false
title = '[C++] STL Unordered_multimap（無序多重映射）'
categories = ['C++ Notes']
tags = ['C++', 'STL', 'Container']
+++

## 什麼是 Unordered_multimap？

`std::unordered_multimap` 是允許重複鍵的無序映射容器，使用哈希表實現。

**特點**：無序、允許重複鍵、O(1) 平均性能

## 基本使用

```cpp
#include <unordered_map>
#include <iostream>
#include <string>

int main() {
    std::unordered_multimap<std::string, int> myMap;

    // 插入重複鍵
    myMap.insert({"Alice", 95});
    myMap.insert({"Alice", 92});
    myMap.insert({"Bob", 87});

    // 查找所有相同鍵的值
    auto range = myMap.equal_range("Alice");
    std::cout << "Alice 的分數: ";
    for (auto it = range.first; it != range.second; ++it) {
        std::cout << it->second << " ";
    }

    return 0;
}
```

## 與其他容器比較

| 特性 | unordered_multimap | multimap |
|------|-------------------|----------|
| 有序性 | ❌ | ✅ |
| 允許重複鍵 | ✅ | ✅ |
| 平均查找 | **O(1)** | O(log n) |

## 小結

**適合需要快速查找且允許重複鍵的一對多映射場景。**
