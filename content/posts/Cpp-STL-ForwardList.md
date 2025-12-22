+++
date = '2025-12-22T11:30:00+08:00'
draft = false
title = '[C++] STL Forward_list（單向鏈表）'
categories = ['C++ Notes']
tags = ['C++', 'STL', 'Container']
+++

## 什麼是 Forward_list？

`std::forward_list` 是 C++11 引入的**單向鏈表容器**，每個節點只儲存指向下一個節點的指標，不能反向遍歷。

**與其他容器的主要區別**：
- 與 `list`：forward_list 是單向鏈表，只能單向遍歷；list 是雙向鏈表，可以雙向遍歷
- 與 `vector`：forward_list 不支援隨機訪問，但在任意位置插入/刪除都是 O(1)
- 與 `deque`：forward_list 沒有連續記憶體，不支援隨機訪問

**適用場景**：
- 只需要單向遍歷的場景
- 極度關注記憶體開銷（比 list 節省一個指標的空間）
- 需要頻繁在已知位置插入/刪除元素
- 與 C 語言的單向鏈表相容

## 內部實現原理

### 底層數據結構

`std::forward_list` 使用**單向鏈表**（singly linked list）實現，每個節點包含：
- 資料欄位（存儲元素值）
- 下一個節點指標（指向下一個節點）

### 記憶體布局示意圖

```
forward_list 物件
┌─────────────┐
│ head ptr  ──┼───→ ┌──────┐    ┌──────┐    ┌──────┐    ┌──────┐
│             │     │ data │───→│ data │───→│ data │───→│ data │───→ nullptr
│             │     │ next │    │ next │    │ next │    │ next │
└─────────────┘     └──────┘    └──────┘    └──────┘    └──────┘
                       [1]         [2]         [3]         [4]

對比 list（雙向鏈表）：
每個節點：
forward_list: sizeof(T) + 1 * sizeof(void*)  // 1 個指標
list:         sizeof(T) + 2 * sizeof(void*)  // 2 個指標（前驅 + 後繼）

記憶體節省：約 50%（在指標開銷部分）
```

**特點**：
1. **單向連結**：只能從頭到尾遍歷，不能反向
2. **記憶體更小**：比 list 節省一個指標（8 bytes on 64-bit）
3. **無 size()**：為了極致效能，不維護元素個數
4. **無尾部操作**：沒有 `push_back()`、`back()` 等函式

### 關鍵操作的實現機制

**插入操作**：
```
在節點 A 之後插入 X：
1. 建立新節點 X
2. X->next = A->next
3. A->next = X
時間複雜度：O(1)
```

**刪除操作**：
```
刪除節點 A 之後的節點：
1. B = A->next       // B 是要刪除的節點
2. A->next = B->next
3. delete B
時間複雜度：O(1)
```

**為什麼沒有 insert() 和 erase()?**
單向鏈表要刪除節點 X，必須知道 X 的前驅節點。因此 forward_list 提供的是：
- `insert_after()` 而非 `insert()`
- `erase_after()` 而非 `erase()`

## 基本使用

### 頭文件和宣告

```cpp
#include <forward_list>
#include <iostream>

std::forward_list<int> myList;              // 空 forward_list
std::forward_list<int> myList2(5);          // 5 個預設值元素
std::forward_list<int> myList3(5, 100);     // 5 個值為 100 的元素
std::forward_list<int> myList4 = {1, 2, 3}; // 初始化列表
```

### 構造方法

```cpp
#include <forward_list>
#include <iostream>

int main() {
    // 預設構造
    std::forward_list<int> list1;

    // 指定大小和初值
    std::forward_list<int> list2(5, 10);  // {10, 10, 10, 10, 10}

    // 從陣列構造
    int arr[] = {1, 2, 3, 4, 5};
    std::forward_list<int> list3(arr, arr + 5);

    // 拷貝構造
    std::forward_list<int> list4(list3);

    // 移動構造
    std::forward_list<int> list5(std::move(list4));

    // 初始化列表（C++11）
    std::forward_list<int> list6 = {10, 20, 30, 40};

    return 0;
}
```

### 常用成員函式

| 函式 | 功能 | 時間複雜度 |
|------|------|-----------|
| `push_front(val)` | 頭部插入 | O(1) |
| `pop_front()` | 刪除頭部元素 | O(1) |
| `insert_after(pos, val)` | 在 pos 之後插入 | O(1) |
| `erase_after(pos)` | 刪除 pos 之後的元素 | O(1) |
| `emplace_front(args...)` | 頭部原地構造 | O(1) |
| `emplace_after(pos, args...)` | 在 pos 之後原地構造 | O(1) |
| `remove(val)` | 刪除所有值為 val 的元素 | O(n) |
| `remove_if(pred)` | 刪除滿足條件的元素 | O(n) |
| `empty()` | 判斷是否為空 | O(1) |
| `clear()` | 清空所有元素 | O(n) |
| `front()` | 訪問第一個元素 | O(1) |
| `sort()` | 排序 | O(n log n) |
| `reverse()` | 反轉 | O(n) |
| `unique()` | 刪除連續重複元素 | O(n) |
| `splice_after()` | 從另一個 forward_list 移動元素 | O(1) 或 O(n) |

**注意**：
- ❌ **沒有** `size()` 函式（為了效能）
- ❌ **沒有** `push_back()`、`back()` 等尾部操作
- ❌ **沒有** `rbegin()`、`rend()` 反向迭代器

## 性能分析

### 時間複雜度表格

| 操作 | forward_list | list | vector |
|------|--------------|------|--------|
| 頭部插入 | **O(1)** | O(1) | O(n) |
| 尾部插入 | **O(n)** | O(1) | O(1) 攤銷 |
| 中間插入（已知位置） | **O(1)** | O(1) | O(n) |
| 頭部刪除 | **O(1)** | O(1) | O(n) |
| 尾部刪除 | **O(n)** | O(1) | O(1) |
| 中間刪除（已知位置） | **O(1)** | O(1) | O(n) |
| 查找 | O(n) | O(n) | O(n) |
| 隨機訪問 | ❌ | ❌ | O(1) |
| 計算大小 | **O(n)** | O(1) | O(1) |

### 空間複雜度分析

**記憶體開銷**：
```cpp
// 64 位元系統
sizeof(Node<int>):
  forward_list: sizeof(int) + sizeof(void*) = 4 + 8 = 12 bytes (實際可能對齊到 16)
  list:         sizeof(int) + 2*sizeof(void*) = 4 + 16 = 20 bytes (實際可能對齊到 24)

// forward_list 比 list 節省約 33%-40% 的記憶體（在節點開銷部分）
```

**與其他容器比較**：
```cpp
// 假設儲存 int 在 64 位元系統
vector<int>:        100 個元素 ≈ 400 bytes（連續記憶體）
forward_list<int>:  100 個元素 ≈ 1200 bytes（每個節點 12 bytes）
list<int>:          100 個元素 ≈ 2000 bytes（每個節點 20 bytes）

// forward_list 記憶體開銷是 list 的 60%
```

### 何時性能最優/最差

**性能最優**：
- 只需要單向遍歷
- 頻繁在頭部插入/刪除
- 極度關注記憶體開銷
- 不需要知道容器大小
- 與 C 語言單向鏈表相容

**性能最差**：
- 需要反向遍歷（根本不支援）
- 需要頻繁在尾部操作（需要 O(n) 遍歷到尾部）
- 需要隨機訪問
- 需要經常查詢容器大小

## 常見操作示例

### 插入元素

```cpp
#include <forward_list>
#include <iostream>

int main() {
    std::forward_list<int> myList = {1, 2, 3, 4, 5};

    // 頭部插入
    myList.push_front(0);
    // myList: {0, 1, 2, 3, 4, 5}

    // 在特定位置之後插入
    auto it = myList.begin();
    std::advance(it, 2);  // 移動到第 3 個位置（索引 2）
    myList.insert_after(it, 99);
    // 在索引 2 之後插入 99
    // myList: {0, 1, 2, 99, 3, 4, 5}

    // 在開頭之後插入多個元素
    myList.insert_after(myList.begin(), 3, 88);
    // myList: {0, 88, 88, 88, 1, 2, 99, 3, 4, 5}

    // 輸出
    std::cout << "myList: ";
    for (int val : myList) {
        std::cout << val << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

**輸出**：
```
myList: 0 88 88 88 1 2 99 3 4 5
```

### 刪除元素

```cpp
#include <forward_list>
#include <iostream>

int main() {
    std::forward_list<int> myList = {1, 2, 3, 2, 4, 2, 5};

    // 刪除頭部
    myList.pop_front();
    // myList: {2, 3, 2, 4, 2, 5}

    // 刪除特定位置之後的元素
    auto it = myList.begin();
    std::advance(it, 1);  // 移動到第 2 個位置
    myList.erase_after(it);  // 刪除第 3 個元素
    // myList: {2, 3, 4, 2, 5}

    // 刪除所有值為 2 的元素
    myList.remove(2);
    // myList: {3, 4, 5}

    // 輸出
    std::cout << "myList: ";
    for (int val : myList) {
        std::cout << val << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

**輸出**：
```
myList: 3 4 5
```

### 查找元素

```cpp
#include <forward_list>
#include <iostream>
#include <algorithm>

int main() {
    std::forward_list<int> myList = {10, 20, 30, 40, 50};

    // 使用 std::find 查找
    auto it = std::find(myList.begin(), myList.end(), 30);

    if (it != myList.end()) {
        std::cout << "找到元素: " << *it << std::endl;

        // 計算距離開頭的位置
        int index = std::distance(myList.begin(), it);
        std::cout << "位於索引: " << index << std::endl;
    }

    // 計算元素個數（需要手動遍歷）
    int count = std::distance(myList.begin(), myList.end());
    std::cout << "元素個數: " << count << std::endl;

    return 0;
}
```

**輸出**：
```
找到元素: 30
位於索引: 2
元素個數: 5
```

### 遍歷容器

```cpp
#include <forward_list>
#include <iostream>

int main() {
    std::forward_list<int> myList = {1, 2, 3, 4, 5};

    // 方法1：範圍 for 迴圈
    std::cout << "方法1: ";
    for (int val : myList) {
        std::cout << val << " ";
    }
    std::cout << std::endl;

    // 方法2：迭代器
    std::cout << "方法2: ";
    for (auto it = myList.begin(); it != myList.end(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    // 方法3：const 迭代器
    std::cout << "方法3: ";
    for (auto it = myList.cbegin(); it != myList.cend(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    // ❌ 不支援反向遍歷
    // for (auto it = myList.rbegin(); it != myList.rend(); ++it) {
    //     // 編譯錯誤！forward_list 沒有 rbegin()
    // }

    return 0;
}
```

**輸出**：
```
方法1: 1 2 3 4 5
方法2: 1 2 3 4 5
方法3: 1 2 3 4 5
```

### 排序與修改

```cpp
#include <forward_list>
#include <iostream>

int main() {
    std::forward_list<int> myList = {5, 2, 8, 1, 9, 3};

    // 排序
    myList.sort();
    std::cout << "排序後: ";
    for (int val : myList) {
        std::cout << val << " ";
    }
    std::cout << std::endl;

    // 反轉
    myList.reverse();
    std::cout << "反轉後: ";
    for (int val : myList) {
        std::cout << val << " ";
    }
    std::cout << std::endl;

    // 去重（需要先排序）
    std::forward_list<int> myList2 = {1, 1, 2, 2, 3, 3, 4};
    myList2.unique();
    std::cout << "去重後: ";
    for (int val : myList2) {
        std::cout << val << " ";
    }
    std::cout << std::endl;

    // 自訂排序（降序）
    myList.sort(std::greater<int>());
    std::cout << "降序排序: ";
    for (int val : myList) {
        std::cout << val << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

**輸出**：
```
排序後: 1 2 3 5 8 9
反轉後: 9 8 5 3 2 1
去重後: 1 2 3 4
降序排序: 9 8 5 3 2 1
```

### Splice_after 操作

```cpp
#include <forward_list>
#include <iostream>

void printList(const std::forward_list<int>& list, const std::string& name) {
    std::cout << name << ": ";
    for (int val : list) {
        std::cout << val << " ";
    }
    std::cout << std::endl;
}

int main() {
    std::forward_list<int> list1 = {1, 2, 3};
    std::forward_list<int> list2 = {4, 5, 6};

    std::cout << "初始狀態:" << std::endl;
    printList(list1, "list1");
    printList(list2, "list2");

    // splice_after：將整個 list2 移動到 list1 的第一個元素之後
    list1.splice_after(list1.begin(), list2);

    std::cout << "\n執行 splice_after 後:" << std::endl;
    printList(list1, "list1");
    printList(list2, "list2");  // list2 變成空的

    return 0;
}
```

**輸出**：
```
初始狀態:
list1: 1 2 3
list2: 4 5 6

執行 splice_after 後:
list1: 1 4 5 6 2 3
list2:
```

### before_begin() 的使用

`before_begin()` 返回一個指向第一個元素之前的迭代器，用於在頭部插入/刪除。

```cpp
#include <forward_list>
#include <iostream>

int main() {
    std::forward_list<int> myList = {1, 2, 3, 4, 5};

    // 在頭部之前（實際上是頭部位置）插入
    myList.insert_after(myList.before_begin(), 0);
    // myList: {0, 1, 2, 3, 4, 5}

    std::cout << "插入後: ";
    for (int val : myList) {
        std::cout << val << " ";
    }
    std::cout << std::endl;

    // 刪除第一個元素
    myList.erase_after(myList.before_begin());
    // myList: {1, 2, 3, 4, 5}

    std::cout << "刪除後: ";
    for (int val : myList) {
        std::cout << val << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

**輸出**：
```
插入後: 0 1 2 3 4 5
刪除後: 1 2 3 4 5
```

## 迭代器支持

### 支援的迭代器類型

`std::forward_list` 提供**前向迭代器**（forward iterator）：

```cpp
#include <forward_list>
#include <iostream>

int main() {
    std::forward_list<int> myList = {1, 2, 3, 4, 5};

    auto it = myList.begin();

    // 前向迭代器支援的操作
    ++it;      // 前進（支援）
    *it;       // 解參考（支援）

    // 不支援的操作
    // --it;      // 錯誤！不支援後退
    // it + 3;    // 錯誤！不支援隨機訪問
    // it[2];     // 錯誤！不支援下標操作

    // 必須使用 std::advance 來移動多步
    std::advance(it, 3);
    std::cout << "移動後的元素: " << *it << std::endl;

    return 0;
}
```

### 迭代器失效問題

**forward_list 的特點**：插入和刪除操作**只會使被刪除元素的迭代器失效**，其他迭代器保持有效。

```cpp
#include <forward_list>
#include <iostream>

int main() {
    std::forward_list<int> myList = {1, 2, 3, 4, 5};

    auto it1 = myList.begin();  // 指向 1
    auto it2 = myList.begin();
    std::advance(it2, 2);       // 指向 3
    auto it3 = myList.begin();
    std::advance(it3, 4);       // 指向 5

    std::cout << "刪除前: it1=" << *it1
              << ", it2=" << *it2
              << ", it3=" << *it3 << std::endl;

    // 刪除第二個元素之後的元素（即第三個元素）
    auto it_before = myList.begin();
    std::advance(it_before, 1);  // 指向 2
    myList.erase_after(it_before);  // 刪除 3

    // it1 仍然有效，it3 仍然有效
    std::cout << "刪除後: it1=" << *it1
              << ", it3=" << *it3 << std::endl;
    // it2 已失效，不能使用

    return 0;
}
```

**輸出**：
```
刪除前: it1=1, it2=3, it3=5
刪除後: it1=1, it3=5
```

**迭代器失效規則總結**：

| 操作 | 失效範圍 |
|------|----------|
| `push_front` | **無失效** |
| `insert_after` | **無失效** |
| `erase_after` | **僅被刪除元素的迭代器失效** |
| `pop_front` | **僅頭部元素的迭代器失效** |
| `remove/remove_if` | **僅被刪除元素的迭代器失效** |
| `clear` | **所有迭代器失效** |

## 常見陷阱與注意事項

### 1. 沒有 size() 函式

```cpp
std::forward_list<int> myList = {1, 2, 3, 4, 5};

// ❌ 錯誤：forward_list 沒有 size()
// int n = myList.size();  // 編譯錯誤！

// ✅ 正確：使用 std::distance（但注意 O(n) 複雜度）
int n = std::distance(myList.begin(), myList.end());
std::cout << "元素個數: " << n << std::endl;

// ✅ 更好：如果需要經常查詢大小，使用 list 而非 forward_list
```

### 2. 沒有尾部操作

```cpp
std::forward_list<int> myList = {1, 2, 3, 4, 5};

// ❌ 錯誤：沒有尾部操作
// myList.push_back(6);   // 編譯錯誤！
// int last = myList.back();  // 編譯錯誤！
// myList.pop_back();     // 編譯錯誤！

// ✅ 如果需要尾部操作，使用 list
std::list<int> myList2 = {1, 2, 3, 4, 5};
myList2.push_back(6);
```

### 3. insert_after 而非 insert

```cpp
std::forward_list<int> myList = {1, 2, 3, 4, 5};

auto it = myList.begin();
std::advance(it, 2);  // 指向第 3 個元素

// ❌ 錯誤：沒有 insert()
// myList.insert(it, 99);  // 編譯錯誤！

// ✅ 正確：使用 insert_after()
myList.insert_after(it, 99);  // 在第 3 個元素之後插入
// myList: {1, 2, 3, 99, 4, 5}
```

### 4. erase_after 而非 erase

```cpp
std::forward_list<int> myList = {1, 2, 3, 4, 5};

auto it = myList.begin();
std::advance(it, 2);  // 指向第 3 個元素

// ❌ 錯誤：沒有 erase()
// myList.erase(it);  // 編譯錯誤！

// ✅ 正確：使用 erase_after()
myList.erase_after(it);  // 刪除第 3 個元素之後的元素（即第 4 個）
// myList: {1, 2, 3, 5}
```

### 5. 不能反向遍歷

```cpp
std::forward_list<int> myList = {1, 2, 3, 4, 5};

// ❌ 錯誤：沒有反向迭代器
// for (auto it = myList.rbegin(); it != myList.rend(); ++it) {
//     std::cout << *it << " ";
// }

// ✅ 如果需要反向遍歷，使用 list 或 先 reverse()
myList.reverse();
for (int val : myList) {
    std::cout << val << " ";
}
// 輸出: 5 4 3 2 1
```

### 6. 使用 before_begin() 操作頭部

```cpp
std::forward_list<int> myList = {1, 2, 3, 4, 5};

// 刪除第一個元素有兩種方法：

// ❌ 容易出錯：使用 erase_after(before_begin())
myList.erase_after(myList.before_begin());

// ✅ 更直接：使用 pop_front()
myList.pop_front();
```

## 實際應用場景

### 場景 1：稀疏圖的鄰接表

圖的鄰接表表示中，每個頂點的鄰居只需要單向遍歷。

```cpp
#include <forward_list>
#include <iostream>
#include <vector>

class Graph {
private:
    int numVertices;
    std::vector<std::forward_list<int>> adjList;

public:
    Graph(int vertices) : numVertices(vertices), adjList(vertices) {}

    void addEdge(int src, int dest) {
        // 單向邊：src -> dest
        adjList[src].push_front(dest);
    }

    void addUndirectedEdge(int v1, int v2) {
        // 無向邊：雙向連接
        adjList[v1].push_front(v2);
        adjList[v2].push_front(v1);
    }

    void printGraph() const {
        for (int v = 0; v < numVertices; ++v) {
            std::cout << "頂點 " << v << " 的鄰居: ";
            for (int neighbor : adjList[v]) {
                std::cout << neighbor << " ";
            }
            std::cout << std::endl;
        }
    }

    // DFS 遍歷
    void DFS(int start) {
        std::vector<bool> visited(numVertices, false);
        DFSUtil(start, visited);
        std::cout << std::endl;
    }

private:
    void DFSUtil(int v, std::vector<bool>& visited) {
        visited[v] = true;
        std::cout << v << " ";

        for (int neighbor : adjList[v]) {
            if (!visited[neighbor]) {
                DFSUtil(neighbor, visited);
            }
        }
    }
};

int main() {
    Graph graph(5);

    // 添加邊
    graph.addUndirectedEdge(0, 1);
    graph.addUndirectedEdge(0, 4);
    graph.addUndirectedEdge(1, 2);
    graph.addUndirectedEdge(1, 3);
    graph.addUndirectedEdge(1, 4);
    graph.addUndirectedEdge(2, 3);
    graph.addUndirectedEdge(3, 4);

    std::cout << "圖的鄰接表表示:" << std::endl;
    graph.printGraph();

    std::cout << "\n從頂點 0 開始的 DFS 遍歷: ";
    graph.DFS(0);

    return 0;
}
```

**輸出**：
```
圖的鄰接表表示:
頂點 0 的鄰居: 4 1
頂點 1 的鄰居: 4 3 2 0
頂點 2 的鄰居: 3 1
頂點 3 的鄰居: 4 2 1
頂點 4 的鄰居: 3 1 0

從頂點 0 開始的 DFS 遍歷: 0 4 3 2 1
```

### 場景 2：簡單的記憶體池（Free List）

記憶體分配器中的空閒鏈表。

```cpp
#include <forward_list>
#include <iostream>

class SimpleMemoryPool {
private:
    struct Block {
        char data[64];  // 64 bytes 區塊
    };

    std::forward_list<Block*> freeList;  // 空閒區塊鏈表
    static constexpr size_t POOL_SIZE = 10;

public:
    SimpleMemoryPool() {
        // 預先分配區塊
        for (size_t i = 0; i < POOL_SIZE; ++i) {
            freeList.push_front(new Block());
        }
    }

    ~SimpleMemoryPool() {
        // 釋放所有區塊
        for (Block* block : freeList) {
            delete block;
        }
    }

    Block* allocate() {
        if (freeList.empty()) {
            std::cerr << "記憶體池已滿" << std::endl;
            return nullptr;
        }

        Block* block = freeList.front();
        freeList.pop_front();
        return block;
    }

    void deallocate(Block* block) {
        if (block) {
            freeList.push_front(block);
        }
    }

    int getFreeCount() const {
        return std::distance(freeList.begin(), freeList.end());
    }
};

int main() {
    SimpleMemoryPool pool;

    std::cout << "初始空閒區塊: " << pool.getFreeCount() << std::endl;

    // 分配 3 個區塊
    auto* block1 = pool.allocate();
    auto* block2 = pool.allocate();
    auto* block3 = pool.allocate();

    std::cout << "分配 3 個後空閒區塊: " << pool.getFreeCount() << std::endl;

    // 歸還 2 個區塊
    pool.deallocate(block1);
    pool.deallocate(block2);

    std::cout << "歸還 2 個後空閒區塊: " << pool.getFreeCount() << std::endl;

    // 清理
    pool.deallocate(block3);

    return 0;
}
```

**輸出**：
```
初始空閒區塊: 10
分配 3 個後空閒區塊: 7
歸還 2 個後空閒區塊: 9
```

### 場景 3：哈希表的開放定址法（鏈式法）

哈希表衝突解決中的鏈表。

```cpp
#include <forward_list>
#include <iostream>
#include <string>
#include <functional>

template<typename K, typename V>
class SimpleHashMap {
private:
    struct Entry {
        K key;
        V value;

        Entry(const K& k, const V& v) : key(k), value(v) {}
    };

    static constexpr size_t TABLE_SIZE = 10;
    std::vector<std::forward_list<Entry>> table;

    size_t hash(const K& key) const {
        return std::hash<K>{}(key) % TABLE_SIZE;
    }

public:
    SimpleHashMap() : table(TABLE_SIZE) {}

    void insert(const K& key, const V& value) {
        size_t index = hash(key);

        // 檢查是否已存在
        for (auto& entry : table[index]) {
            if (entry.key == key) {
                entry.value = value;  // 更新
                return;
            }
        }

        // 新增
        table[index].emplace_front(key, value);
    }

    bool find(const K& key, V& value) const {
        size_t index = hash(key);

        for (const auto& entry : table[index]) {
            if (entry.key == key) {
                value = entry.value;
                return true;
            }
        }

        return false;
    }

    void remove(const K& key) {
        size_t index = hash(key);
        table[index].remove_if([&key](const Entry& entry) {
            return entry.key == key;
        });
    }

    void printTable() const {
        for (size_t i = 0; i < TABLE_SIZE; ++i) {
            if (!table[i].empty()) {
                std::cout << "Bucket " << i << ": ";
                for (const auto& entry : table[i]) {
                    std::cout << "{" << entry.key << ":" << entry.value << "} ";
                }
                std::cout << std::endl;
            }
        }
    }
};

int main() {
    SimpleHashMap<std::string, int> map;

    map.insert("apple", 100);
    map.insert("banana", 200);
    map.insert("cherry", 300);
    map.insert("date", 400);

    std::cout << "哈希表內容:" << std::endl;
    map.printTable();

    int value;
    if (map.find("banana", value)) {
        std::cout << "\nbanana 的值: " << value << std::endl;
    }

    map.remove("banana");
    std::cout << "\n刪除 banana 後:" << std::endl;
    map.printTable();

    return 0;
}
```

**輸出**：
```
哈希表內容:
Bucket 0: {date:400}
Bucket 1: {cherry:300}
Bucket 5: {banana:200}
Bucket 7: {apple:100}

banana 的值: 200

刪除 banana 後:
Bucket 0: {date:400}
Bucket 1: {cherry:300}
Bucket 7: {apple:100}
```

## 與其他容器的選擇

### 對比表格

| 特性 | forward_list | list | vector |
|------|--------------|------|--------|
| 單向遍歷 | ✅ | ✅ | ✅ |
| 雙向遍歷 | ❌ | ✅ | ✅ |
| 隨機訪問 | ❌ | ❌ | ✅ |
| 頭部插入 | ✅ O(1) | ✅ O(1) | ❌ O(n) |
| 尾部插入 | ❌ O(n) | ✅ O(1) | ✅ O(1) |
| 中間插入 | ✅ O(1)* | ✅ O(1)* | ❌ O(n) |
| 迭代器穩定 | ✅ | ✅ | ❌ |
| 記憶體開銷 | 最小 | 中 | 小 |
| 緩存友好 | ❌ | ❌ | ✅ |
| size() | ❌ O(n) | ✅ O(1) | ✅ O(1) |

*：已知位置的情況

### 選擇建議

**選擇 forward_list 當**：
- ✅ 只需要單向遍歷
- ✅ 極度關注記憶體開銷
- ✅ 頻繁在頭部插入/刪除
- ✅ 不需要知道容器大小
- ✅ 與 C 語言單向鏈表相容

**選擇 list 當**：
- ✅ 需要雙向遍歷
- ✅ 需要在尾部插入/刪除
- ✅ 需要經常查詢容器大小
- ✅ 記憶體不是極度受限

**選擇 vector 當**：
- ✅ 需要隨機訪問
- ✅ 大部分操作在尾部
- ✅ 需要高緩存性能
- ✅ 元素數量變化不大

## 實務建議

### DO（應該做的）

#### 1. ✅ 極致記憶體優化時優先選擇

```cpp
// ✅ 推薦：記憶體受限環境
class EmbeddedSystem {
    std::forward_list<Event> eventQueue;  // 最小記憶體開銷
};

// ❌ 避免：不必要的記憶體浪費
// std::list<Event> eventQueue;  // 比 forward_list 多 50% 記憶體
```

#### 2. ✅ 使用 emplace_front 減少複製

```cpp
struct Data {
    int x, y;
    Data(int x, int y) : x(x), y(y) {}
};

std::forward_list<Data> list;

// ✅ 推薦：原地構造
list.emplace_front(10, 20);

// ❌ 避免：構造後移動
// list.push_front(Data(10, 20));
```

#### 3. ✅ 使用 before_begin() 操作頭部

```cpp
std::forward_list<int> list = {1, 2, 3};

// ✅ 推薦：在頭部插入
list.insert_after(list.before_begin(), 0);
// list: {0, 1, 2, 3}

// ✅ 也可以：使用 push_front
list.push_front(-1);
// list: {-1, 0, 1, 2, 3}
```

#### 4. ✅ 利用 remove_if 批次刪除

```cpp
std::forward_list<int> list = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

// ✅ 推薦：使用 remove_if
list.remove_if([](int x) { return x % 2 == 0; });  // 刪除偶數

// ❌ 避免：手動遍歷刪除（更複雜）
```

#### 5. ✅ 使用 splice_after 高效移動元素

```cpp
std::forward_list<int> list1 = {1, 2, 3};
std::forward_list<int> list2 = {4, 5, 6};

// ✅ 推薦：splice_after（O(1)，無複製）
list1.splice_after(list1.begin(), list2);

// ❌ 避免：複製後清空（O(n)，有複製開銷）
```

### DON'T（不應該做的）

#### 1. ❌ 不要在需要雙向遍歷時使用

```cpp
// ❌ 避免：需要反向遍歷卻用 forward_list
std::forward_list<int> list = {1, 2, 3, 4, 5};
// 無法反向遍歷！

// ✅ 正確：使用 list
std::list<int> list2 = {1, 2, 3, 4, 5};
for (auto it = list2.rbegin(); it != list2.rend(); ++it) {
    // 可以反向遍歷
}
```

#### 2. ❌ 不要頻繁查詢容器大小

```cpp
std::forward_list<int> list = {1, 2, 3, 4, 5};

// ❌ 避免：每次都 O(n) 計算大小
for (int i = 0; i < std::distance(list.begin(), list.end()); ++i) {
    // 每次迴圈都重新計算！
}

// ✅ 正確：快取大小或使用 list
int size = std::distance(list.begin(), list.end());
for (int i = 0; i < size; ++i) {
    // ...
}
```

#### 3. ❌ 不要嘗試尾部操作

```cpp
std::forward_list<int> list = {1, 2, 3, 4, 5};

// ❌ 錯誤：沒有 push_back()
// list.push_back(6);  // 編譯錯誤

// ❌ 低效：手動遍歷到尾部（O(n)）
auto it = list.begin();
while (std::next(it) != list.end()) {
    ++it;
}
list.insert_after(it, 6);

// ✅ 正確：如需尾部操作，使用 list
std::list<int> list2 = {1, 2, 3, 4, 5};
list2.push_back(6);  // O(1)
```

#### 4. ❌ 不要忘記 insert_after/erase_after 的語意

```cpp
std::forward_list<int> list = {1, 2, 3, 4, 5};
auto it = list.begin();
std::advance(it, 2);  // 指向 3

// ❌ 常見錯誤：以為 erase_after 刪除當前元素
// list.erase_after(it);  // 實際刪除的是 4，不是 3！

// ✅ 正確：要刪除當前元素，需要前一個迭代器
auto prev = list.before_begin();
for (auto curr = list.begin(); curr != it; ++curr) {
    prev = curr;
}
list.erase_after(prev);  // 刪除 3
```

#### 5. ❌ 不要對小型資料使用鏈表

```cpp
// ❌ 避免：只有幾個元素時鏈表開銷大
std::forward_list<int> smallList = {1, 2, 3};
// 12 bytes * 3 = 36 bytes（節點）+ 指標開銷

// ✅ 推薦：使用 vector 或 array
std::vector<int> smallVec = {1, 2, 3};
// 4 bytes * 3 = 12 bytes（資料）+ vector 開銷
```

## 小結

### 核心概念總結

1. **單向鏈表**：每個節點只有一個指向下一個節點的指標，不能反向遍歷
2. **極致效能**：比 list 節省 50% 的指標記憶體，但犧牲了雙向遍歷能力
3. **無 size()**：為了極致效能，不維護元素個數（查詢需要 O(n)）
4. **insert_after/erase_after**：操作的是指定位置**之後**的元素，而非當前元素
5. **僅前向遍歷**：不支援反向迭代器和反向遍歷

### 關鍵記憶點

- ✅ **適合場景**：單向遍歷、極致記憶體優化、與 C 單向鏈表相容
- ❌ **不適合場景**：需要雙向遍歷、尾部操作、經常查詢大小
- 🔧 **核心優勢**：最小記憶體開銷、與 C 相容
- ⚠️ **主要問題**：無反向遍歷、無 size()、無尾部操作

### 一句話總結

**`std::forward_list` 是單向鏈表容器，以犧牲雙向遍歷能力換取最小的記憶體開銷，適合只需單向遍歷且極度關注記憶體使用的場景，是 STL 中最輕量的序列容器。**
