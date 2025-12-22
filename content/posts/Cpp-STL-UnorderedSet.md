+++
date = '2025-12-22T14:15:00+08:00'
draft = false
title = '[C++] STL Unordered_set（無序集合容器）'
categories = ['C++ Notes']
tags = ['C++', 'STL', 'Container']
+++

## 什麼是 Unordered_set？

`std::unordered_set` 是 C++11 引入的**無序集合容器**，使用哈希表實現，元素唯一且無序。

**與其他容器的主要區別**：
- 與 `set`：unordered_set 無序，set 有序
- 與 `unordered_multiset`：unordered_set 元素唯一

**適用場景**：需要快速查找且不需要排序的唯一元素集合。

## 基本使用

```cpp
#include <unordered_set>
#include <iostream>

int main() {
    std::unordered_set<int> mySet = {1, 2, 3, 4, 5, 5};  // 重複的 5 被忽略

    // 插入
    mySet.insert(6);

    // 查找
    if (mySet.find(3) != mySet.end()) {
        std::cout << "找到 3" << std::endl;
    }

    // 刪除
    mySet.erase(2);

    // 遍歷（無序）
    for (int val : mySet) {
        std::cout << val << " ";
    }

    return 0;
}
```

## 性能分析

| 操作 | unordered_set | set |
|------|---------------|-----|
| 查找 | **O(1) 平均** | O(log n) |
| 插入 | **O(1) 平均** | O(log n) |
| 刪除 | **O(1) 平均** | O(log n) |

## 實際應用：去重

```cpp
#include <unordered_set>
#include <vector>
#include <iostream>

std::vector<int> removeDuplicates(const std::vector<int>& nums) {
    std::unordered_set<int> seen;
    std::vector<int> result;

    for (int num : nums) {
        if (seen.insert(num).second) {  // 插入成功表示未見過
            result.push_back(num);
        }
    }

    return result;
}

int main() {
    std::vector<int> nums = {1, 2, 3, 2, 4, 1, 5};
    auto result = removeDuplicates(nums);

    for (int val : result) {
        std::cout << val << " ";  // 1 2 3 4 5
    }

    return 0;
}
```

## 小結

**`std::unordered_set` 提供 O(1) 平均性能的無序唯一元素集合，適合快速去重和查找場景。**
