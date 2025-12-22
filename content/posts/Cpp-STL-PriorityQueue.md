+++
date = '2025-12-22T15:30:00+08:00'
draft = false
title = '[C++] STL Priority_queue（優先佇列容器適配器）'
categories = ['C++ Notes']
tags = ['C++', 'STL', 'Container']
+++

## 什麼是 Priority_queue？

`std::priority_queue` 是**容器適配器**，提供**優先順序佇列**，預設為**最大堆**（max heap）。

**特點**：
- 自動按優先順序排序
- 只能訪問堆頂（最大或最小元素）
- 底層預設使用 `vector` 實現堆

## 基本使用

```cpp
#include <queue>
#include <iostream>

int main() {
    // 預設：最大堆
    std::priority_queue<int> maxHeap;

    maxHeap.push(30);
    maxHeap.push(10);
    maxHeap.push(50);
    maxHeap.push(20);

    std::cout << "最大堆頂: " << maxHeap.top() << std::endl;  // 50

    // 逐個彈出（降序）
    while (!maxHeap.empty()) {
        std::cout << maxHeap.top() << " ";
        maxHeap.pop();
    }
    // 輸出: 50 30 20 10

    return 0;
}
```

## 最小堆

```cpp
#include <queue>
#include <vector>
#include <functional>
#include <iostream>

int main() {
    // 最小堆
    std::priority_queue<int, std::vector<int>, std::greater<int>> minHeap;

    minHeap.push(30);
    minHeap.push(10);
    minHeap.push(50);
    minHeap.push(20);

    std::cout << "最小堆頂: " << minHeap.top() << std::endl;  // 10

    // 逐個彈出（升序）
    while (!minHeap.empty()) {
        std::cout << minHeap.top() << " ";
        minHeap.pop();
    }
    // 輸出: 10 20 30 50

    return 0;
}
```

## 常用成員函式

| 函式 | 功能 | 時間複雜度 |
|------|------|-----------|
| `push(val)` | 插入元素 | O(log n) |
| `pop()` | 移除堆頂 | O(log n) |
| `top()` | 訪問堆頂 | O(1) |
| `empty()` | 判斷是否為空 | O(1) |
| `size()` | 返回元素個數 | O(1) |

## 實際應用：前 K 大元素

```cpp
#include <queue>
#include <vector>
#include <iostream>

std::vector<int> topKFrequent(const std::vector<int>& nums, int k) {
    // 使用最小堆維護前 k 大的頻率
    auto cmp = [](const std::pair<int, int>& a, const std::pair<int, int>& b) {
        return a.second > b.second;  // 最小堆
    };

    std::priority_queue<std::pair<int, int>,
                        std::vector<std::pair<int, int>>,
                        decltype(cmp)> minHeap(cmp);

    // 統計頻率（簡化示例）
    std::unordered_map<int, int> freq = {{1, 3}, {2, 2}, {3, 1}};

    for (const auto& [num, count] : freq) {
        minHeap.push({num, count});
        if (minHeap.size() > k) {
            minHeap.pop();
        }
    }

    std::vector<int> result;
    while (!minHeap.empty()) {
        result.push_back(minHeap.top().first);
        minHeap.pop();
    }

    return result;
}

int main() {
    std::vector<int> nums = {1, 1, 1, 2, 2, 3};
    int k = 2;

    auto result = topKFrequent(nums, k);

    std::cout << "前 " << k << " 高頻元素: ";
    for (int num : result) {
        std::cout << num << " ";
    }

    return 0;
}
```

## 自訂比較函式

```cpp
#include <queue>
#include <iostream>

struct Task {
    int priority;
    std::string name;

    Task(int p, const std::string& n) : priority(p), name(n) {}
};

// 自訂比較（優先級高的在前）
struct TaskCompare {
    bool operator()(const Task& a, const Task& b) const {
        return a.priority < b.priority;  // 注意：小於號表示最大堆
    }
};

int main() {
    std::priority_queue<Task, std::vector<Task>, TaskCompare> taskQueue;

    taskQueue.push(Task(3, "低優先級"));
    taskQueue.push(Task(10, "高優先級"));
    taskQueue.push(Task(5, "中優先級"));

    while (!taskQueue.empty()) {
        auto task = taskQueue.top();
        std::cout << "執行: " << task.name
                  << " (優先級: " << task.priority << ")" << std::endl;
        taskQueue.pop();
    }

    return 0;
}
```

**輸出**：
```
執行: 高優先級 (優先級: 10)
執行: 中優先級 (優先級: 5)
執行: 低優先級 (優先級: 3)
```

## 小結

**`std::priority_queue` 是基於堆實現的優先佇列，自動維護最大或最小元素在堆頂，適合需要動態取最值的場景，如任務排程、Dijkstra 最短路徑、合併 K 個有序鏈表等。**
