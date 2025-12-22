+++
date = '2025-12-22T15:15:00+08:00'
draft = false
title = '[C++] STL Queue（佇列容器適配器）'
categories = ['C++ Notes']
tags = ['C++', 'STL', 'Container']
+++

## 什麼是 Queue？

`std::queue` 是**容器適配器**，提供**先進先出**（FIFO）的資料結構。

**特點**：
- 只能訪問隊首和隊尾
- 底層預設使用 `deque`
- 可以指定底層容器（deque、list）

## 基本使用

```cpp
#include <queue>
#include <iostream>

int main() {
    std::queue<int> myQueue;

    // 入隊
    myQueue.push(1);
    myQueue.push(2);
    myQueue.push(3);

    std::cout << "隊首: " << myQueue.front() << std::endl;  // 1
    std::cout << "隊尾: " << myQueue.back() << std::endl;   // 3
    std::cout << "大小: " << myQueue.size() << std::endl;   // 3

    // 出隊
    myQueue.pop();
    std::cout << "出隊後隊首: " << myQueue.front() << std::endl;  // 2

    // 遍歷（需要不斷 pop）
    while (!myQueue.empty()) {
        std::cout << myQueue.front() << " ";
        myQueue.pop();
    }

    return 0;
}
```

## 常用成員函式

| 函式 | 功能 | 時間複雜度 |
|------|------|-----------|
| `push(val)` | 入隊 | O(1) |
| `pop()` | 出隊 | O(1) |
| `front()` | 訪問隊首 | O(1) |
| `back()` | 訪問隊尾 | O(1) |
| `empty()` | 判斷是否為空 | O(1) |
| `size()` | 返回元素個數 | O(1) |

## 實際應用：BFS

```cpp
#include <queue>
#include <vector>
#include <iostream>

void BFS(int start, const std::vector<std::vector<int>>& graph) {
    std::queue<int> q;
    std::vector<bool> visited(graph.size(), false);

    q.push(start);
    visited[start] = true;

    while (!q.empty()) {
        int node = q.front();
        q.pop();
        std::cout << node << " ";

        for (int neighbor : graph[node]) {
            if (!visited[neighbor]) {
                q.push(neighbor);
                visited[neighbor] = true;
            }
        }
    }
}

int main() {
    std::vector<std::vector<int>> graph = {
        {1, 2},    // 0 -> 1, 2
        {0, 3, 4}, // 1 -> 0, 3, 4
        {0, 5},    // 2 -> 0, 5
        {1},       // 3 -> 1
        {1},       // 4 -> 1
        {2}        // 5 -> 2
    };

    std::cout << "BFS 從節點 0 開始: ";
    BFS(0, graph);

    return 0;
}
```

## 小結

**`std::queue` 是 FIFO 容器適配器，適合需要先進先出操作的場景，如 BFS、任務排程、緩衝區等。**
