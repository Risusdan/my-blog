+++
date = '2025-12-22T15:00:00+08:00'
draft = false
title = '[C++] STL Stack（棧容器適配器）'
categories = ['C++ Notes']
tags = ['C++', 'STL', 'Container']
+++

## 什麼是 Stack？

`std::stack` 是**容器適配器**，提供**後進先出**（LIFO）的資料結構。

**特點**：
- 只能訪問棧頂元素
- 底層預設使用 `deque`
- 可以指定底層容器（deque、vector、list）

## 基本使用

```cpp
#include <stack>
#include <iostream>

int main() {
    std::stack<int> myStack;

    // 入棧
    myStack.push(1);
    myStack.push(2);
    myStack.push(3);

    std::cout << "棧頂: " << myStack.top() << std::endl;  // 3
    std::cout << "大小: " << myStack.size() << std::endl;  // 3

    // 出棧
    myStack.pop();
    std::cout << "出棧後棧頂: " << myStack.top() << std::endl;  // 2

    // 遍歷（需要不斷 pop）
    while (!myStack.empty()) {
        std::cout << myStack.top() << " ";
        myStack.pop();
    }

    return 0;
}
```

## 常用成員函式

| 函式 | 功能 | 時間複雜度 |
|------|------|-----------|
| `push(val)` | 入棧 | O(1) |
| `pop()` | 出棧 | O(1) |
| `top()` | 訪問棧頂 | O(1) |
| `empty()` | 判斷是否為空 | O(1) |
| `size()` | 返回元素個數 | O(1) |

## 實際應用：括號匹配

```cpp
#include <stack>
#include <string>
#include <iostream>

bool isValid(const std::string& s) {
    std::stack<char> stk;

    for (char c : s) {
        if (c == '(' || c == '[' || c == '{') {
            stk.push(c);
        } else {
            if (stk.empty()) return false;

            char top = stk.top();
            if ((c == ')' && top == '(') ||
                (c == ']' && top == '[') ||
                (c == '}' && top == '{')) {
                stk.pop();
            } else {
                return false;
            }
        }
    }

    return stk.empty();
}

int main() {
    std::cout << isValid("()[]{}") << std::endl;  // 1 (true)
    std::cout << isValid("([)]") << std::endl;    // 0 (false)
    std::cout << isValid("{[]}") << std::endl;    // 1 (true)

    return 0;
}
```

## 指定底層容器

```cpp
#include <stack>
#include <vector>
#include <list>

// 使用 vector 作為底層容器
std::stack<int, std::vector<int>> stack1;

// 使用 list 作為底層容器
std::stack<int, std::list<int>> stack2;

// 預設使用 deque
std::stack<int> stack3;
```

## 小結

**`std::stack` 是 LIFO 容器適配器，適合需要後進先出操作的場景，如函式呼叫棧、表達式求值、括號匹配等。**
