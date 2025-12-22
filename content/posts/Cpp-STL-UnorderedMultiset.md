+++
date = '2025-12-22T14:45:00+08:00'
draft = false
title = '[C++] STL Unordered_multiset（無序多重集合）'
categories = ['C++ Notes']
tags = ['C++', 'STL', 'Container']
+++

## 什麼是 Unordered_multiset？

`std::unordered_multiset` 是允許重複元素的無序集合容器。

**特點**：無序、允許重複、O(1) 平均性能

## 基本使用

```cpp
#include <unordered_set>
#include <iostream>

int main() {
    std::unordered_multiset<int> mySet = {1, 2, 2, 3, 3, 3};

    // 統計出現次數
    std::cout << "2 出現 " << mySet.count(2) << " 次" << std::endl;
    std::cout << "3 出現 " << mySet.count(3) << " 次" << std::endl;

    // 插入重複元素
    mySet.insert(2);
    std::cout << "插入後，2 出現 " << mySet.count(2) << " 次" << std::endl;

    return 0;
}
```

## 小結

**適合需要快速統計頻率且允許重複的場景。**
