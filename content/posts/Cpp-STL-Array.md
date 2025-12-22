+++
date = '2025-12-22T11:00:00+08:00'
draft = false
title = '[C++] STL Array（固定大小陣列）'
categories = ['C++ Notes']
tags = ['C++', 'STL', 'Container']
+++

## 什麼是 Array？

`std::array` 是 C++11 引入的**固定大小陣列容器**，在編譯期確定大小，封裝了傳統 C 陣列並提供 STL 容器的介面。

**與其他容器的主要區別**：
- 與 C 陣列：提供 STL 容器介面（迭代器、size()、at() 等），更安全
- 與 `vector`：大小固定，不能動態擴展，但無記憶體分配開銷
- 與 `deque`：連續記憶體，固定大小，性能更好但無法增長
- 與 `list`：連續記憶體，支援隨機訪問

**適用場景**：
- 元素數量在編譯期已知且固定
- 需要 STL 容器介面的固定陣列
- 需要在棧上分配且避免堆記憶體分配
- 需要將陣列作為值傳遞或返回
- 性能關鍵且大小固定的場景

## 內部實現原理

### 底層數據結構

`std::array` 本質上是對**原生 C 陣列的封裝**，內部實作極為簡單：

```cpp
template<typename T, size_t N>
struct array {
    T elements[N];  // 原生 C 陣列

    // 提供 STL 容器介面
    size_t size() const { return N; }
    T& operator[](size_t i) { return elements[i]; }
    // ... 其他成員函式
};
```

### 記憶體布局示意圖

```
std::array<int, 5> arr = {1, 2, 3, 4, 5};

記憶體布局（棧上）：
┌──────────────────────────────┐
│  arr[0]  arr[1]  arr[2]  arr[3]  arr[4] │
│    1       2       3       4       5     │
└──────────────────────────────────────────┘
  連續記憶體，無額外開銷

C 陣列: int arr[5] = {1, 2, 3, 4, 5};
記憶體布局（完全相同）：
┌──────────────────────────────┐
│  arr[0]  arr[1]  arr[2]  arr[3]  arr[4] │
│    1       2       3       4       5     │
└──────────────────────────────────────────┘
```

**特點**：
1. **零開銷封裝**：`std::array` 的大小和性能與原生 C 陣列完全相同
2. **棧分配**：預設在棧上分配（除非是全域或靜態變數）
3. **編譯期大小**：大小必須是編譯期常數
4. **連續記憶體**：所有元素連續儲存，緩存友好

### 關鍵操作的實現機制

**訪問操作**：
```
arr[i]  // 直接指標運算：*(arr.elements + i)
        // 時間複雜度：O(1)
```

**邊界檢查**：
```
arr.at(i)  // 檢查 i < N，超出範圍拋出 std::out_of_range
           // 時間複雜度：O(1)
```

**大小查詢**：
```
arr.size()  // 返回模板參數 N（編譯期常數）
            // 時間複雜度：O(1)，通常被編譯器優化為常數
```

## 基本使用

### 頭文件和宣告

```cpp
#include <array>
#include <iostream>

std::array<int, 5> arr1;               // 預設初始化（值未定義）
std::array<int, 5> arr2 = {};          // 零初始化
std::array<int, 5> arr3 = {1, 2, 3};   // 部分初始化（剩餘為 0）
std::array<int, 5> arr4{1, 2, 3, 4, 5};// 完整初始化（C++11）
```

### 構造方法

```cpp
#include <array>
#include <iostream>

int main() {
    // 預設構造（元素未初始化）
    std::array<int, 5> arr1;

    // 零初始化
    std::array<int, 5> arr2 = {};
    // arr2: {0, 0, 0, 0, 0}

    // 列表初始化
    std::array<int, 5> arr3 = {1, 2, 3, 4, 5};
    // arr3: {1, 2, 3, 4, 5}

    // 部分初始化（剩餘為 0）
    std::array<int, 5> arr4 = {1, 2};
    // arr4: {1, 2, 0, 0, 0}

    // C++11 統一初始化（可省略 =）
    std::array<int, 5> arr5{10, 20, 30, 40, 50};

    // 拷貝構造
    std::array<int, 5> arr6 = arr5;

    // 從 C 陣列構造（需要手動複製）
    int cArr[] = {1, 2, 3, 4, 5};
    std::array<int, 5> arr7;
    std::copy(std::begin(cArr), std::end(cArr), arr7.begin());

    return 0;
}
```

### 常用成員函式

| 函式 | 功能 | 時間複雜度 |
|------|------|-----------|
| `operator[]` | 訪問元素（無邊界檢查） | O(1) |
| `at(index)` | 訪問元素（有邊界檢查） | O(1) |
| `front()` | 返回第一個元素 | O(1) |
| `back()` | 返回最後一個元素 | O(1) |
| `data()` | 返回指向底層陣列的指標 | O(1) |
| `size()` | 返回元素個數（編譯期常數） | O(1) |
| `empty()` | 判斷是否為空（編譯期常數） | O(1) |
| `fill(val)` | 所有元素填充為 val | O(n) |
| `swap(other)` | 交換兩個陣列 | O(n) |
| `begin()/end()` | 返回迭代器 | O(1) |

**注意**：`std::array` **沒有** `push_back()`、`insert()`、`erase()` 等動態操作，因為大小固定。

## 性能分析

### 時間複雜度表格

| 操作 | array | vector | C 陣列 |
|------|-------|--------|--------|
| 隨機訪問 | **O(1)** | O(1) | O(1) |
| 頭部插入 | **不支援** | O(n) | 不支援 |
| 尾部插入 | **不支援** | O(1) 攤銷 | 不支援 |
| 查找 | O(n) | O(n) | O(n) |
| 排序 | O(n log n) | O(n log n) | O(n log n) |

### 空間複雜度分析

**記憶體開銷**：
```cpp
sizeof(std::array<int, 5>)  // = 5 * sizeof(int) = 20 bytes
sizeof(int[5])              // = 5 * sizeof(int) = 20 bytes

// 零開銷！std::array 沒有任何額外開銷
```

**與 vector 比較**：
```cpp
std::array<int, 5> arr;     // 20 bytes（棧）
std::vector<int> vec(5);    // 20 bytes（堆）+ 24 bytes（vector 物件）

// array 更輕量，且在棧上分配
```

### 何時性能最優/最差

**性能最優**：
- 大小固定且已知
- 需要棧上分配（避免堆記憶體開銷）
- 需要與 C 介面互動
- 性能關鍵路徑（無動態分配）
- 小型陣列（棧空間充足）

**性能最差**：
- 需要動態改變大小（根本不支援）
- 陣列很大（棧溢出風險）
- 需要頻繁拷貝大型陣列（因為 array 是值語義）

## 常見操作示例

### 訪問元素

```cpp
#include <array>
#include <iostream>

int main() {
    std::array<int, 5> arr = {10, 20, 30, 40, 50};

    // 方法1：下標訪問（無邊界檢查，最快）
    std::cout << "arr[0] = " << arr[0] << std::endl;
    std::cout << "arr[4] = " << arr[4] << std::endl;

    // 方法2：at() 訪問（有邊界檢查，更安全）
    std::cout << "arr.at(0) = " << arr.at(0) << std::endl;

    try {
        std::cout << arr.at(10) << std::endl;  // 超出範圍
    } catch (const std::out_of_range& e) {
        std::cout << "錯誤: " << e.what() << std::endl;
    }

    // 方法3：front() 和 back()
    std::cout << "第一個元素: " << arr.front() << std::endl;
    std::cout << "最後一個元素: " << arr.back() << std::endl;

    // 方法4：data() 取得原生指標
    int* ptr = arr.data();
    std::cout << "ptr[0] = " << ptr[0] << std::endl;

    return 0;
}
```

**輸出**：
```
arr[0] = 10
arr[4] = 50
arr.at(0) = 10
錯誤: array::at: __n (which is 10) >= _Nm (which is 5)
第一個元素: 10
最後一個元素: 50
ptr[0] = 10
```

### 修改元素

```cpp
#include <array>
#include <iostream>

int main() {
    std::array<int, 5> arr = {1, 2, 3, 4, 5};

    // 修改單個元素
    arr[0] = 100;
    arr.at(1) = 200;

    // 使用 fill() 填充所有元素
    std::array<int, 5> arr2;
    arr2.fill(99);

    // 輸出
    std::cout << "arr: ";
    for (int val : arr) {
        std::cout << val << " ";
    }
    std::cout << std::endl;

    std::cout << "arr2: ";
    for (int val : arr2) {
        std::cout << val << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

**輸出**：
```
arr: 100 200 3 4 5
arr2: 99 99 99 99 99
```

### 遍歷陣列

```cpp
#include <array>
#include <iostream>

int main() {
    std::array<int, 5> arr = {1, 2, 3, 4, 5};

    // 方法1：範圍 for 迴圈
    std::cout << "方法1: ";
    for (int val : arr) {
        std::cout << val << " ";
    }
    std::cout << std::endl;

    // 方法2：傳統 for 迴圈 + 下標
    std::cout << "方法2: ";
    for (size_t i = 0; i < arr.size(); ++i) {
        std::cout << arr[i] << " ";
    }
    std::cout << std::endl;

    // 方法3：迭代器
    std::cout << "方法3: ";
    for (auto it = arr.begin(); it != arr.end(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    // 方法4：反向遍歷
    std::cout << "方法4（反向）: ";
    for (auto it = arr.rbegin(); it != arr.rend(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

**輸出**：
```
方法1: 1 2 3 4 5
方法2: 1 2 3 4 5
方法3: 1 2 3 4 5
方法4（反向）: 5 4 3 2 1
```

### 排序與查找

```cpp
#include <array>
#include <algorithm>
#include <iostream>

int main() {
    std::array<int, 7> arr = {5, 2, 8, 1, 9, 3, 7};

    // 排序
    std::sort(arr.begin(), arr.end());
    std::cout << "排序後: ";
    for (int val : arr) {
        std::cout << val << " ";
    }
    std::cout << std::endl;

    // 二分查找（需要先排序）
    bool found = std::binary_search(arr.begin(), arr.end(), 7);
    std::cout << "是否找到 7: " << (found ? "是" : "否") << std::endl;

    // 查找第一個出現位置
    auto it = std::find(arr.begin(), arr.end(), 8);
    if (it != arr.end()) {
        std::cout << "找到 8，位於索引: "
                  << std::distance(arr.begin(), it) << std::endl;
    }

    // 反轉
    std::reverse(arr.begin(), arr.end());
    std::cout << "反轉後: ";
    for (int val : arr) {
        std::cout << val << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

**輸出**：
```
排序後: 1 2 3 5 7 8 9
是否找到 7: 是
找到 8，位於索引: 5
反轉後: 9 8 7 5 3 2 1
```

### 多維陣列

```cpp
#include <array>
#include <iostream>

int main() {
    // 2D 陣列：3x4 矩陣
    std::array<std::array<int, 4>, 3> matrix = {{
        {1, 2, 3, 4},
        {5, 6, 7, 8},
        {9, 10, 11, 12}
    }};

    // 訪問元素
    std::cout << "matrix[1][2] = " << matrix[1][2] << std::endl;

    // 遍歷 2D 陣列
    std::cout << "矩陣內容:" << std::endl;
    for (size_t i = 0; i < matrix.size(); ++i) {
        for (size_t j = 0; j < matrix[i].size(); ++j) {
            std::cout << matrix[i][j] << " ";
        }
        std::cout << std::endl;
    }

    // 使用範圍 for
    std::cout << "使用範圍 for:" << std::endl;
    for (const auto& row : matrix) {
        for (int val : row) {
            std::cout << val << " ";
        }
        std::cout << std::endl;
    }

    return 0;
}
```

**輸出**：
```
matrix[1][2] = 7
矩陣內容:
1 2 3 4
5 6 7 8
9 10 11 12
使用範圍 for:
1 2 3 4
5 6 7 8
9 10 11 12
```

### 交換陣列

```cpp
#include <array>
#include <iostream>

void printArray(const std::array<int, 5>& arr, const std::string& name) {
    std::cout << name << ": ";
    for (int val : arr) {
        std::cout << val << " ";
    }
    std::cout << std::endl;
}

int main() {
    std::array<int, 5> arr1 = {1, 2, 3, 4, 5};
    std::array<int, 5> arr2 = {10, 20, 30, 40, 50};

    std::cout << "交換前:" << std::endl;
    printArray(arr1, "arr1");
    printArray(arr2, "arr2");

    // 使用 swap() 成員函式
    arr1.swap(arr2);

    std::cout << "\n交換後:" << std::endl;
    printArray(arr1, "arr1");
    printArray(arr2, "arr2");

    // 也可以使用 std::swap
    std::swap(arr1, arr2);

    std::cout << "\n再次交換後:" << std::endl;
    printArray(arr1, "arr1");
    printArray(arr2, "arr2");

    return 0;
}
```

**輸出**：
```
交換前:
arr1: 1 2 3 4 5
arr2: 10 20 30 40 50

交換後:
arr1: 10 20 30 40 50
arr2: 1 2 3 4 5

再次交換後:
arr1: 1 2 3 4 5
arr2: 10 20 30 40 50
```

## 迭代器支持

### 支援的迭代器類型

`std::array` 提供**隨機訪問迭代器**（random access iterator）：

```cpp
#include <array>
#include <iostream>

int main() {
    std::array<int, 5> arr = {1, 2, 3, 4, 5};

    auto it = arr.begin();

    // 隨機訪問迭代器支援所有操作
    ++it;           // 前進
    --it;           // 後退
    it += 3;        // 跳躍
    it -= 2;        // 反向跳躍
    *it;            // 解參考
    it[2];          // 下標訪問

    std::cout << "it[2] = " << it[2] << std::endl;

    // 迭代器運算
    auto it2 = arr.begin() + 3;
    int distance = it2 - arr.begin();
    std::cout << "距離: " << distance << std::endl;

    return 0;
}
```

### 迭代器失效問題

**array 的特點**：由於大小固定，**迭代器幾乎永不失效**。

```cpp
#include <array>
#include <iostream>

int main() {
    std::array<int, 5> arr = {1, 2, 3, 4, 5};

    auto it1 = arr.begin();
    auto it2 = arr.begin() + 2;

    // 修改元素不會使迭代器失效
    arr[0] = 100;
    arr[2] = 300;

    std::cout << "*it1 = " << *it1 << std::endl;  // 100（已修改）
    std::cout << "*it2 = " << *it2 << std::endl;  // 300（已修改）

    // fill() 不會使迭代器失效
    arr.fill(99);
    std::cout << "*it1 = " << *it1 << std::endl;  // 99

    // swap() 會使迭代器失效！
    std::array<int, 5> arr2 = {10, 20, 30, 40, 50};
    arr.swap(arr2);
    // 現在 it1 指向 arr2 的元素！

    return 0;
}
```

**迭代器失效規則總結**：

| 操作 | 失效範圍 |
|------|----------|
| `operator[]`/`at()` | **永不失效** |
| `fill()` | **永不失效** |
| `swap()` | **所有迭代器失效**（指向交換後的另一個容器） |

## 常見陷阱與注意事項

### 1. 大小必須是編譯期常數

```cpp
// ❌ 錯誤：執行期變數
int n = 5;
// std::array<int, n> arr;  // 編譯錯誤！

// ✅ 正確：編譯期常數
const int N = 5;
std::array<int, N> arr1;

constexpr int M = 10;
std::array<int, M> arr2;

std::array<int, 5> arr3;  // 字面常數
```

### 2. 大型陣列可能導致棧溢出

```cpp
// ❌ 危險：可能棧溢出（1MB 陣列）
std::array<int, 250000> hugeArray;  // 局部變數在棧上

// ✅ 解決方案1：使用 static（放在靜態儲存區）
static std::array<int, 250000> hugeArray;

// ✅ 解決方案2：使用 vector（堆分配）
std::vector<int> vec(250000);

// ✅ 解決方案3：放在全域或類別成員
```

### 3. 未初始化的陣列內容未定義

```cpp
// ❌ 錯誤：元素值未定義
std::array<int, 5> arr1;
std::cout << arr1[0] << std::endl;  // 未定義行為

// ✅ 正確：零初始化
std::array<int, 5> arr2 = {};
std::cout << arr2[0] << std::endl;  // 保證是 0

// ✅ 正確：使用 fill()
std::array<int, 5> arr3;
arr3.fill(0);
```

### 4. 傳遞陣列時是值拷貝

```cpp
void processArray(std::array<int, 1000> arr) {  // ⚠️ 拷貝整個陣列！
    // 處理...
}

// ✅ 推薦：使用 const 參考
void processArray(const std::array<int, 1000>& arr) {
    // 處理...
}

// ✅ 如果需要修改：使用參考
void modifyArray(std::array<int, 1000>& arr) {
    // 修改...
}
```

### 5. 不同大小的 array 是不同型別

```cpp
std::array<int, 5> arr1 = {1, 2, 3, 4, 5};
std::array<int, 6> arr2 = {1, 2, 3, 4, 5, 6};

// ❌ 編譯錯誤：不同型別
// arr1 = arr2;

// ❌ 編譯錯誤：函式無法接受不同大小的 array
void func(std::array<int, 5> arr);
// func(arr2);  // 錯誤

// ✅ 解決方案：使用模板
template<size_t N>
void func(const std::array<int, N>& arr) {
    // 可以接受任意大小的 array
}
```

### 6. 空陣列是合法的

```cpp
// ✅ 合法：大小為 0 的陣列
std::array<int, 0> emptyArray;

std::cout << "size: " << emptyArray.size() << std::endl;     // 0
std::cout << "empty: " << emptyArray.empty() << std::endl;   // true

// ⚠️ 不能訪問元素（未定義行為）
// emptyArray[0];    // 未定義
// emptyArray.front();  // 未定義
```

## 實際應用場景

### 場景 1：固定大小的緩衝區

網路程式設計中常用固定大小的緩衝區。

```cpp
#include <array>
#include <iostream>
#include <cstring>

class NetworkBuffer {
private:
    static constexpr size_t BUFFER_SIZE = 1024;
    std::array<char, BUFFER_SIZE> buffer;
    size_t dataLength;

public:
    NetworkBuffer() : dataLength(0) {
        buffer.fill(0);
    }

    bool write(const char* data, size_t len) {
        if (dataLength + len > BUFFER_SIZE) {
            std::cerr << "緩衝區已滿" << std::endl;
            return false;
        }

        std::memcpy(buffer.data() + dataLength, data, len);
        dataLength += len;
        return true;
    }

    const char* data() const {
        return buffer.data();
    }

    size_t size() const {
        return dataLength;
    }

    void clear() {
        buffer.fill(0);
        dataLength = 0;
    }

    void print() const {
        std::cout << "緩衝區內容 (" << dataLength << " bytes): ";
        for (size_t i = 0; i < dataLength; ++i) {
            std::cout << buffer[i];
        }
        std::cout << std::endl;
    }
};

int main() {
    NetworkBuffer buf;

    buf.write("Hello, ", 7);
    buf.write("World!", 6);
    buf.print();

    buf.clear();
    buf.write("New data", 8);
    buf.print();

    return 0;
}
```

**輸出**：
```
緩衝區內容 (13 bytes): Hello, World!
緩衝區內容 (8 bytes): New data
```

### 場景 2：查找表（Lookup Table）

編譯期已知的對映關係可以使用 array 實作查找表。

```cpp
#include <array>
#include <iostream>
#include <string>

class MonthInfo {
private:
    static constexpr size_t NUM_MONTHS = 12;

    // 月份名稱查找表
    static constexpr std::array<const char*, NUM_MONTHS> monthNames = {
        "January", "February", "March", "April", "May", "June",
        "July", "August", "September", "October", "November", "December"
    };

    // 月份天數查找表（平年）
    static constexpr std::array<int, NUM_MONTHS> daysInMonth = {
        31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31
    };

public:
    static const char* getMonthName(int month) {
        if (month < 1 || month > 12) {
            return "Invalid";
        }
        return monthNames[month - 1];
    }

    static int getDaysInMonth(int month, bool isLeapYear = false) {
        if (month < 1 || month > 12) {
            return -1;
        }

        int days = daysInMonth[month - 1];
        if (month == 2 && isLeapYear) {
            days = 29;
        }
        return days;
    }

    static void printCalendarInfo() {
        std::cout << "月份資訊（平年）:" << std::endl;
        for (int i = 1; i <= 12; ++i) {
            std::cout << i << ". " << getMonthName(i)
                      << " - " << getDaysInMonth(i) << " 天" << std::endl;
        }
    }
};

// 初始化靜態成員
constexpr std::array<const char*, 12> MonthInfo::monthNames;
constexpr std::array<int, 12> MonthInfo::daysInMonth;

int main() {
    MonthInfo::printCalendarInfo();

    std::cout << "\n二月天數（閏年）: "
              << MonthInfo::getDaysInMonth(2, true) << std::endl;

    return 0;
}
```

**輸出**：
```
月份資訊（平年）:
1. January - 31 天
2. February - 28 天
3. March - 31 天
4. April - 30 天
5. May - 31 天
6. June - 30 天
7. July - 31 天
8. August - 31 天
9. September - 30 天
10. October - 31 天
11. November - 30 天
12. December - 31 天

二月天數（閏年）: 29
```

### 場景 3：矩陣運算（小型固定矩陣）

3D 圖形學中常用 4x4 矩陣。

```cpp
#include <array>
#include <iostream>
#include <iomanip>

class Matrix4x4 {
private:
    std::array<std::array<float, 4>, 4> data;

public:
    // 構造單位矩陣
    Matrix4x4() {
        for (int i = 0; i < 4; ++i) {
            for (int j = 0; j < 4; ++j) {
                data[i][j] = (i == j) ? 1.0f : 0.0f;
            }
        }
    }

    // 訪問元素
    float& at(int row, int col) {
        return data[row][col];
    }

    float at(int row, int col) const {
        return data[row][col];
    }

    // 矩陣乘法
    Matrix4x4 operator*(const Matrix4x4& other) const {
        Matrix4x4 result;
        for (int i = 0; i < 4; ++i) {
            for (int j = 0; j < 4; ++j) {
                result.data[i][j] = 0;
                for (int k = 0; k < 4; ++k) {
                    result.data[i][j] += data[i][k] * other.data[k][j];
                }
            }
        }
        return result;
    }

    // 縮放矩陣
    static Matrix4x4 scale(float sx, float sy, float sz) {
        Matrix4x4 mat;
        mat.data[0][0] = sx;
        mat.data[1][1] = sy;
        mat.data[2][2] = sz;
        return mat;
    }

    // 平移矩陣
    static Matrix4x4 translate(float tx, float ty, float tz) {
        Matrix4x4 mat;
        mat.data[0][3] = tx;
        mat.data[1][3] = ty;
        mat.data[2][3] = tz;
        return mat;
    }

    // 輸出
    void print() const {
        for (int i = 0; i < 4; ++i) {
            for (int j = 0; j < 4; ++j) {
                std::cout << std::setw(8) << std::setprecision(2)
                          << data[i][j] << " ";
            }
            std::cout << std::endl;
        }
    }
};

int main() {
    std::cout << "單位矩陣:" << std::endl;
    Matrix4x4 identity;
    identity.print();

    std::cout << "\n縮放矩陣 (2, 3, 4):" << std::endl;
    Matrix4x4 scaleMatrix = Matrix4x4::scale(2, 3, 4);
    scaleMatrix.print();

    std::cout << "\n平移矩陣 (5, 6, 7):" << std::endl;
    Matrix4x4 translateMatrix = Matrix4x4::translate(5, 6, 7);
    translateMatrix.print();

    std::cout << "\n複合變換（縮放 × 平移）:" << std::endl;
    Matrix4x4 combined = scaleMatrix * translateMatrix;
    combined.print();

    return 0;
}
```

**輸出**：
```
單位矩陣:
    1.00     0.00     0.00     0.00
    0.00     1.00     0.00     0.00
    0.00     0.00     1.00     0.00
    0.00     0.00     0.00     1.00

縮放矩陣 (2, 3, 4):
    2.00     0.00     0.00     0.00
    0.00     3.00     0.00     0.00
    0.00     0.00     4.00     0.00
    0.00     0.00     0.00     1.00

平移矩陣 (5, 6, 7):
    1.00     0.00     0.00     5.00
    0.00     1.00     0.00     6.00
    0.00     0.00     1.00     7.00
    0.00     0.00     0.00     1.00

複合變換（縮放 × 平移）:
    2.00     0.00     0.00    10.00
    0.00     3.00     0.00    18.00
    0.00     0.00     4.00    28.00
    0.00     0.00     0.00     1.00
```

## 與其他容器的選擇

### 對比表格

| 特性 | array | vector | C 陣列 | deque |
|------|-------|--------|--------|-------|
| 大小固定 | ✅ | ❌ | ✅ | ❌ |
| 編譯期大小 | ✅ | ❌ | ✅ | ❌ |
| 隨機訪問 | ✅ O(1) | ✅ O(1) | ✅ O(1) | ✅ O(1) |
| 棧分配 | ✅ | ❌ | ✅ | ❌ |
| STL 介面 | ✅ | ✅ | ❌ | ✅ |
| 邊界檢查 | ✅ at() | ✅ at() | ❌ | ✅ at() |
| 零開銷 | ✅ | ❌ | ✅ | ❌ |
| 可動態增長 | ❌ | ✅ | ❌ | ✅ |

### 選擇建議

**選擇 array 當**：
- ✅ 元素數量在編譯期已知且固定
- ✅ 需要棧分配（避免堆記憶體開銷）
- ✅ 需要 STL 介面的固定陣列
- ✅ 性能關鍵且大小固定
- ✅ 需要零開銷封裝

**選擇 vector 當**：
- ✅ 元素數量會動態變化
- ✅ 需要在尾部頻繁插入/刪除
- ✅ 陣列很大（避免棧溢出）
- ✅ 大小在執行期決定

**選擇 C 陣列當**：
- ✅ 與 C 介面互動
- ✅ 不需要 STL 介面
- ✅ 極度追求效能（省略 array 的封裝）

**選擇 deque 當**：
- ✅ 需要在頭尾兩端插入/刪除
- ✅ 需要隨機訪問但也需要動態大小

## 實務建議

### DO（應該做的）

#### 1. ✅ 使用 array 而非 C 陣列

```cpp
// ✅ 推薦：使用 std::array
std::array<int, 5> arr = {1, 2, 3, 4, 5};
std::cout << "size: " << arr.size() << std::endl;
std::sort(arr.begin(), arr.end());

// ❌ 避免：使用 C 陣列（無 STL 介面）
int cArr[5] = {1, 2, 3, 4, 5};
// cArr.size();  // 錯誤！
std::sort(std::begin(cArr), std::end(cArr));  // 需要 std::begin/end
```

#### 2. ✅ 使用 at() 進行邊界檢查（除錯模式）

```cpp
std::array<int, 5> arr = {1, 2, 3, 4, 5};

// ✅ 開發階段：使用 at() 捕獲錯誤
try {
    int val = arr.at(10);  // 拋出異常
} catch (const std::out_of_range& e) {
    std::cerr << "索引錯誤: " << e.what() << std::endl;
}

// ✅ 發佈階段：使用 [] 提升效能
// int val = arr[2];  // 無邊界檢查，更快
```

#### 3. ✅ 使用統一初始化避免 most vexing parse

```cpp
// ✅ 推薦：統一初始化
std::array<int, 5> arr1{1, 2, 3, 4, 5};

// ❌ 可能有問題：使用 = {} 在某些情境下有歧義
// std::array<int, 5> arr2 = {1, 2, 3, 4, 5};  // 通常 OK，但不一致
```

#### 4. ✅ 傳遞 array 時使用 const 參考

```cpp
// ✅ 推薦：const 參考（避免拷貝）
void processArray(const std::array<int, 100>& arr) {
    for (int val : arr) {
        // 處理...
    }
}

// ❌ 避免：值傳遞（拷貝整個陣列）
// void processArray(std::array<int, 100> arr) {
//     // 400 bytes 拷貝！
// }
```

#### 5. ✅ 使用 constexpr 定義大小

```cpp
// ✅ 推薦：使用 constexpr
constexpr size_t ARRAY_SIZE = 10;
std::array<int, ARRAY_SIZE> arr;

// ✅ 也可以：直接使用字面常數
std::array<int, 10> arr2;

// ❌ 避免：魔術數字散布各處
// std::array<int, 10> arr1;
// for (int i = 0; i < 10; ++i) { ... }
```

#### 6. ✅ 利用 data() 與 C 介面互動

```cpp
#include <cstring>

std::array<char, 100> buffer;

// ✅ 推薦：使用 data() 取得原生指標
std::memset(buffer.data(), 0, buffer.size());

// C 函式
extern "C" void c_function(char* buf, size_t len);
c_function(buffer.data(), buffer.size());
```

#### 7. ✅ 使用 fill() 初始化所有元素

```cpp
std::array<int, 1000> arr;

// ✅ 推薦：使用 fill() 批次初始化
arr.fill(0);

// ❌ 避免：逐個賦值
// for (size_t i = 0; i < arr.size(); ++i) {
//     arr[i] = 0;
// }
```

#### 8. ✅ 多維陣列使用型別別名簡化

```cpp
// ✅ 推薦：使用 using 定義別名
using Matrix3x3 = std::array<std::array<int, 3>, 3>;

Matrix3x3 mat = {{
    {1, 2, 3},
    {4, 5, 6},
    {7, 8, 9}
}};

// ❌ 避免：每次都寫完整型別
// std::array<std::array<int, 3>, 3> mat = ...
```

### DON'T（不應該做的）

#### 1. ❌ 不要在棧上分配大型 array

```cpp
// ❌ 危險：可能棧溢出（400KB）
void func() {
    std::array<int, 100000> hugeArray;  // 局部變數在棧上
}

// ✅ 解決方案1：使用 static
void func() {
    static std::array<int, 100000> hugeArray;
}

// ✅ 解決方案2：使用 vector
void func() {
    std::vector<int> vec(100000);  // 堆分配
}

// ✅ 解決方案3：改為類別成員
class MyClass {
    std::array<int, 100000> data;  // 跟隨物件分配
};
```

#### 2. ❌ 不要嘗試在執行期決定大小

```cpp
int n;
std::cin >> n;

// ❌ 錯誤：編譯失敗
// std::array<int, n> arr;

// ✅ 正確：使用 vector
std::vector<int> vec(n);
```

#### 3. ❌ 不要忘記零初始化

```cpp
// ❌ 錯誤：元素值未定義
std::array<int, 5> arr1;
std::cout << arr1[0] << std::endl;  // 未定義行為

// ✅ 正確：零初始化
std::array<int, 5> arr2 = {};
std::cout << arr2[0] << std::endl;  // 保證是 0
```

#### 4. ❌ 不要混淆不同大小的 array

```cpp
std::array<int, 5> arr1;
std::array<int, 6> arr2;

// ❌ 錯誤：不同型別
// arr1 = arr2;  // 編譯錯誤

// ✅ 如果需要不同大小，使用模板或 vector
template<size_t N>
void processArray(const std::array<int, N>& arr) {
    // 可以處理任意大小
}
```

#### 5. ❌ 不要對 array 使用動態操作

```cpp
std::array<int, 5> arr = {1, 2, 3, 4, 5};

// ❌ 錯誤：array 沒有這些函式
// arr.push_back(6);   // 編譯錯誤
// arr.insert(arr.begin(), 0);  // 編譯錯誤
// arr.resize(10);     // 編譯錯誤

// ✅ array 的大小是固定的，如需動態操作使用 vector
```

#### 6. ❌ 不要忽略 swap() 的成本

```cpp
std::array<int, 10000> arr1, arr2;

// ⚠️ swap() 會逐個交換元素，O(n) 複雜度
arr1.swap(arr2);  // 交換 10000 個元素

// 💡 提示：vector::swap() 只交換指標，O(1)
std::vector<int> vec1(10000), vec2(10000);
vec1.swap(vec2);  // 僅交換指標，非常快
```

#### 7. ❌ 不要返回 array 的原生指標

```cpp
// ❌ 危險：返回指向局部變數的指標
int* getArrayData() {
    std::array<int, 5> arr = {1, 2, 3, 4, 5};
    return arr.data();  // 危險！arr 在函式結束時銷毀
}

// ✅ 正確：返回整個 array（會複製）
std::array<int, 5> getArray() {
    std::array<int, 5> arr = {1, 2, 3, 4, 5};
    return arr;  // RVO 優化，效率可接受
}

// ✅ 或者：使用參考參數
void fillArray(std::array<int, 5>& arr) {
    arr = {1, 2, 3, 4, 5};
}
```

#### 8. ❌ 不要假設 array 和 C 陣列完全相同

```cpp
std::array<int, 5> arr = {1, 2, 3, 4, 5};

// ⚠️ array 不會隱式轉換為指標
void cFunc(int* ptr);
// cFunc(arr);  // 錯誤！

// ✅ 使用 data() 明確轉換
cFunc(arr.data());

// ⚠️ sizeof 行為不同
int cArr[5];
std::cout << sizeof(cArr) / sizeof(int) << std::endl;  // 5
std::cout << sizeof(arr) / sizeof(int) << std::endl;   // 5（但語意不同）

// ✅ 使用 size() 更清晰
std::cout << arr.size() << std::endl;  // 5
```

## 小結

### 核心概念總結

1. **零開銷封裝**：`std::array` 是對 C 陣列的零開銷封裝，提供 STL 介面
2. **固定大小**：大小必須在編譯期確定，不能動態改變
3. **棧分配**：預設在棧上分配，避免堆記憶體開銷
4. **連續記憶體**：所有元素連續儲存，緩存友好，支援隨機訪問
5. **安全性**：提供 `at()` 邊界檢查，比 C 陣列更安全

### 關鍵記憶點

- ✅ **適合場景**：大小固定、編譯期已知、需要 STL 介面、性能關鍵
- ❌ **不適合場景**：需要動態大小、陣列很大（棧溢出）、執行期決定大小
- 🔧 **核心優勢**：零開銷、棧分配、STL 介面、類型安全
- ⚠️ **主要問題**：大小固定、大型陣列可能棧溢出、不同大小是不同型別

### 一句話總結

**`std::array` 是固定大小的陣列容器，在編譯期確定大小並提供零開銷的 STL 介面封裝，適合大小已知且固定的場景，結合了 C 陣列的效能和 STL 容器的安全性與便利性。**
