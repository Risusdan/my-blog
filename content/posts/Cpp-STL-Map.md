+++
date = '2025-12-22T12:00:00+08:00'
draft = false
title = '[C++] STL Map（映射容器）'
categories = ['C++ Notes']
tags = ['C++', 'STL', 'Container']
+++

## 什麼是 Map？

`std::map` 是 C++ STL 中的**關聯容器**，儲存**鍵值對**（key-value pairs），鍵是唯一的，並且**自動按鍵排序**。

**與其他容器的主要區別**：
- 與 `unordered_map`：map 有序（紅黑樹），unordered_map 無序（哈希表）
- 與 `multimap`：map 的鍵唯一，multimap 允許重複鍵
- 與 `set`：map 儲存鍵值對，set 只儲存鍵
- 與 `vector`：map 通過鍵訪問，vector 通過索引訪問

**適用場景**：
- 需要鍵值對映射關係
- 需要按鍵排序
- 需要快速查找、插入、刪除（O(log n)）
- 鍵必須唯一
- 需要範圍查詢或有序遍歷

## 內部實現原理

### 底層數據結構

`std::map` 使用**紅黑樹**（Red-Black Tree）實現，這是一種自平衡二元搜尋樹。

**紅黑樹特性**：
1. 每個節點是紅色或黑色
2. 根節點是黑色
3. 所有葉子節點（NIL）是黑色
4. 紅色節點的兩個子節點都是黑色（不能有連續的紅色節點）
5. 從任一節點到其每個葉子的路徑都包含相同數量的黑色節點

這些特性保證了樹的**大致平衡**，確保操作的時間複雜度為 O(log n)。

### 記憶體布局示意圖

```
std::map<int, string> myMap = {{3, "three"}, {1, "one"}, {5, "five"}, {2, "two"}, {4, "four"}};

紅黑樹結構（示意）：
                    ┌─────────┐
                    │ 3:"three"│ (黑)
                    └─────────┘
                   /             \
            ┌─────────┐       ┌─────────┐
            │ 1:"one" │ (黑)  │ 5:"five"│ (黑)
            └─────────┘       └─────────┘
                 \            /
            ┌─────────┐  ┌─────────┐
            │ 2:"two" │  │ 4:"four"│
            └─────────┘  └─────────┘
              (紅)          (紅)

中序遍歷結果（自動排序）：1, 2, 3, 4, 5

每個節點包含：
- 鍵（key）
- 值（value）
- 左子節點指標
- 右子節點指標
- 父節點指標
- 顏色標記（紅/黑）
```

**特點**：
1. **自動排序**：中序遍歷得到有序序列
2. **平衡性**：樹高保持在 O(log n)
3. **唯一鍵**：每個鍵只出現一次
4. **非連續記憶體**：節點散布在堆記憶體中

### 關鍵操作的實現機制

**查找操作**：
```
從根節點開始二元搜尋：
- 如果 key == 當前節點的鍵，找到
- 如果 key < 當前節點的鍵，往左子樹查找
- 如果 key > 當前節點的鍵，往右子樹查找
時間複雜度：O(log n)
```

**插入操作**：
```
1. 按照 BST 規則插入新節點（紅色）
2. 如果違反紅黑樹性質，進行旋轉和重新著色
3. 最多 O(log n) 次旋轉
時間複雜度：O(log n)
```

**刪除操作**：
```
1. 按照 BST 規則刪除節點
2. 如果違反紅黑樹性質，進行旋轉和重新著色
3. 最多 O(log n) 次旋轉
時間複雜度：O(log n)
```

## 基本使用

### 頭文件和宣告

```cpp
#include <map>
#include <iostream>
#include <string>

std::map<int, std::string> myMap;                    // 空 map
std::map<int, std::string> myMap2 = {{1, "one"}, {2, "two"}};  // 初始化列表
std::map<std::string, int> myMap3;                   // string -> int 映射
```

### 構造方法

```cpp
#include <map>
#include <iostream>
#include <string>

int main() {
    // 預設構造
    std::map<int, std::string> map1;

    // 初始化列表
    std::map<int, std::string> map2 = {
        {1, "one"},
        {2, "two"},
        {3, "three"}
    };

    // 拷貝構造
    std::map<int, std::string> map3(map2);

    // 移動構造
    std::map<int, std::string> map4(std::move(map3));

    // 範圍構造
    std::pair<int, std::string> arr[] = {{4, "four"}, {5, "five"}};
    std::map<int, std::string> map5(arr, arr + 2);

    // 自訂比較函式
    std::map<int, std::string, std::greater<int>> map6;  // 降序排列

    return 0;
}
```

### 常用成員函式

| 函式 | 功能 | 時間複雜度 |
|------|------|-----------|
| `operator[]` | 訪問或插入元素 | O(log n) |
| `at(key)` | 訪問元素（有邊界檢查） | O(log n) |
| `insert(pair)` | 插入鍵值對 | O(log n) |
| `emplace(key, val)` | 原地構造並插入 | O(log n) |
| `erase(key)` | 刪除指定鍵 | O(log n) |
| `find(key)` | 查找鍵，返回迭代器 | O(log n) |
| `count(key)` | 計算鍵的出現次數（0 或 1） | O(log n) |
| `lower_bound(key)` | 返回 >= key 的第一個元素 | O(log n) |
| `upper_bound(key)` | 返回 > key 的第一個元素 | O(log n) |
| `equal_range(key)` | 返回等於 key 的範圍 | O(log n) |
| `size()` | 返回元素個數 | O(1) |
| `empty()` | 判斷是否為空 | O(1) |
| `clear()` | 清空所有元素 | O(n) |

## 性能分析

### 時間複雜度表格

| 操作 | map | unordered_map | vector |
|------|-----|---------------|--------|
| 查找 | **O(log n)** | O(1) 平均 | O(n) |
| 插入 | **O(log n)** | O(1) 平均 | O(n) |
| 刪除 | **O(log n)** | O(1) 平均 | O(n) |
| 有序遍歷 | **O(n)** | ❌ | 需排序 O(n log n) |
| 範圍查詢 | **O(log n + k)** | ❌ | O(n) |
| 空間開銷 | 大（節點 + 指標 + 顏色） | 大（哈希表） | 小 |

### 空間複雜度分析

**記憶體開銷**：
```cpp
// 64 位元系統
sizeof(map<int, int>::node):
  key:    4 bytes
  value:  4 bytes
  left:   8 bytes (指標)
  right:  8 bytes (指標)
  parent: 8 bytes (指標)
  color:  1 byte
  總計：  ~40 bytes（含對齊）

100 個元素的 map<int, int> ≈ 4000 bytes
100 個元素的 vector<pair<int, int>> ≈ 800 bytes

map 記憶體開銷約為 vector 的 5 倍
```

### 何時性能最優/最差

**性能最優**：
- 需要有序儲存和遍歷
- 需要範圍查詢（lower_bound, upper_bound）
- 插入/刪除/查找頻繁且分散
- 需要穩定的 O(log n) 性能保證

**性能最差**：
- 只需要快速查找（unordered_map 更快）
- 資料量很小（vector 可能更快）
- 不需要有序性
- 記憶體非常受限

## 常見操作示例

### 插入元素

```cpp
#include <map>
#include <iostream>
#include <string>

int main() {
    std::map<int, std::string> myMap;

    // 方法1：使用 insert()
    myMap.insert({1, "one"});
    myMap.insert(std::make_pair(2, "two"));
    myMap.insert(std::pair<int, std::string>(3, "three"));

    // 方法2：使用 operator[]
    myMap[4] = "four";
    myMap[5] = "five";

    // 方法3：使用 emplace()（原地構造）
    myMap.emplace(6, "six");

    // 檢查插入是否成功
    auto result = myMap.insert({1, "ONE"});  // 鍵 1 已存在
    if (!result.second) {
        std::cout << "插入失敗，鍵 1 已存在" << std::endl;
    }

    // 輸出
    for (const auto& pair : myMap) {
        std::cout << pair.first << ": " << pair.second << std::endl;
    }

    return 0;
}
```

**輸出**：
```
插入失敗，鍵 1 已存在
1: one
2: two
3: three
4: four
5: five
6: six
```

### 訪問元素

```cpp
#include <map>
#include <iostream>
#include <string>

int main() {
    std::map<int, std::string> myMap = {
        {1, "one"}, {2, "two"}, {3, "three"}
    };

    // 方法1：使用 operator[]（如果鍵不存在會插入）
    std::cout << "myMap[1] = " << myMap[1] << std::endl;
    std::cout << "myMap[10] = " << myMap[10] << std::endl;  // 插入 {10, ""}
    std::cout << "myMap.size() = " << myMap.size() << std::endl;  // 4

    // 方法2：使用 at()（如果鍵不存在會拋出異常）
    try {
        std::cout << "myMap.at(2) = " << myMap.at(2) << std::endl;
        std::cout << "myMap.at(20) = " << myMap.at(20) << std::endl;  // 異常
    } catch (const std::out_of_range& e) {
        std::cout << "錯誤: " << e.what() << std::endl;
    }

    // 方法3：使用 find()（最安全）
    auto it = myMap.find(3);
    if (it != myMap.end()) {
        std::cout << "找到 3: " << it->second << std::endl;
    }

    it = myMap.find(30);
    if (it == myMap.end()) {
        std::cout << "未找到 30" << std::endl;
    }

    return 0;
}
```

**輸出**：
```
myMap[1] = one
myMap[10] =
myMap.size() = 4
myMap.at(2) = two
錯誤: map::at
找到 3: three
未找到 30
```

### 刪除元素

```cpp
#include <map>
#include <iostream>
#include <string>

int main() {
    std::map<int, std::string> myMap = {
        {1, "one"}, {2, "two"}, {3, "three"}, {4, "four"}, {5, "five"}
    };

    // 方法1：通過鍵刪除
    myMap.erase(3);
    std::cout << "刪除 3 後，size = " << myMap.size() << std::endl;

    // 方法2：通過迭代器刪除
    auto it = myMap.find(2);
    if (it != myMap.end()) {
        myMap.erase(it);
    }

    // 方法3：刪除範圍
    auto first = myMap.find(1);
    auto last = myMap.find(5);
    myMap.erase(first, last);  // 刪除 [1, 5)，即 1 和 4

    std::cout << "剩餘元素: ";
    for (const auto& pair : myMap) {
        std::cout << pair.first << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

**輸出**：
```
刪除 3 後，size = 4
剩餘元素: 5
```

### 查找與計數

```cpp
#include <map>
#include <iostream>
#include <string>

int main() {
    std::map<std::string, int> scores = {
        {"Alice", 95},
        {"Bob", 87},
        {"Charlie", 92},
        {"David", 88}
    };

    // 使用 find() 查找
    auto it = scores.find("Bob");
    if (it != scores.end()) {
        std::cout << it->first << " 的分數: " << it->second << std::endl;
    }

    // 使用 count() 檢查是否存在
    if (scores.count("Charlie") > 0) {
        std::cout << "Charlie 存在" << std::endl;
    }

    if (scores.count("Eve") == 0) {
        std::cout << "Eve 不存在" << std::endl;
    }

    // 使用 contains()（C++20）
    // if (scores.contains("Alice")) {
    //     std::cout << "Alice 存在" << std::endl;
    // }

    return 0;
}
```

**輸出**：
```
Bob 的分數: 87
Charlie 存在
Eve 不存在
```

### 遍歷容器

```cpp
#include <map>
#include <iostream>
#include <string>

int main() {
    std::map<int, std::string> myMap = {
        {3, "three"}, {1, "one"}, {4, "four"}, {2, "two"}
    };

    // 方法1：範圍 for 迴圈（自動按鍵排序）
    std::cout << "方法1（自動排序）: " << std::endl;
    for (const auto& pair : myMap) {
        std::cout << pair.first << ": " << pair.second << std::endl;
    }

    // 方法2：迭代器
    std::cout << "\n方法2（迭代器）: " << std::endl;
    for (auto it = myMap.begin(); it != myMap.end(); ++it) {
        std::cout << it->first << ": " << it->second << std::endl;
    }

    // 方法3：反向遍歷
    std::cout << "\n方法3（反向）: " << std::endl;
    for (auto it = myMap.rbegin(); it != myMap.rend(); ++it) {
        std::cout << it->first << ": " << it->second << std::endl;
    }

    // 方法4：結構化綁定（C++17）
    std::cout << "\n方法4（結構化綁定）: " << std::endl;
    for (const auto& [key, value] : myMap) {
        std::cout << key << ": " << value << std::endl;
    }

    return 0;
}
```

**輸出**：
```
方法1（自動排序）:
1: one
2: two
3: three
4: four

方法2（迭代器）:
1: one
2: two
3: three
4: four

方法3（反向）:
4: four
3: three
2: two
1: one

方法4（結構化綁定）:
1: one
2: two
3: three
4: four
```

### 範圍查詢

```cpp
#include <map>
#include <iostream>

int main() {
    std::map<int, std::string> myMap = {
        {10, "ten"}, {20, "twenty"}, {30, "thirty"},
        {40, "forty"}, {50, "fifty"}
    };

    // lower_bound：>= key 的第一個元素
    auto lb = myMap.lower_bound(25);
    if (lb != myMap.end()) {
        std::cout << "lower_bound(25): " << lb->first << std::endl;  // 30
    }

    // upper_bound：> key 的第一個元素
    auto ub = myMap.upper_bound(30);
    if (ub != myMap.end()) {
        std::cout << "upper_bound(30): " << ub->first << std::endl;  // 40
    }

    // equal_range：等於 key 的範圍
    auto range = myMap.equal_range(30);
    std::cout << "equal_range(30): ";
    for (auto it = range.first; it != range.second; ++it) {
        std::cout << it->first << " ";
    }
    std::cout << std::endl;

    // 查詢範圍 [20, 40)
    std::cout << "\n範圍 [20, 40) 的元素: " << std::endl;
    auto start = myMap.lower_bound(20);
    auto end = myMap.lower_bound(40);
    for (auto it = start; it != end; ++it) {
        std::cout << it->first << ": " << it->second << std::endl;
    }

    return 0;
}
```

**輸出**：
```
lower_bound(25): 30
upper_bound(30): 40
equal_range(30): 30

範圍 [20, 40) 的元素:
20: twenty
30: thirty
```

## 迭代器支持

### 支援的迭代器類型

`std::map` 提供**雙向迭代器**（bidirectional iterator）：

```cpp
#include <map>
#include <iostream>

int main() {
    std::map<int, std::string> myMap = {{1, "one"}, {2, "two"}, {3, "three"}};

    auto it = myMap.begin();

    // 雙向迭代器支援的操作
    ++it;      // 前進
    --it;      // 後退
    it->first;   // 訪問鍵
    it->second;  // 訪問值

    // 不支援的操作
    // it + 3;    // 錯誤！不支援隨機訪問
    // it[2];     // 錯誤！不支援下標操作

    // 使用 std::advance 移動
    std::advance(it, 2);
    std::cout << "移動後: " << it->first << std::endl;

    return 0;
}
```

### 迭代器失效問題

**map 的特點**：插入操作**不會使任何迭代器失效**，刪除操作**只使被刪除元素的迭代器失效**。

```cpp
#include <map>
#include <iostream>

int main() {
    std::map<int, std::string> myMap = {
        {1, "one"}, {2, "two"}, {3, "three"}, {4, "four"}, {5, "five"}
    };

    auto it1 = myMap.find(1);
    auto it2 = myMap.find(3);
    auto it5 = myMap.find(5);

    std::cout << "刪除前: it1=" << it1->first
              << ", it2=" << it2->first
              << ", it5=" << it5->first << std::endl;

    // 刪除 3
    myMap.erase(3);

    // it1 和 it5 仍然有效！
    std::cout << "刪除後: it1=" << it1->first
              << ", it5=" << it5->first << std::endl;
    // it2 已失效，不能使用

    // 插入也不會使迭代器失效
    myMap.insert({6, "six"});
    std::cout << "插入後: it1=" << it1->first
              << ", it5=" << it5->first << std::endl;

    return 0;
}
```

**輸出**：
```
刪除前: it1=1, it2=3, it5=5
刪除後: it1=1, it5=5
插入後: it1=1, it5=5
```

**迭代器失效規則總結**：

| 操作 | 失效範圍 |
|------|----------|
| `insert/emplace` | **無失效** |
| `erase(key)` | **僅被刪除元素的迭代器失效** |
| `erase(iterator)` | **僅該迭代器失效** |
| `clear()` | **所有迭代器失效** |
| `operator[]` | **無失效** |

## 常見陷阱與注意事項

### 1. operator[] 會插入不存在的鍵

```cpp
std::map<int, std::string> myMap = {{1, "one"}, {2, "two"}};

// ❌ 常見錯誤：以為只是查詢
std::string val = myMap[3];  // 插入了 {3, ""}！
std::cout << "size: " << myMap.size() << std::endl;  // 3

// ✅ 正確：使用 find() 或 at()
auto it = myMap.find(3);
if (it != myMap.end()) {
    std::string val = it->second;
}

// 或使用 at()（會拋出異常）
try {
    std::string val = myMap.at(3);
} catch (...) {
    // 處理不存在的情況
}
```

### 2. 鍵必須支援比較運算

```cpp
struct Point {
    int x, y;
};

// ❌ 錯誤：Point 沒有定義 operator<
// std::map<Point, std::string> myMap;  // 編譯錯誤！

// ✅ 解決方案1：定義 operator<
struct Point {
    int x, y;
    bool operator<(const Point& other) const {
        if (x != other.x) return x < other.x;
        return y < other.y;
    }
};

// ✅ 解決方案2：提供自訂比較函式
struct PointCompare {
    bool operator()(const Point& a, const Point& b) const {
        if (a.x != b.x) return a.x < b.x;
        return a.y < b.y;
    }
};
std::map<Point, std::string, PointCompare> myMap;
```

### 3. 不要在遍歷時修改容器結構

```cpp
std::map<int, std::string> myMap = {{1, "one"}, {2, "two"}, {3, "three"}};

// ❌ 錯誤：遍歷時刪除會使迭代器失效
for (auto it = myMap.begin(); it != myMap.end(); ++it) {
    if (it->first == 2) {
        myMap.erase(it);  // it 失效，++it 未定義行為！
    }
}

// ✅ 正確：使用 erase 的返回值
for (auto it = myMap.begin(); it != myMap.end(); ) {
    if (it->first == 2) {
        it = myMap.erase(it);  // erase 返回下一個有效迭代器
    } else {
        ++it;
    }
}
```

### 4. 自訂比較函式要滿足嚴格弱序

```cpp
// ❌ 錯誤：不滿足嚴格弱序
struct BadCompare {
    bool operator()(int a, int b) const {
        return a <= b;  // 錯誤！應該是 <
    }
};
// 這會導致未定義行為

// ✅ 正確：使用 < 運算子
struct GoodCompare {
    bool operator()(int a, int b) const {
        return a < b;
    }
};
```

### 5. 值類型的拷貝開銷

```cpp
struct LargeObject {
    char data[1000];
};

std::map<int, LargeObject> myMap;

// ❌ 效率低：返回拷貝
LargeObject obj = myMap[1];  // 拷貝 1000 bytes

// ✅ 推薦：使用參考
const LargeObject& obj = myMap.at(1);  // 無拷貝

// ✅ 或使用指標
auto it = myMap.find(1);
if (it != myMap.end()) {
    const LargeObject& obj = it->second;  // 無拷貝
}
```

## 實際應用場景

### 場景 1：單字計數器

統計文本中每個單字出現的次數。

```cpp
#include <map>
#include <iostream>
#include <string>
#include <sstream>

std::map<std::string, int> wordCount(const std::string& text) {
    std::map<std::string, int> counts;
    std::istringstream iss(text);
    std::string word;

    while (iss >> word) {
        // 移除標點符號（簡化處理）
        word.erase(std::remove_if(word.begin(), word.end(), ::ispunct), word.end());

        // 轉小寫
        std::transform(word.begin(), word.end(), word.begin(), ::tolower);

        // 計數
        counts[word]++;  // 如果不存在會自動插入 {word, 0} 然後 +1
    }

    return counts;
}

int main() {
    std::string text = "Hello world! Hello C++ programming. "
                       "C++ is great for programming.";

    auto counts = wordCount(text);

    std::cout << "單字出現次數（按字母順序）:" << std::endl;
    for (const auto& [word, count] : counts) {
        std::cout << word << ": " << count << std::endl;
    }

    return 0;
}
```

**輸出**：
```
單字出現次數（按字母順序）:
c: 2
for: 1
great: 1
hello: 2
is: 1
programming: 2
world: 1
```

### 場景 2：學生成績管理系統

```cpp
#include <map>
#include <iostream>
#include <string>
#include <iomanip>

class GradeManager {
private:
    std::map<std::string, std::map<std::string, double>> grades;
    // 外層 map：學生名 -> 內層 map
    // 內層 map：科目 -> 分數

public:
    void addGrade(const std::string& student, const std::string& subject, double score) {
        grades[student][subject] = score;
    }

    double getGrade(const std::string& student, const std::string& subject) const {
        auto studentIt = grades.find(student);
        if (studentIt != grades.end()) {
            auto subjectIt = studentIt->second.find(subject);
            if (subjectIt != studentIt->second.end()) {
                return subjectIt->second;
            }
        }
        return -1;  // 未找到
    }

    double getAverage(const std::string& student) const {
        auto it = grades.find(student);
        if (it == grades.end() || it->second.empty()) {
            return 0;
        }

        double sum = 0;
        for (const auto& [subject, score] : it->second) {
            sum += score;
        }
        return sum / it->second.size();
    }

    void printReport() const {
        std::cout << std::fixed << std::setprecision(2);
        for (const auto& [student, subjects] : grades) {
            std::cout << "\n學生: " << student << std::endl;
            for (const auto& [subject, score] : subjects) {
                std::cout << "  " << subject << ": " << score << std::endl;
            }
            std::cout << "  平均: " << getAverage(student) << std::endl;
        }
    }
};

int main() {
    GradeManager gm;

    gm.addGrade("Alice", "Math", 95);
    gm.addGrade("Alice", "English", 88);
    gm.addGrade("Alice", "Physics", 92);

    gm.addGrade("Bob", "Math", 87);
    gm.addGrade("Bob", "English", 90);

    gm.addGrade("Charlie", "Math", 78);
    gm.addGrade("Charlie", "Physics", 85);

    gm.printReport();

    std::cout << "\nAlice 的數學成績: " << gm.getGrade("Alice", "Math") << std::endl;

    return 0;
}
```

**輸出**：
```
學生: Alice
  English: 88.00
  Math: 95.00
  Physics: 92.00
  平均: 91.67

學生: Bob
  English: 90.00
  Math: 87.00
  平均: 88.50

學生: Charlie
  Math: 78.00
  Physics: 85.00
  平均: 81.50

Alice 的數學成績: 95.00
```

### 場景 3：配置檔案解析器

```cpp
#include <map>
#include <iostream>
#include <string>
#include <fstream>
#include <sstream>

class ConfigParser {
private:
    std::map<std::string, std::string> config;

public:
    bool loadFromString(const std::string& content) {
        std::istringstream iss(content);
        std::string line;

        while (std::getline(iss, line)) {
            // 移除註解
            size_t commentPos = line.find('#');
            if (commentPos != std::string::npos) {
                line = line.substr(0, commentPos);
            }

            // 解析 key=value
            size_t equalPos = line.find('=');
            if (equalPos != std::string::npos) {
                std::string key = line.substr(0, equalPos);
                std::string value = line.substr(equalPos + 1);

                // 移除前後空白
                key.erase(0, key.find_first_not_of(" \t"));
                key.erase(key.find_last_not_of(" \t") + 1);
                value.erase(0, value.find_first_not_of(" \t"));
                value.erase(value.find_last_not_of(" \t") + 1);

                if (!key.empty()) {
                    config[key] = value;
                }
            }
        }

        return true;
    }

    std::string get(const std::string& key, const std::string& defaultValue = "") const {
        auto it = config.find(key);
        return (it != config.end()) ? it->second : defaultValue;
    }

    int getInt(const std::string& key, int defaultValue = 0) const {
        auto it = config.find(key);
        if (it != config.end()) {
            try {
                return std::stoi(it->second);
            } catch (...) {
                return defaultValue;
            }
        }
        return defaultValue;
    }

    void print() const {
        std::cout << "配置項目:" << std::endl;
        for (const auto& [key, value] : config) {
            std::cout << "  " << key << " = " << value << std::endl;
        }
    }
};

int main() {
    std::string configContent = R"(
# 伺服器配置
server_host = localhost
server_port = 8080

# 資料庫配置
db_host = 127.0.0.1
db_port = 3306
db_name = mydb
db_user = admin

# 其他設定
max_connections = 100
timeout = 30  # 秒
)";

    ConfigParser parser;
    parser.loadFromString(configContent);

    parser.print();

    std::cout << "\n查詢配置:" << std::endl;
    std::cout << "Server: " << parser.get("server_host")
              << ":" << parser.getInt("server_port") << std::endl;
    std::cout << "Max connections: " << parser.getInt("max_connections") << std::endl;

    return 0;
}
```

**輸出**：
```
配置項目:
  db_host = 127.0.0.1
  db_name = mydb
  db_port = 3306
  db_user = admin
  max_connections = 100
  server_host = localhost
  server_port = 8080
  timeout = 30

查詢配置:
Server: localhost:8080
Max connections: 100
```

## 與其他容器的選擇

### 對比表格

| 特性 | map | unordered_map | multimap | set |
|------|-----|---------------|----------|-----|
| 有序性 | ✅ 有序 | ❌ 無序 | ✅ 有序 | ✅ 有序 |
| 鍵唯一 | ✅ 唯一 | ✅ 唯一 | ❌ 可重複 | ✅ 唯一 |
| 查找 | O(log n) | **O(1) 平均** | O(log n) | O(log n) |
| 插入 | O(log n) | **O(1) 平均** | O(log n) | O(log n) |
| 刪除 | O(log n) | **O(1) 平均** | O(log n) | O(log n) |
| 範圍查詢 | ✅ | ❌ | ✅ | ✅ |
| 記憶體開銷 | 中 | 大 | 中 | 小 |
| 最壞情況 | **O(log n)** | O(n) | O(log n) | O(log n) |

### 選擇建議

**選擇 map 當**：
- ✅ 需要鍵值對映射
- ✅ 需要按鍵排序
- ✅ 需要範圍查詢
- ✅ 需要穩定的 O(log n) 性能
- ✅ 鍵必須唯一

**選擇 unordered_map 當**：
- ✅ 只需要快速查找（不需要排序）
- ✅ 插入/刪除/查找是主要操作
- ✅ 記憶體不是極度受限
- ✅ 可以接受最壞情況 O(n)

**選擇 multimap 當**：
- ✅ 需要允許重複鍵
- ✅ 需要按鍵排序
- ✅ 一對多映射關係

**選擇 set 當**：
- ✅ 只需要儲存鍵（不需要值）
- ✅ 需要去重和排序

## 實務建議

### DO（應該做的）

#### 1. ✅ 使用 find() 而非 operator[] 查詢

```cpp
std::map<int, std::string> myMap = {{1, "one"}, {2, "two"}};

// ✅ 推薦：使用 find()（不會插入）
auto it = myMap.find(3);
if (it != myMap.end()) {
    std::cout << it->second << std::endl;
}

// ❌ 避免：使用 operator[]（會插入 {3, ""}）
// if (!myMap[3].empty()) {  // 已經插入了！
//     std::cout << myMap[3] << std::endl;
// }
```

#### 2. ✅ 使用 emplace 減少複製

```cpp
std::map<int, std::string> myMap;

// ✅ 推薦：使用 emplace（原地構造）
myMap.emplace(1, "one");

// ❌ 避免：使用 insert（可能有額外複製）
// myMap.insert({1, "one"});
// myMap.insert(std::make_pair(1, "one"));
```

#### 3. ✅ 使用結構化綁定簡化代碼（C++17）

```cpp
std::map<std::string, int> myMap = {{"Alice", 95}, {"Bob", 87}};

// ✅ 推薦：結構化綁定（清晰易讀）
for (const auto& [name, score] : myMap) {
    std::cout << name << ": " << score << std::endl;
}

// ❌ 較繁瑣：使用 pair
// for (const auto& pair : myMap) {
//     std::cout << pair.first << ": " << pair.second << std::endl;
// }
```

#### 4. ✅ 利用 lower_bound/upper_bound 進行範圍查詢

```cpp
std::map<int, std::string> myMap = {
    {10, "ten"}, {20, "twenty"}, {30, "thirty"}, {40, "forty"}
};

// ✅ 推薦：使用 lower_bound/upper_bound
auto start = myMap.lower_bound(15);
auto end = myMap.upper_bound(35);

std::cout << "範圍 [15, 35] 的元素:" << std::endl;
for (auto it = start; it != end; ++it) {
    std::cout << it->first << ": " << it->second << std::endl;
}
```

#### 5. ✅ 使用 at() 進行安全訪問

```cpp
std::map<int, std::string> myMap = {{1, "one"}, {2, "two"}};

// ✅ 推薦：使用 at()（有異常保護）
try {
    std::string val = myMap.at(10);
} catch (const std::out_of_range&) {
    std::cout << "鍵不存在" << std::endl;
}

// ❌ 危險：使用 operator[]
// std::string val = myMap[10];  // 插入 {10, ""}
```

#### 6. ✅ 遍歷時刪除使用 erase 返回值

```cpp
std::map<int, std::string> myMap = {{1, "one"}, {2, "two"}, {3, "three"}};

// ✅ 推薦：使用 erase 的返回值
for (auto it = myMap.begin(); it != myMap.end(); ) {
    if (it->first % 2 == 0) {
        it = myMap.erase(it);  // 返回下一個有效迭代器
    } else {
        ++it;
    }
}
```

#### 7. ✅ 使用 count() 檢查存在性

```cpp
std::map<std::string, int> myMap = {{"Alice", 95}, {"Bob", 87}};

// ✅ 推薦：使用 count()（清晰表達意圖）
if (myMap.count("Charlie") > 0) {
    // 存在
}

// ✅ 也可以：使用 find()
if (myMap.find("Charlie") != myMap.end()) {
    // 存在
}
```

#### 8. ✅ 使用 const 參考傳遞 map

```cpp
// ✅ 推薦：const 參考（避免拷貝）
void processMap(const std::map<int, std::string>& myMap) {
    for (const auto& [key, value] : myMap) {
        // 處理...
    }
}

// ❌ 避免：值傳遞（拷貝整個 map）
// void processMap(std::map<int, std::string> myMap) {
//     // 拷貝開銷大！
// }
```

### DON'T（不應該做的）

#### 1. ❌ 不要使用 operator[] 檢查存在性

```cpp
std::map<int, std::string> myMap = {{1, "one"}, {2, "two"}};

// ❌ 錯誤：會插入不存在的鍵
if (myMap[3].empty()) {  // 已經插入 {3, ""}！
    std::cout << "不存在" << std::endl;
}
std::cout << "size: " << myMap.size() << std::endl;  // 3

// ✅ 正確：使用 find() 或 count()
if (myMap.find(3) == myMap.end()) {
    std::cout << "不存在" << std::endl;
}
std::cout << "size: " << myMap.size() << std::endl;  // 2
```

#### 2. ❌ 不要在不需要排序時使用 map

```cpp
// ❌ 浪費：只需要快速查找，不需要排序
std::map<std::string, int> cache;  // O(log n) 查找

// ✅ 推薦：使用 unordered_map
std::unordered_map<std::string, int> cache;  // O(1) 平均查找
```

#### 3. ❌ 不要修改鍵

```cpp
std::map<int, std::string> myMap = {{1, "one"}, {2, "two"}};

auto it = myMap.find(1);
if (it != myMap.end()) {
    // ❌ 錯誤：不能修改鍵（編譯錯誤）
    // it->first = 10;

    // ✅ 正確：只能修改值
    it->second = "ONE";
}
```

#### 4. ❌ 不要忘記檢查 insert 的返回值

```cpp
std::map<int, std::string> myMap = {{1, "one"}};

// ❌ 忽略返回值：不知道是否插入成功
myMap.insert({1, "ONE"});  // 插入失敗，鍵 1 已存在

// ✅ 檢查返回值
auto [it, success] = myMap.insert({1, "ONE"});
if (!success) {
    std::cout << "插入失敗，鍵已存在" << std::endl;
    // 如果需要更新，使用 operator[] 或 insert_or_assign
    myMap[1] = "ONE";
}
```

#### 5. ❌ 不要對大型值使用值傳遞

```cpp
struct LargeValue {
    char data[10000];
};

std::map<int, LargeValue> myMap;

// ❌ 避免：返回值拷貝
void processValue(LargeValue val) {  // 拷貝 10000 bytes
    // 處理...
}

auto it = myMap.find(1);
if (it != myMap.end()) {
    processValue(it->second);  // 拷貝！
}

// ✅ 推薦：使用 const 參考
void processValue(const LargeValue& val) {
    // 處理...
}

auto it = myMap.find(1);
if (it != myMap.end()) {
    processValue(it->second);  // 無拷貝
}
```

#### 6. ❌ 不要假設插入順序

```cpp
std::map<int, std::string> myMap;

myMap.insert({3, "three"});
myMap.insert({1, "one"});
myMap.insert({2, "two"});

// ❌ 錯誤假設：按插入順序遍歷
// 實際上會按鍵排序：1, 2, 3

for (const auto& [key, value] : myMap) {
    std::cout << key << " ";  // 輸出: 1 2 3（不是 3 1 2）
}
```

#### 7. ❌ 不要在性能關鍵路徑頻繁插入刪除

```cpp
std::map<int, int> myMap;

// ❌ 效率低：頻繁插入刪除（每次 O(log n) + 平衡操作）
for (int i = 0; i < 1000000; ++i) {
    myMap[i] = i * 2;
}
for (int i = 0; i < 500000; ++i) {
    myMap.erase(i);
}

// ✅ 考慮：如果不需要排序，使用 unordered_map
// ✅ 或者：批次操作，減少插入刪除次數
```

#### 8. ❌ 不要忽略自訂型別的比較函式

```cpp
struct Person {
    std::string name;
    int age;
};

// ❌ 錯誤：Person 沒有定義 operator<
// std::map<Person, int> myMap;  // 編譯錯誤！

// ✅ 解決方案：定義 operator< 或提供比較函式
struct Person {
    std::string name;
    int age;

    bool operator<(const Person& other) const {
        if (name != other.name) return name < other.name;
        return age < other.age;
    }
};

std::map<Person, int> myMap;  // OK
```

## 小結

### 核心概念總結

1. **有序鍵值對**：map 儲存鍵值對並自動按鍵排序，基於紅黑樹實現
2. **唯一鍵**：每個鍵只能出現一次，重複插入會失敗
3. **O(log n) 操作**：插入、刪除、查找都是 O(log n)，穩定可靠
4. **範圍查詢**：支援 lower_bound、upper_bound 等範圍操作
5. **迭代器穩定**：插入不使迭代器失效，刪除只使被刪除元素失效

### 關鍵記憶點

- ✅ **適合場景**：需要有序鍵值對、範圍查詢、穩定性能保證
- ❌ **不適合場景**：只需快速查找（用 unordered_map）、不需排序
- 🔧 **核心優勢**：自動排序、範圍查詢、穩定 O(log n)
- ⚠️ **主要問題**：operator[] 會插入、比 unordered_map 慢、記憶體開銷大

### 一句話總結

**`std::map` 是基於紅黑樹的有序關聯容器，儲存唯一鍵值對並自動排序，提供穩定的 O(log n) 查找、插入、刪除性能和範圍查詢能力，適合需要有序性和穩定性能的鍵值映射場景。**
