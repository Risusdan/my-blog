+++
date = '2025-12-22T10:30:00+08:00'
draft = false
title = '[C++] STL List（雙向鏈表）'
categories = ['C++ Notes']
tags = ['C++', 'STL', 'Container']
+++

## 什麼是 List？

`std::list` 是 C++ STL 中的**雙向鏈表容器**，每個元素都儲存在獨立的節點中，節點之間通過指標連接。

**與其他容器的主要區別**：
- 與 `vector`：list 不支援隨機訪問，但在任意位置插入/刪除都是 O(1)
- 與 `deque`：list 沒有連續記憶體的優勢，但插入/刪除不會使其他元素的迭代器失效
- 與 `forward_list`：list 是雙向鏈表，可以雙向遍歷；forward_list 是單向鏈表，記憶體開銷更小

**適用場景**：
- 需要頻繁在中間位置插入/刪除元素
- 不需要隨機訪問元素
- 需要保證插入/刪除操作不影響其他迭代器的有效性
- 需要在容器間快速移動元素（splice 操作）

## 內部實現原理

### 底層數據結構

`std::list` 使用**雙向鏈表**（doubly linked list）實現，每個節點包含：
- 資料欄位（存儲元素值）
- 前驅指標（指向前一個節點）
- 後繼指標（指向後一個節點）

### 記憶體布局示意圖

```
list 物件
┌─────────────┐
│ head ptr  ──┼───→ ┌──────┐      ┌──────┐      ┌──────┐      ┌──────┐
│             │     │ prev │←────→│ prev │←────→│ prev │←────→│ prev │
│ tail ptr  ──┼─┐   │ data │      │ data │      │ data │      │ data │
│             │ │   │ next │      │ next │      │ next │      │ next │
│ size        │ │   └──────┘      └──────┘      └──────┘      └──────┘
└─────────────┘ │      [1]           [2]           [3]           [4]
                │
                └───────────────────────────────────────────────────→
```

**特點**：
1. **非連續記憶體**：每個節點獨立分配，散布在堆記憶體中
2. **雙向連結**：每個節點都知道前後節點的位置
3. **環狀結構**：通常實作為環狀鏈表，有一個哨兵節點（sentinel node）簡化邊界處理
4. **固定節點大小**：每個節點的大小 = `sizeof(T) + 2 * sizeof(pointer)`

### 關鍵操作的實現機制

**插入操作**：
```
插入 X 到節點 A 和 B 之間：
1. 建立新節點 X
2. X->prev = A
3. X->next = B
4. A->next = X
5. B->prev = X
時間複雜度：O(1)（已知位置）
```

**刪除操作**：
```
刪除節點 X：
1. A = X->prev
2. B = X->next
3. A->next = B
4. B->prev = A
5. delete X
時間複雜度：O(1)（已知位置）
```

**查找操作**：
必須從頭或尾開始逐個遍歷，時間複雜度：O(n)

## 基本使用

### 頭文件和宣告

```cpp
#include <list>
#include <iostream>

std::list<int> myList;              // 空 list
std::list<int> myList2(5);          // 5 個預設值元素
std::list<int> myList3(5, 100);     // 5 個值為 100 的元素
std::list<int> myList4 = {1, 2, 3}; // 初始化列表
```

### 構造方法

```cpp
#include <list>
#include <iostream>

int main() {
    // 預設構造
    std::list<int> list1;

    // 指定大小和初值
    std::list<int> list2(5, 10);  // {10, 10, 10, 10, 10}

    // 從陣列構造
    int arr[] = {1, 2, 3, 4, 5};
    std::list<int> list3(arr, arr + 5);

    // 拷貝構造
    std::list<int> list4(list3);

    // 移動構造（C++11）
    std::list<int> list5(std::move(list4));

    // 初始化列表（C++11）
    std::list<int> list6 = {10, 20, 30, 40};

    return 0;
}
```

### 常用成員函式

| 函式 | 功能 | 時間複雜度 |
|------|------|-----------|
| `push_back(val)` | 尾部插入 | O(1) |
| `push_front(val)` | 頭部插入 | O(1) |
| `pop_back()` | 刪除尾部元素 | O(1) |
| `pop_front()` | 刪除頭部元素 | O(1) |
| `insert(pos, val)` | 在 pos 位置插入 | O(1) |
| `erase(pos)` | 刪除 pos 位置元素 | O(1) |
| `remove(val)` | 刪除所有值為 val 的元素 | O(n) |
| `size()` | 返回元素個數 | O(1) |
| `empty()` | 判斷是否為空 | O(1) |
| `clear()` | 清空所有元素 | O(n) |
| `front()` | 訪問第一個元素 | O(1) |
| `back()` | 訪問最後一個元素 | O(1) |
| `sort()` | 排序（歸併排序） | O(n log n) |
| `reverse()` | 反轉 | O(n) |
| `unique()` | 刪除連續重複元素 | O(n) |
| `splice()` | 從另一個 list 移動元素 | O(1) 或 O(n) |

## 性能分析

### 時間複雜度表格

| 操作 | list | vector | deque |
|------|------|--------|-------|
| 隨機訪問 | **O(n)** | O(1) | O(1) |
| 頭部插入 | **O(1)** | O(n) | O(1) |
| 尾部插入 | **O(1)** | O(1) 攤銷 | O(1) |
| 中間插入 | **O(1)** | O(n) | O(n) |
| 頭部刪除 | **O(1)** | O(n) | O(1) |
| 尾部刪除 | **O(1)** | O(1) | O(1) |
| 中間刪除 | **O(1)** | O(n) | O(n) |
| 查找 | O(n) | O(n) | O(n) |

**注意**：list 的插入/刪除時間複雜度 O(1) 是指**已知位置**的情況。如果需要先找到位置，總時間複雜度仍是 O(n)。

### 空間複雜度分析

**記憶體開銷**：
- 每個元素：`sizeof(T) + 2 * sizeof(void*)`
- 在 64 位元系統上，額外開銷約為 16 bytes（兩個指標）
- 總空間：`n * (sizeof(T) + 16) + 常數開銷`

**與 vector 比較**：
```cpp
// 假設 int 在 64 位元系統上
vector<int>: 100 個元素 ≈ 400 bytes（100 * 4）
list<int>:   100 個元素 ≈ 2000 bytes（100 * (4 + 16)）
```

list 的記憶體開銷**大約是 vector 的 5 倍**（對於 int 類型）。

### 何時性能最優/最差

**性能最優**：
- 頻繁在已知位置插入/刪除元素
- 需要穩定的迭代器（插入/刪除不影響其他迭代器）
- 需要在容器間移動元素（splice 操作非常高效）
- 元素類型的移動/拷貝成本很高

**性能最差**：
- 需要隨機訪問元素（必須線性遍歷）
- 需要大量查找操作（沒有緩存友好性）
- 記憶體受限的環境（記憶體開銷大）
- 元素類型很小（如 int、char），額外開銷比例高

## 常見操作示例

### 插入元素

```cpp
#include <list>
#include <iostream>

int main() {
    std::list<int> myList = {1, 2, 3, 4, 5};

    // 頭部插入
    myList.push_front(0);
    // myList: {0, 1, 2, 3, 4, 5}

    // 尾部插入
    myList.push_back(6);
    // myList: {0, 1, 2, 3, 4, 5, 6}

    // 在特定位置插入
    auto it = myList.begin();
    std::advance(it, 3);  // 移動到第 4 個位置
    myList.insert(it, 99);
    // myList: {0, 1, 2, 99, 3, 4, 5, 6}

    // 插入多個元素
    it = myList.begin();
    std::advance(it, 2);
    myList.insert(it, 3, 88);  // 插入 3 個 88
    // myList: {0, 1, 88, 88, 88, 2, 99, 3, 4, 5, 6}

    // 輸出
    for (int val : myList) {
        std::cout << val << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

**輸出**：
```
0 1 88 88 88 2 99 3 4 5 6
```

### 刪除元素

```cpp
#include <list>
#include <iostream>

int main() {
    std::list<int> myList = {1, 2, 3, 2, 4, 2, 5};

    // 刪除頭部
    myList.pop_front();
    // myList: {2, 3, 2, 4, 2, 5}

    // 刪除尾部
    myList.pop_back();
    // myList: {2, 3, 2, 4, 2}

    // 刪除特定位置
    auto it = myList.begin();
    std::advance(it, 1);  // 移動到第 2 個位置
    myList.erase(it);
    // myList: {2, 2, 4, 2}

    // 刪除所有值為 2 的元素
    myList.remove(2);
    // myList: {4}

    std::cout << "剩餘元素: ";
    for (int val : myList) {
        std::cout << val << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

**輸出**：
```
剩餘元素: 4
```

### 查找元素

```cpp
#include <list>
#include <iostream>
#include <algorithm>

int main() {
    std::list<int> myList = {10, 20, 30, 40, 50};

    // 使用 std::find 查找
    auto it = std::find(myList.begin(), myList.end(), 30);

    if (it != myList.end()) {
        std::cout << "找到元素: " << *it << std::endl;

        // 計算距離開頭的位置
        int index = std::distance(myList.begin(), it);
        std::cout << "位於索引: " << index << std::endl;
    } else {
        std::cout << "未找到元素" << std::endl;
    }

    // 使用 std::count 計算出現次數
    std::list<int> myList2 = {1, 2, 3, 2, 4, 2, 5};
    int count = std::count(myList2.begin(), myList2.end(), 2);
    std::cout << "元素 2 出現 " << count << " 次" << std::endl;

    return 0;
}
```

**輸出**：
```
找到元素: 30
位於索引: 2
元素 2 出現 3 次
```

### 遍歷容器

```cpp
#include <list>
#include <iostream>

int main() {
    std::list<int> myList = {1, 2, 3, 4, 5};

    // 方法1：範圍 for 迴圈（C++11）
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

    // 方法3：反向遍歷
    std::cout << "方法3（反向）: ";
    for (auto it = myList.rbegin(); it != myList.rend(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    // 方法4：const 迭代器（只讀）
    std::cout << "方法4（唯讀）: ";
    for (auto it = myList.cbegin(); it != myList.cend(); ++it) {
        std::cout << *it << " ";
        // *it = 10;  // 錯誤！const 迭代器不能修改
    }
    std::cout << std::endl;

    return 0;
}
```

**輸出**：
```
方法1: 1 2 3 4 5
方法2: 1 2 3 4 5
方法3（反向）: 5 4 3 2 1
方法4（唯讀）: 1 2 3 4 5
```

### 排序與修改

```cpp
#include <list>
#include <iostream>

int main() {
    std::list<int> myList = {5, 2, 8, 1, 9, 3};

    // 排序（使用 list 的成員函式 sort）
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
    std::list<int> myList2 = {1, 1, 2, 2, 3, 3, 4};
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

### Splice 操作（列表拼接）

```cpp
#include <list>
#include <iostream>

void printList(const std::list<int>& list, const std::string& name) {
    std::cout << name << ": ";
    for (int val : list) {
        std::cout << val << " ";
    }
    std::cout << std::endl;
}

int main() {
    std::list<int> list1 = {1, 2, 3};
    std::list<int> list2 = {4, 5, 6};

    std::cout << "初始狀態:" << std::endl;
    printList(list1, "list1");
    printList(list2, "list2");

    // splice：將整個 list2 移動到 list1 的末尾
    list1.splice(list1.end(), list2);

    std::cout << "\n執行 splice 後:" << std::endl;
    printList(list1, "list1");
    printList(list2, "list2");  // list2 變成空的

    // 將 list1 的一部分移動到新 list
    std::list<int> list3;
    auto it = list1.begin();
    std::advance(it, 2);  // 移動到第 3 個元素
    list3.splice(list3.begin(), list1, it);  // 移動單個元素

    std::cout << "\n移動單個元素後:" << std::endl;
    printList(list1, "list1");
    printList(list3, "list3");

    return 0;
}
```

**輸出**：
```
初始狀態:
list1: 1 2 3
list2: 4 5 6

執行 splice 後:
list1: 1 2 3 4 5 6
list2:

移動單個元素後:
list1: 1 2 4 5 6
list3: 3
```

## 迭代器支持

### 支援的迭代器類型

`std::list` 提供**雙向迭代器**（bidirectional iterator）：

```cpp
#include <list>
#include <iostream>

int main() {
    std::list<int> myList = {1, 2, 3, 4, 5};

    auto it = myList.begin();

    // 雙向迭代器支援的操作
    ++it;      // 前進（支援）
    --it;      // 後退（支援）
    *it;       // 解參考（支援）

    // 不支援的操作
    // it + 3;   // 錯誤！不支援隨機訪問
    // it[2];    // 錯誤！不支援下標操作

    // 必須使用 std::advance 來移動多步
    std::advance(it, 3);  // 移動 3 步
    std::cout << "移動後的元素: " << *it << std::endl;

    return 0;
}
```

### 迭代器失效問題

**list 的優勢**：插入和刪除操作**只會使被刪除元素的迭代器失效**，其他迭代器保持有效。

```cpp
#include <list>
#include <iostream>

int main() {
    std::list<int> myList = {1, 2, 3, 4, 5};

    auto it1 = myList.begin();  // 指向 1
    auto it2 = myList.begin();
    std::advance(it2, 2);       // 指向 3
    auto it3 = myList.begin();
    std::advance(it3, 4);       // 指向 5

    std::cout << "刪除前: it1=" << *it1
              << ", it2=" << *it2
              << ", it3=" << *it3 << std::endl;

    // 刪除中間元素
    auto it_to_delete = myList.begin();
    std::advance(it_to_delete, 2);  // 指向 3
    myList.erase(it_to_delete);     // 刪除 3

    // it1 和 it3 仍然有效！
    std::cout << "刪除後: it1=" << *it1
              << ", it3=" << *it3 << std::endl;
    // it2 已失效，不能使用

    // 插入也不會使其他迭代器失效
    myList.insert(it3, 99);
    std::cout << "插入後: it1=" << *it1
              << ", it3=" << *it3 << std::endl;

    return 0;
}
```

**輸出**：
```
刪除前: it1=1, it2=3, it3=5
刪除後: it1=1, it3=5
插入後: it1=1, it3=5
```

**迭代器失效規則總結**：

| 操作 | 失效範圍 |
|------|----------|
| `push_front/push_back` | **無失效** |
| `insert` | **無失效** |
| `erase` | **僅被刪除元素的迭代器失效** |
| `pop_front/pop_back` | **僅被刪除元素的迭代器失效** |
| `remove/remove_if` | **僅被刪除元素的迭代器失效** |
| `clear` | **所有迭代器失效** |

## 常見陷阱與注意事項

### 1. 不支援隨機訪問

```cpp
// ❌ 錯誤寫法
std::list<int> myList = {1, 2, 3, 4, 5};
// int val = myList[2];  // 編譯錯誤！list 不支援 operator[]

// ✅ 正確寫法
auto it = myList.begin();
std::advance(it, 2);
int val = *it;
```

### 2. 不要使用 std::sort

```cpp
std::list<int> myList = {5, 2, 8, 1, 9};

// ❌ 錯誤寫法
// std::sort(myList.begin(), myList.end());
// 編譯錯誤！std::sort 需要隨機訪問迭代器

// ✅ 正確寫法
myList.sort();  // 使用 list 的成員函式
```

### 3. 記憶體開銷大

```cpp
// ❌ 不推薦：儲存小型元素時記憶體浪費嚴重
std::list<char> charList;  // 每個 char (1 byte) 需要額外 16 bytes

// ✅ 推薦：儲存大型物件或需要穩定迭代器時使用
struct LargeObject {
    char data[1000];
};
std::list<LargeObject> objList;  // 額外 16 bytes 相對合理
```

### 4. 緩存性能差

```cpp
// list 的元素分散在記憶體中，緩存命中率低
std::list<int> myList(1000000);

// 遍歷 list 比遍歷 vector 慢很多（即使都是 O(n)）
for (int val : myList) {  // 緩存不友好
    // 處理...
}
```

### 5. splice 會修改源容器

```cpp
std::list<int> list1 = {1, 2, 3};
std::list<int> list2 = {4, 5, 6};

// splice 是「移動」不是「複製」
list1.splice(list1.end(), list2);

// ⚠️ list2 現在是空的！
std::cout << "list2 size: " << list2.size() << std::endl;  // 輸出 0
```

### 6. 使用 erase 時要更新迭代器

```cpp
std::list<int> myList = {1, 2, 3, 4, 5};

// ❌ 錯誤寫法
for (auto it = myList.begin(); it != myList.end(); ++it) {
    if (*it == 3) {
        myList.erase(it);  // it 失效，++it 未定義行為
    }
}

// ✅ 正確寫法
for (auto it = myList.begin(); it != myList.end(); ) {
    if (*it == 3) {
        it = myList.erase(it);  // erase 返回下一個有效迭代器
    } else {
        ++it;
    }
}
```

## 實際應用場景

### 場景 1：LRU Cache（最近最少使用緩存）

LRU Cache 需要在 O(1) 時間內將訪問的元素移到最前面，list 的 splice 非常適合。

```cpp
#include <list>
#include <unordered_map>
#include <iostream>

class LRUCache {
private:
    int capacity;
    std::list<std::pair<int, int>> cacheList;  // {key, value}
    std::unordered_map<int, std::list<std::pair<int, int>>::iterator> cacheMap;

public:
    LRUCache(int cap) : capacity(cap) {}

    int get(int key) {
        if (cacheMap.find(key) == cacheMap.end()) {
            return -1;  // 未找到
        }

        // 將訪問的元素移到最前面（最近使用）
        auto it = cacheMap[key];
        int value = it->second;
        cacheList.erase(it);
        cacheList.push_front({key, value});
        cacheMap[key] = cacheList.begin();

        return value;
    }

    void put(int key, int value) {
        if (cacheMap.find(key) != cacheMap.end()) {
            // 已存在，更新並移到最前面
            cacheList.erase(cacheMap[key]);
        } else if (cacheList.size() >= capacity) {
            // 容量已滿，刪除最舊的（最後一個）
            int oldKey = cacheList.back().first;
            cacheList.pop_back();
            cacheMap.erase(oldKey);
        }

        // 插入到最前面
        cacheList.push_front({key, value});
        cacheMap[key] = cacheList.begin();
    }

    void print() {
        std::cout << "Cache: ";
        for (const auto& p : cacheList) {
            std::cout << "{" << p.first << ":" << p.second << "} ";
        }
        std::cout << std::endl;
    }
};

int main() {
    LRUCache cache(3);

    cache.put(1, 100);
    cache.put(2, 200);
    cache.put(3, 300);
    cache.print();  // Cache: {3:300} {2:200} {1:100}

    cache.get(1);   // 訪問 1，移到最前面
    cache.print();  // Cache: {1:100} {3:300} {2:200}

    cache.put(4, 400);  // 容量已滿，刪除最舊的 2
    cache.print();      // Cache: {4:400} {1:100} {3:300}

    std::cout << "Get 2: " << cache.get(2) << std::endl;  // -1（已被淘汰）
    std::cout << "Get 1: " << cache.get(1) << std::endl;  // 100

    return 0;
}
```

**輸出**：
```
Cache: {3:300} {2:200} {1:100}
Cache: {1:100} {3:300} {2:200}
Cache: {4:400} {1:100} {3:300}
Get 2: -1
Get 1: 100
```

### 場景 2：文字編輯器的 Undo/Redo 功能

文字編輯器需要在任意位置插入/刪除，並保持操作歷史。

```cpp
#include <list>
#include <string>
#include <iostream>

class TextEditor {
private:
    std::list<std::string> lines;           // 文字內容（每行一個元素）
    std::list<std::string> undoStack;       // 操作歷史

public:
    // 在指定行插入文字
    void insertLine(int lineNum, const std::string& text) {
        auto it = lines.begin();
        std::advance(it, lineNum);
        lines.insert(it, text);
        undoStack.push_back("INSERT:" + std::to_string(lineNum));
    }

    // 刪除指定行
    void deleteLine(int lineNum) {
        auto it = lines.begin();
        std::advance(it, lineNum);
        undoStack.push_back("DELETE:" + std::to_string(lineNum) + ":" + *it);
        lines.erase(it);
    }

    // 顯示所有行
    void display() {
        std::cout << "=== 文件內容 ===" << std::endl;
        int lineNum = 0;
        for (const auto& line : lines) {
            std::cout << lineNum++ << ": " << line << std::endl;
        }
    }

    // Undo 操作
    void undo() {
        if (undoStack.empty()) {
            std::cout << "沒有可撤銷的操作" << std::endl;
            return;
        }

        std::string lastOp = undoStack.back();
        undoStack.pop_back();

        // 解析操作並執行反向操作
        if (lastOp.find("INSERT:") == 0) {
            int lineNum = std::stoi(lastOp.substr(7));
            auto it = lines.begin();
            std::advance(it, lineNum);
            lines.erase(it);
        } else if (lastOp.find("DELETE:") == 0) {
            size_t pos1 = lastOp.find(':', 7);
            int lineNum = std::stoi(lastOp.substr(7, pos1 - 7));
            std::string text = lastOp.substr(pos1 + 1);
            auto it = lines.begin();
            std::advance(it, lineNum);
            lines.insert(it, text);
        }
    }
};

int main() {
    TextEditor editor;

    editor.insertLine(0, "第一行");
    editor.insertLine(1, "第二行");
    editor.insertLine(2, "第三行");
    editor.display();

    std::cout << "\n刪除第 1 行後:" << std::endl;
    editor.deleteLine(1);
    editor.display();

    std::cout << "\n執行 undo 後:" << std::endl;
    editor.undo();
    editor.display();

    return 0;
}
```

**輸出**：
```
=== 文件內容 ===
0: 第一行
1: 第二行
2: 第三行

刪除第 1 行後:
=== 文件內容 ===
0: 第一行
1: 第三行

執行 undo 後:
=== 文件內容 ===
0: 第一行
1: 第二行
2: 第三行
```

### 場景 3：多項式運算

多項式需要頻繁插入和刪除項，list 的穩定迭代器很有用。

```cpp
#include <list>
#include <iostream>
#include <algorithm>

struct Term {
    int coefficient;  // 係數
    int exponent;     // 指數

    Term(int c, int e) : coefficient(c), exponent(e) {}
};

class Polynomial {
private:
    std::list<Term> terms;  // 按指數降序排列

public:
    void addTerm(int coefficient, int exponent) {
        if (coefficient == 0) return;

        // 找到插入位置（保持降序）
        auto it = terms.begin();
        while (it != terms.end() && it->exponent > exponent) {
            ++it;
        }

        // 如果指數相同，合併係數
        if (it != terms.end() && it->exponent == exponent) {
            it->coefficient += coefficient;
            if (it->coefficient == 0) {
                terms.erase(it);  // 係數為 0，刪除該項
            }
        } else {
            terms.insert(it, Term(coefficient, exponent));
        }
    }

    void display() const {
        if (terms.empty()) {
            std::cout << "0" << std::endl;
            return;
        }

        bool first = true;
        for (const auto& term : terms) {
            if (!first && term.coefficient > 0) {
                std::cout << " + ";
            } else if (term.coefficient < 0) {
                std::cout << " - ";
            }

            int absCoeff = std::abs(term.coefficient);
            if (absCoeff != 1 || term.exponent == 0) {
                std::cout << absCoeff;
            }

            if (term.exponent > 0) {
                std::cout << "x";
                if (term.exponent > 1) {
                    std::cout << "^" << term.exponent;
                }
            }

            first = false;
        }
        std::cout << std::endl;
    }

    Polynomial operator+(const Polynomial& other) const {
        Polynomial result;

        // 合併兩個多項式的所有項
        for (const auto& term : terms) {
            result.addTerm(term.coefficient, term.exponent);
        }
        for (const auto& term : other.terms) {
            result.addTerm(term.coefficient, term.exponent);
        }

        return result;
    }
};

int main() {
    Polynomial p1;
    p1.addTerm(3, 2);   // 3x^2
    p1.addTerm(2, 1);   // 2x
    p1.addTerm(-5, 0);  // -5

    std::cout << "p1 = ";
    p1.display();  // 3x^2 + 2x - 5

    Polynomial p2;
    p2.addTerm(1, 2);   // x^2
    p2.addTerm(-2, 1);  // -2x
    p2.addTerm(7, 0);   // 7

    std::cout << "p2 = ";
    p2.display();  // x^2 - 2x + 7

    Polynomial p3 = p1 + p2;
    std::cout << "p1 + p2 = ";
    p3.display();  // 4x^2 + 2

    return 0;
}
```

**輸出**：
```
p1 = 3x^2 + 2x - 5
p2 = 1x^2 - 2x + 7
p1 + p2 = 4x^2 + 2
```

## 與其他容器的選擇

### 對比表格

| 特性 | list | vector | deque | forward_list |
|------|------|--------|-------|--------------|
| 隨機訪問 | ❌ O(n) | ✅ O(1) | ✅ O(1) | ❌ O(n) |
| 頭部插入 | ✅ O(1) | ❌ O(n) | ✅ O(1) | ✅ O(1) |
| 尾部插入 | ✅ O(1) | ✅ O(1) | ✅ O(1) | ❌ O(n) |
| 中間插入 | ✅ O(1)* | ❌ O(n) | ❌ O(n) | ✅ O(1)* |
| 迭代器穩定 | ✅ 是 | ❌ 否 | ❌ 否 | ✅ 是 |
| 記憶體開銷 | 大 | 小 | 中 | 更小 |
| 緩存友好 | ❌ | ✅ | 中 | ❌ |
| 反向遍歷 | ✅ | ✅ | ✅ | ❌ |

*：已知位置的情況

### 選擇建議

**選擇 list 當**：
- ✅ 需要在中間頻繁插入/刪除元素
- ✅ 需要穩定的迭代器（插入/刪除不影響其他元素）
- ✅ 需要在容器間快速移動元素（splice）
- ✅ 元素的移動/拷貝成本很高
- ✅ 需要雙向遍歷

**選擇 vector 當**：
- ✅ 需要隨機訪問元素
- ✅ 大部分操作在尾部進行
- ✅ 元素數量已知或變化不大
- ✅ 需要高緩存性能
- ✅ 記憶體受限

**選擇 deque 當**：
- ✅ 需要在頭尾兩端插入/刪除
- ✅ 需要隨機訪問但也需要頭部操作
- ✅ 不需要連續記憶體

**選擇 forward_list 當**：
- ✅ 只需要單向遍歷
- ✅ 極度關注記憶體開銷
- ✅ 需要與 C 語言的單向鏈表相容

## 實務建議

### DO（應該做的）

#### 1. ✅ 使用 list 的成員函式而非 STL 算法

```cpp
std::list<int> myList = {5, 2, 8, 1, 9};

// ✅ 推薦：使用成員函式（更高效）
myList.sort();         // O(n log n)，針對 list 優化
myList.reverse();      // O(n)
myList.remove(5);      // O(n)
myList.unique();       // O(n)

// ❌ 避免：使用 STL 算法
// std::sort(myList.begin(), myList.end());  // 編譯錯誤！
// std::reverse(myList.begin(), myList.end());  // 可用但較慢
```

#### 2. ✅ 利用 splice 來移動元素（零成本）

```cpp
std::list<int> list1 = {1, 2, 3};
std::list<int> list2 = {4, 5, 6};

// ✅ 推薦：splice（O(1)，不需要複製）
list1.splice(list1.end(), list2);

// ❌ 避免：複製後刪除（O(n)，有複製開銷）
// list1.insert(list1.end(), list2.begin(), list2.end());
// list2.clear();
```

#### 3. ✅ 使用 emplace_front/emplace_back 減少複製

```cpp
struct Point {
    int x, y;
    Point(int x, int y) : x(x), y(y) {
        std::cout << "構造 Point(" << x << ", " << y << ")" << std::endl;
    }
};

std::list<Point> points;

// ✅ 推薦：直接在容器內構造（零複製）
points.emplace_back(10, 20);

// ❌ 避免：先構造再複製/移動
// points.push_back(Point(10, 20));
```

#### 4. ✅ 利用迭代器穩定性避免重新查找

```cpp
std::list<int> myList = {1, 2, 3, 4, 5};

// ✅ 推薦：保存迭代器後直接使用
auto it = std::find(myList.begin(), myList.end(), 3);
if (it != myList.end()) {
    myList.insert(it, 99);  // 在 3 之前插入 99
    myList.erase(it);       // 刪除 3
}

// ❌ 避免：每次都重新查找
// myList.insert(std::find(myList.begin(), myList.end(), 3), 99);
// myList.erase(std::find(myList.begin(), myList.end(), 3));
```

#### 5. ✅ 使用 remove_if 配合 lambda 表達式

```cpp
std::list<int> myList = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

// ✅ 推薦：使用 remove_if（簡潔高效）
myList.remove_if([](int x) { return x % 2 == 0; });  // 刪除偶數

// ❌ 避免：手動遍歷刪除
// for (auto it = myList.begin(); it != myList.end(); ) {
//     if (*it % 2 == 0) {
//         it = myList.erase(it);
//     } else {
//         ++it;
//     }
// }
```

#### 6. ✅ 需要查找時結合 unordered_map 使用

```cpp
#include <list>
#include <unordered_map>

// ✅ 推薦：組合使用實現 O(1) 查找 + O(1) 刪除
std::list<int> dataList;
std::unordered_map<int, std::list<int>::iterator> dataMap;

// 插入
dataList.push_back(42);
dataMap[42] = std::prev(dataList.end());

// O(1) 查找和刪除
auto mapIt = dataMap.find(42);
if (mapIt != dataMap.end()) {
    dataList.erase(mapIt->second);  // O(1)
    dataMap.erase(mapIt);
}
```

#### 7. ✅ 使用 auto 和範圍 for 簡化代碼

```cpp
std::list<int> myList = {1, 2, 3, 4, 5};

// ✅ 推薦：簡潔易讀
for (auto& val : myList) {
    val *= 2;  // 修改元素
}

for (const auto& val : myList) {
    std::cout << val << " ";  // 唯讀訪問
}
```

#### 8. ✅ 在遍歷時刪除元素要正確更新迭代器

```cpp
std::list<int> myList = {1, 2, 3, 4, 5};

// ✅ 推薦：使用 erase 的返回值
for (auto it = myList.begin(); it != myList.end(); ) {
    if (*it % 2 == 0) {
        it = myList.erase(it);  // erase 返回下一個有效迭代器
    } else {
        ++it;
    }
}
```

### DON'T（不應該做的）

#### 1. ❌ 不要對小型元素使用 list

```cpp
// ❌ 避免：記憶體浪費嚴重
std::list<char> charList;     // 每個 char (1 byte) + 16 bytes 指標
std::list<bool> boolList;     // 每個 bool (1 byte) + 16 bytes 指標

// ✅ 推薦：使用 vector
std::vector<char> charVec;    // 連續儲存，無額外開銷
std::vector<bool> boolVec;    // 特化版本，位元壓縮
```

#### 2. ❌ 不要嘗試隨機訪問

```cpp
std::list<int> myList = {1, 2, 3, 4, 5};

// ❌ 避免：編譯錯誤
// int val = myList[2];
// int val = myList.at(2);

// ❌ 避免：雖然可行但效率很低 O(n)
auto it = myList.begin();
std::advance(it, 1000);  // 需要遍歷 1000 次

// ✅ 推薦：如果需要隨機訪問，使用 vector
std::vector<int> myVec = {1, 2, 3, 4, 5};
int val = myVec[2];  // O(1)
```

#### 3. ❌ 不要忘記 splice 會修改源容器

```cpp
std::list<int> list1 = {1, 2, 3};
std::list<int> list2 = {4, 5, 6};

// ❌ 常見錯誤：以為 splice 會複製
list1.splice(list1.end(), list2);
// list2 現在是空的！不是 {4, 5, 6}

// ✅ 如果需要複製，使用 insert
list1.insert(list1.end(), list2.begin(), list2.end());
```

#### 4. ❌ 不要在性能關鍵路徑使用 list

```cpp
// ❌ 避免：緩存性能差，遍歷慢
std::list<int> data(1000000);
for (int val : data) {  // 緩存未命中率高
    // 大量計算...
}

// ✅ 推薦：使用 vector
std::vector<int> data(1000000);
for (int val : data) {  // 連續記憶體，緩存友好
    // 大量計算...
}
```

#### 5. ❌ 不要忽略記憶體碎片問題

```cpp
// ❌ 問題：大量插入刪除可能導致記憶體碎片
std::list<int> myList;
for (int i = 0; i < 1000000; ++i) {
    myList.push_back(i);
}
// 100 萬個節點分散在堆記憶體中

// ✅ 解決：考慮使用記憶體池或 vector
```

#### 6. ❌ 不要對 list 使用 std::sort

```cpp
std::list<int> myList = {5, 2, 8, 1, 9};

// ❌ 錯誤：編譯失敗
// std::sort(myList.begin(), myList.end());

// ✅ 正確：使用成員函式
myList.sort();

// ✅ 自訂比較
myList.sort([](int a, int b) { return a > b; });  // 降序
```

#### 7. ❌ 不要依賴元素的記憶體位置

```cpp
std::list<int> myList = {1, 2, 3};

auto it = myList.begin();
int* ptr = &(*it);  // 取得元素位址

myList.push_front(0);  // 插入新元素

// ⚠️ ptr 仍然有效（這是 list 的優勢）
// 但不要假設元素在記憶體中連續
// int* next = ptr + 1;  // 錯誤！不是下一個元素
```

#### 8. ❌ 不要在不需要雙向遍歷時使用 list

```cpp
// ❌ 浪費：只需要單向遍歷卻用雙向鏈表
std::list<int> myList = {1, 2, 3, 4, 5};
for (auto it = myList.begin(); it != myList.end(); ++it) {
    // 只向前遍歷...
}

// ✅ 推薦：使用 forward_list（節省一個指標的空間）
std::forward_list<int> myForwardList = {1, 2, 3, 4, 5};
for (auto it = myForwardList.begin(); it != myForwardList.end(); ++it) {
    // 單向遍歷
}
```

## 小結

### 核心概念總結

1. **雙向鏈表結構**：每個節點包含資料和兩個指標（前驅、後繼），散布在堆記憶體中
2. **插入/刪除 O(1)**：在已知位置插入/刪除非常高效，但查找需要 O(n)
3. **迭代器穩定性**：插入/刪除不會使其他迭代器失效（除了被刪除的元素）
4. **記憶體開銷大**：每個元素需要額外 16 bytes（64 位元系統），約為 vector 的 5 倍
5. **無隨機訪問**：不支援 `[]` 和 `at()`，必須使用迭代器遍歷

### 關鍵記憶點

- ✅ **適合場景**：頻繁在中間插入/刪除，需要穩定迭代器
- ❌ **不適合場景**：需要隨機訪問、記憶體受限、小型元素、性能關鍵路徑
- 🔧 **核心優勢**：splice 操作（零成本移動）、迭代器穩定性
- ⚠️ **主要問題**：記憶體開銷大、緩存性能差、無隨機訪問

### 一句話總結

**`std::list` 是雙向鏈表容器，擅長任意位置的插入/刪除和穩定迭代器管理，但犧牲了隨機訪問能力和記憶體效率，適合需要頻繁中間操作且不需要快速訪問的場景。**
