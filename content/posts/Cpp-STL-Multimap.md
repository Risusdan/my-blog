+++
date = '2025-12-22T13:00:00+08:00'
draft = false
title = '[C++] STL Multimap（多重映射容器）'
categories = ['C++ Notes']
tags = ['C++', 'STL', 'Container']
+++

## 什麼是 Multimap？

`std::multimap` 是 C++ STL 中的**關聯容器**，儲存**鍵值對**且**允許重複鍵**，自動按鍵排序。

**與其他容器的主要區別**：
- 與 `map`：multimap 允許重複鍵，map 的鍵唯一
- 與 `unordered_multimap`：multimap 有序（紅黑樹），unordered_multimap 無序（哈希表）
- 與 `multiset`：multimap 儲存鍵值對，multiset 只儲存鍵

**適用場景**：
- 一對多映射關係（一個鍵對應多個值）
- 需要按鍵排序
- 需要快速查找所有相同鍵的值
- 需要範圍查詢

## 內部實現原理

### 底層數據結構

`std::multimap` 使用**紅黑樹**實現，與 `map` 相同，但允許重複鍵。

### 記憶體布局示意圖

```
std::multimap<int, string> myMap = {{1, "a"}, {2, "b"}, {1, "c"}, {3, "d"}, {2, "e"}};

紅黑樹結構（示意）：
                ┌──────────┐
                │ 2:"b"    │ (黑)
                └──────────┘
               /            \
        ┌──────────┐    ┌──────────┐
        │ 1:"a"    │    │ 3:"d"    │
        └──────────┘    └──────────┘
            \           /
        ┌──────────┐ ┌──────────┐
        │ 1:"c"    │ │ 2:"e"    │
        └──────────┘ └──────────┘

中序遍歷（自動排序，相同鍵按插入順序）：
{1, "a"}, {1, "c"}, {2, "b"}, {2, "e"}, {3, "d"}
```

## 基本使用

### 頭文件和宣告

```cpp
#include <map>
#include <iostream>
#include <string>

std::multimap<int, std::string> myMap;
std::multimap<int, std::string> myMap2 = {{1, "one"}, {1, "uno"}, {2, "two"}};
```

### 構造方法

```cpp
#include <map>
#include <iostream>
#include <string>

int main() {
    // 預設構造
    std::multimap<int, std::string> map1;

    // 初始化列表（允許重複鍵）
    std::multimap<int, std::string> map2 = {
        {1, "one"}, {1, "uno"}, {1, "一"},
        {2, "two"}, {2, "dos"}
    };

    // 拷貝構造
    std::multimap<int, std::string> map3(map2);

    // 範圍構造
    std::pair<int, std::string> arr[] = {{3, "three"}, {3, "tres"}};
    std::multimap<int, std::string> map4(arr, arr + 2);

    return 0;
}
```

### 常用成員函式

| 函式 | 功能 | 時間複雜度 |
|------|------|-----------|
| `insert(pair)` | 插入鍵值對（總是成功） | O(log n) |
| `emplace(key, val)` | 原地構造並插入 | O(log n) |
| `erase(key)` | 刪除所有指定鍵 | O(log n + k) |
| `find(key)` | 查找鍵，返回第一個匹配 | O(log n) |
| `count(key)` | 計算鍵的出現次數 | O(log n + k) |
| `equal_range(key)` | 返回等於 key 的範圍 | O(log n) |
| `lower_bound(key)` | 返回 >= key 的第一個元素 | O(log n) |
| `upper_bound(key)` | 返回 > key 的第一個元素 | O(log n) |
| `size()` | 返回元素個數 | O(1) |
| `clear()` | 清空所有元素 | O(n) |

**注意**：multimap **沒有** `operator[]`，因為一個鍵對應多個值。

## 常見操作示例

### 插入元素

```cpp
#include <map>
#include <iostream>
#include <string>

int main() {
    std::multimap<int, std::string> myMap;

    // 方法1：使用 insert()（總是成功）
    myMap.insert({1, "one"});
    myMap.insert({1, "uno"});    // 相同鍵，成功插入
    myMap.insert({1, "一"});     // 再次插入相同鍵

    // 方法2：使用 emplace()
    myMap.emplace(2, "two");
    myMap.emplace(2, "dos");

    // 輸出（按鍵排序）
    std::cout << "myMap 內容:" << std::endl;
    for (const auto& [key, value] : myMap) {
        std::cout << key << ": " << value << std::endl;
    }

    return 0;
}
```

**輸出**：
```
myMap 內容:
1: one
1: uno
1: 一
2: two
2: dos
```

### 查找元素

```cpp
#include <map>
#include <iostream>
#include <string>

int main() {
    std::multimap<int, std::string> myMap = {
        {1, "one"}, {1, "uno"}, {2, "two"}, {2, "dos"}, {3, "three"}
    };

    // 方法1：使用 find()（返回第一個匹配）
    auto it = myMap.find(1);
    if (it != myMap.end()) {
        std::cout << "找到第一個鍵 1: " << it->second << std::endl;
    }

    // 方法2：使用 count() 統計數量
    int count = myMap.count(1);
    std::cout << "鍵 1 出現 " << count << " 次" << std::endl;

    // 方法3：使用 equal_range() 查找所有匹配
    auto range = myMap.equal_range(2);
    std::cout << "\n鍵 2 的所有值:" << std::endl;
    for (auto it = range.first; it != range.second; ++it) {
        std::cout << "  " << it->second << std::endl;
    }

    return 0;
}
```

**輸出**：
```
找到第一個鍵 1: one
鍵 1 出現 2 次

鍵 2 的所有值:
  two
  dos
```

### 刪除元素

```cpp
#include <map>
#include <iostream>
#include <string>

int main() {
    std::multimap<int, std::string> myMap = {
        {1, "one"}, {1, "uno"}, {2, "two"}, {2, "dos"}, {3, "three"}
    };

    std::cout << "初始大小: " << myMap.size() << std::endl;

    // 方法1：刪除所有指定鍵
    size_t removed = myMap.erase(1);  // 刪除所有鍵為 1 的元素
    std::cout << "刪除了 " << removed << " 個元素" << std::endl;
    std::cout << "刪除後大小: " << myMap.size() << std::endl;

    // 方法2：通過迭代器刪除單個元素
    auto it = myMap.find(2);
    if (it != myMap.end()) {
        myMap.erase(it);  // 只刪除這一個
    }

    std::cout << "\n剩餘元素:" << std::endl;
    for (const auto& [key, value] : myMap) {
        std::cout << key << ": " << value << std::endl;
    }

    return 0;
}
```

**輸出**：
```
初始大小: 5
刪除了 2 個元素
刪除後大小: 3

剩餘元素:
2: dos
3: three
```

### 遍歷相同鍵的所有值

```cpp
#include <map>
#include <iostream>
#include <string>

int main() {
    std::multimap<std::string, int> scores = {
        {"Alice", 95}, {"Bob", 87}, {"Alice", 92}, {"Charlie", 88}, {"Alice", 90}
    };

    // 查找 Alice 的所有分數
    std::string name = "Alice";
    auto range = scores.equal_range(name);

    std::cout << name << " 的所有分數:" << std::endl;
    int total = 0;
    int count = 0;

    for (auto it = range.first; it != range.second; ++it) {
        std::cout << "  " << it->second << std::endl;
        total += it->second;
        count++;
    }

    if (count > 0) {
        std::cout << "平均分數: " << (double)total / count << std::endl;
    }

    return 0;
}
```

**輸出**：
```
Alice 的所有分數:
  95
  92
  90
平均分數: 92.3333
```

## 實際應用場景

### 場景 1：圖的鄰接表（有向圖）

```cpp
#include <map>
#include <iostream>

class Graph {
private:
    std::multimap<int, int> adjList;  // 頂點 -> 相鄰頂點

public:
    void addEdge(int from, int to) {
        adjList.insert({from, to});
    }

    void printGraph() const {
        int currentVertex = -1;

        for (const auto& [from, to] : adjList) {
            if (from != currentVertex) {
                if (currentVertex != -1) std::cout << std::endl;
                std::cout << "頂點 " << from << " -> ";
                currentVertex = from;
            } else {
                std::cout << ", ";
            }
            std::cout << to;
        }
        std::cout << std::endl;
    }

    void printNeighbors(int vertex) const {
        auto range = adjList.equal_range(vertex);
        std::cout << "頂點 " << vertex << " 的鄰居: ";

        for (auto it = range.first; it != range.second; ++it) {
            std::cout << it->second << " ";
        }
        std::cout << std::endl;
    }
};

int main() {
    Graph graph;

    graph.addEdge(0, 1);
    graph.addEdge(0, 2);
    graph.addEdge(1, 2);
    graph.addEdge(1, 3);
    graph.addEdge(2, 3);
    graph.addEdge(3, 0);

    std::cout << "圖結構:" << std::endl;
    graph.printGraph();

    std::cout << "\n查詢鄰居:" << std::endl;
    graph.printNeighbors(0);
    graph.printNeighbors(1);

    return 0;
}
```

**輸出**：
```
圖結構:
頂點 0 -> 1, 2
頂點 1 -> 2, 3
頂點 2 -> 3
頂點 3 -> 0

查詢鄰居:
頂點 0 的鄰居: 1 2
頂點 1 的鄰居: 2 3
```

### 場景 2：電話簿（一人多號碼）

```cpp
#include <map>
#include <iostream>
#include <string>

class PhoneBook {
private:
    std::multimap<std::string, std::string> contacts;

public:
    void addContact(const std::string& name, const std::string& phone) {
        contacts.insert({name, phone});
    }

    void printPhones(const std::string& name) const {
        auto range = contacts.equal_range(name);
        int count = std::distance(range.first, range.second);

        if (count == 0) {
            std::cout << name << " 沒有電話號碼" << std::endl;
            return;
        }

        std::cout << name << " 的電話號碼:" << std::endl;
        for (auto it = range.first; it != range.second; ++it) {
            std::cout << "  " << it->second << std::endl;
        }
    }

    void printAll() const {
        std::string currentName;

        for (const auto& [name, phone] : contacts) {
            if (name != currentName) {
                std::cout << "\n" << name << ":" << std::endl;
                currentName = name;
            }
            std::cout << "  " << phone << std::endl;
        }
    }
};

int main() {
    PhoneBook book;

    book.addContact("Alice", "555-1234");
    book.addContact("Alice", "555-5678");
    book.addContact("Bob", "555-8765");
    book.addContact("Charlie", "555-4321");
    book.addContact("Alice", "555-9999");

    book.printPhones("Alice");

    std::cout << "\n所有聯絡人:" << std::endl;
    book.printAll();

    return 0;
}
```

**輸出**：
```
Alice 的電話號碼:
  555-1234
  555-5678
  555-9999

所有聯絡人:

Alice:
  555-1234
  555-5678
  555-9999

Bob:
  555-8765

Charlie:
  555-4321
```

### 場景 3：倒排索引（搜索引擎）

```cpp
#include <map>
#include <iostream>
#include <string>
#include <sstream>
#include <set>

class InvertedIndex {
private:
    std::multimap<std::string, int> index;  // 單字 -> 文檔 ID

public:
    void addDocument(int docId, const std::string& content) {
        std::istringstream iss(content);
        std::string word;

        while (iss >> word) {
            // 轉小寫
            for (char& c : word) {
                c = std::tolower(c);
            }
            index.insert({word, docId});
        }
    }

    std::set<int> search(const std::string& word) const {
        std::set<int> results;
        auto range = index.equal_range(word);

        for (auto it = range.first; it != range.second; ++it) {
            results.insert(it->second);
        }

        return results;
    }

    void printIndex() const {
        std::string currentWord;

        for (const auto& [word, docId] : index) {
            if (word != currentWord) {
                if (!currentWord.empty()) std::cout << std::endl;
                std::cout << word << ": ";
                currentWord = word;
            } else {
                std::cout << ", ";
            }
            std::cout << docId;
        }
        std::cout << std::endl;
    }
};

int main() {
    InvertedIndex idx;

    idx.addDocument(1, "The quick brown fox");
    idx.addDocument(2, "The lazy dog");
    idx.addDocument(3, "Quick brown dogs");

    std::cout << "倒排索引:" << std::endl;
    idx.printIndex();

    std::cout << "\n搜尋 'quick':" << std::endl;
    auto results = idx.search("quick");
    for (int docId : results) {
        std::cout << "文檔 " << docId << std::endl;
    }

    return 0;
}
```

**輸出**：
```
倒排索引:
brown: 1, 3
dog: 2
dogs: 3
fox: 1
lazy: 2
quick: 1, 3
the: 1, 2

搜尋 'quick':
文檔 1
文檔 3
```

## 與其他容器的選擇

### 對比表格

| 特性 | multimap | map | unordered_multimap |
|------|----------|-----|-------------------|
| 鍵唯一 | ❌ 可重複 | ✅ 唯一 | ❌ 可重複 |
| 有序性 | ✅ 有序 | ✅ 有序 | ❌ 無序 |
| operator[] | ❌ | ✅ | ❌ |
| 查找 | O(log n) | O(log n) | **O(1) 平均** |
| 插入 | O(log n) | O(log n) | **O(1) 平均** |

### 選擇建議

**選擇 multimap 當**：
- ✅ 需要一對多映射
- ✅ 需要按鍵排序
- ✅ 需要範圍查詢

**選擇 map 當**：
- ✅ 鍵必須唯一
- ✅ 需要 operator[] 訪問

**選擇 unordered_multimap 當**：
- ✅ 只需要快速查找（不需要排序）

## 實務建議

### DO（應該做的）

#### 1. ✅ 使用 equal_range 遍歷相同鍵

```cpp
std::multimap<int, std::string> myMap = {{1, "a"}, {1, "b"}, {2, "c"}};

// ✅ 推薦：使用 equal_range
auto range = myMap.equal_range(1);
for (auto it = range.first; it != range.second; ++it) {
    std::cout << it->second << std::endl;
}
```

#### 2. ✅ 使用 count 檢查鍵的數量

```cpp
std::multimap<int, std::string> myMap = {{1, "a"}, {1, "b"}};

// ✅ 推薦：使用 count
if (myMap.count(1) > 1) {
    std::cout << "鍵 1 有多個值" << std::endl;
}
```

### DON'T（不應該做的）

#### 1. ❌ 不要使用 operator[]

```cpp
std::multimap<int, std::string> myMap;

// ❌ 錯誤：multimap 沒有 operator[]
// myMap[1] = "value";  // 編譯錯誤！

// ✅ 正確：使用 insert
myMap.insert({1, "value"});
```

#### 2. ❌ 不要忘記 erase(key) 會刪除所有相同鍵

```cpp
std::multimap<int, std::string> myMap = {{1, "a"}, {1, "b"}, {1, "c"}};

// ❌ 注意：刪除所有鍵為 1 的元素
myMap.erase(1);  // 全部刪除！

// ✅ 如果只想刪除一個，使用迭代器
auto it = myMap.find(1);
if (it != myMap.end()) {
    myMap.erase(it);  // 只刪除一個
}
```

## 小結

### 核心概念總結

1. **允許重複鍵**：一個鍵可以對應多個值
2. **自動排序**：按鍵排序，相同鍵按插入順序
3. **無 operator[]**：因為一個鍵對應多個值
4. **equal_range**：查找所有相同鍵的值
5. **一對多映射**：適合需要多值關聯的場景

### 一句話總結

**`std::multimap` 是基於紅黑樹的有序關聯容器，允許重複鍵並自動排序，提供 O(log n) 性能，適合一對多映射和需要按鍵排序的場景。**
