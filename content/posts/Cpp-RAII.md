+++
date = '2025-12-16T23:53:39+08:00'
draft = false
title = '[C++] RAII: 資源取得即初始化'
categories = ['C++ Notes']
tags = ['C++', 'RAII']
+++

## 什麼是 RAII？

**RAII（Resource Acquisition Is Initialization）** 是 C++ 中最重要的資源管理技術之一，中文翻譯為「資源取得即初始化」。

**核心概念：**
> 將資源的生命週期綁定到物件的生命週期上：
> - **建構子**：取得資源
> - **解構子**：釋放資源

當物件離開作用域時，解構子會自動被呼叫，確保資源一定會被釋放，即使發生例外也不例外。

### RAII 的精神

```cpp
{
    RAIIObject obj;  // 建構：取得資源

    // 使用資源...

}  // 離開作用域：解構子自動釋放資源
```

## 為什麼需要 RAII？

### 問題：手動管理資源容易出錯

不使用 RAII 時，資源管理容易出現以下問題：

#### 問題 1：忘記釋放資源（資源洩漏）

```cpp
void processFile() {
    FILE* file = fopen("data.txt", "r");

    // 處理檔案...

    // 糟糕！忘記呼叫 fclose(file)
    // → 檔案句柄洩漏
}
```

#### 問題 2：提早 return 導致資源未釋放

```cpp
void processData() {
    int* data = new int[1000];

    if (someCondition) {
        return;  // 糟糕！data 沒有被 delete[]
    }

    // 正常處理...
    delete[] data;  // 這行永遠不會執行
}
```

#### 問題 3：例外導致資源未釋放

```cpp
void dangerousOperation() {
    std::mutex* mtx = new std::mutex();
    mtx->lock();

    riskyFunction();  // 如果拋出例外...
                      // → mtx 永遠不會被 unlock 和 delete
                      // → 造成死鎖和記憶體洩漏

    mtx->unlock();
    delete mtx;
}
```

### RAII 的解決方案

使用 RAII，資源會自動管理，不會有上述問題：

```cpp
void safeOperation() {
    std::unique_ptr<std::mutex> mtx = std::make_unique<std::mutex>();
    std::lock_guard<std::mutex> lock(*mtx);

    riskyFunction();  // 即使拋出例外
                      // → lock 的解構子會自動 unlock
                      // → unique_ptr 的解構子會自動 delete

}  // 離開作用域時自動清理
```

## RAII 的基本實作

### 範例 1：管理動態記憶體

**不使用 RAII（危險）：**

```cpp
void badExample() {
    int* ptr = new int(42);

    // 做一些事情...

    if (errorCondition) {
        return;  // 記憶體洩漏！
    }

    delete ptr;  // 可能永遠執行不到
}
```

**使用 RAII（安全）：**

```cpp
template<typename T>
class SmartPtr {
private:
    T* ptr;

public:
    // 建構子：取得資源
    explicit SmartPtr(T* p = nullptr) : ptr(p) {
        std::cout << "取得資源 (位址: " << ptr << ")\n";
    }

    // 解構子：釋放資源
    ~SmartPtr() {
        std::cout << "釋放資源 (位址: " << ptr << ")\n";
        delete ptr;
    }

    // 禁止複製（簡化版本）
    SmartPtr(const SmartPtr&) = delete;
    SmartPtr& operator=(const SmartPtr&) = delete;

    // 解引用運算子
    T& operator*() { return *ptr; }
    T* operator->() { return ptr; }
};

void goodExample() {
    SmartPtr<int> ptr(new int(42));

    // 做一些事情...

    if (errorCondition) {
        return;  // 安全！解構子會自動釋放記憶體
    }

}  // 離開作用域時自動釋放
```

### 範例 2：管理檔案

**不使用 RAII（危險）：**

```cpp
void readFile() {
    FILE* file = fopen("data.txt", "r");

    if (!file) {
        return;
    }

    // 讀取資料...

    if (parseError) {
        return;  // 檔案沒有關閉！
    }

    fclose(file);  // 可能永遠執行不到
}
```

**使用 RAII（安全）：**

```cpp
class FileHandle {
private:
    FILE* file;
    std::string filename;

public:
    // 建構子：開啟檔案
    explicit FileHandle(const char* fname, const char* mode)
        : filename(fname), file(fopen(fname, mode))
    {
        if (!file) {
            throw std::runtime_error("無法開啟檔案: " + filename);
        }
        std::cout << "開啟檔案: " << filename << "\n";
    }

    // 解構子：關閉檔案
    ~FileHandle() {
        if (file) {
            std::cout << "關閉檔案: " << filename << "\n";
            fclose(file);
        }
    }

    // 禁止複製
    FileHandle(const FileHandle&) = delete;
    FileHandle& operator=(const FileHandle&) = delete;

    // 取得檔案指標
    FILE* get() { return file; }

    // 檢查檔案是否有效
    operator bool() const { return file != nullptr; }
};

void safeReadFile() {
    FileHandle file("data.txt", "r");

    // 讀取資料...

    if (parseError) {
        return;  // 安全！解構子會自動關閉檔案
    }

}  // 離開作用域時自動關閉檔案
```

### 範例 3：管理互斥鎖

**不使用 RAII（危險）：**

```cpp
std::mutex mtx;

void criticalSection() {
    mtx.lock();

    // 執行關鍵區段...

    if (errorOccurred) {
        return;  // 死鎖！mtx 沒有被 unlock
    }

    mtx.unlock();  // 可能永遠執行不到
}
```

**使用 RAII（安全）：**

```cpp
class LockGuard {
private:
    std::mutex& mtx;

public:
    // 建構子：上鎖
    explicit LockGuard(std::mutex& m) : mtx(m) {
        mtx.lock();
        std::cout << "已上鎖\n";
    }

    // 解構子：解鎖
    ~LockGuard() {
        std::cout << "已解鎖\n";
        mtx.unlock();
    }

    // 禁止複製和移動
    LockGuard(const LockGuard&) = delete;
    LockGuard& operator=(const LockGuard&) = delete;
};

std::mutex mtx;

void safeCriticalSection() {
    LockGuard lock(mtx);

    // 執行關鍵區段...

    if (errorOccurred) {
        return;  // 安全！解構子會自動 unlock
    }

}  // 離開作用域時自動解鎖
```

## RAII 的例外安全保證

RAII 最強大的特性之一是**自動的例外安全**。

### 例外安全範例

```cpp
#include <iostream>
#include <stdexcept>

class Resource {
private:
    int id;

public:
    explicit Resource(int i) : id(i) {
        std::cout << "取得資源 " << id << "\n";
    }

    ~Resource() {
        std::cout << "釋放資源 " << id << "\n";
    }
};

void mightThrow() {
    throw std::runtime_error("發生錯誤！");
}

void demonstrateExceptionSafety() {
    try {
        Resource r1(1);  // 取得資源 1
        Resource r2(2);  // 取得資源 2

        std::cout << "即將拋出例外...\n";
        mightThrow();  // 拋出例外

        std::cout << "這行永遠不會執行\n";

    } catch (const std::exception& e) {
        std::cout << "捕獲例外: " << e.what() << "\n";
    }

    std::cout << "函式結束\n";
}

int main() {
    demonstrateExceptionSafety();
    return 0;
}
```

### 執行結果

```
取得資源 1
取得資源 2
即將拋出例外...
釋放資源 2    ← 自動釋放！
釋放資源 1    ← 自動釋放！
捕獲例外: 發生錯誤！
函式結束
```

**關鍵點：**
- 即使發生例外，所有已建構的物件都會被正確解構
- 解構順序與建構順序相反（Stack Unwinding）
- 不需要手動清理，完全自動化

## 標準函式庫中的 RAII

C++ 標準函式庫大量使用 RAII 原則：

### 1. 智慧指標（Smart Pointers）

```cpp
#include <memory>

void smartPointerExample() {
    // unique_ptr: 獨占所有權
    {
        std::unique_ptr<int> ptr1 = std::make_unique<int>(42);
        std::cout << *ptr1 << "\n";
    }  // 自動 delete

    // shared_ptr: 共享所有權
    {
        std::shared_ptr<int> ptr2 = std::make_shared<int>(100);
        {
            std::shared_ptr<int> ptr3 = ptr2;  // 共享所有權
            std::cout << "引用計數: " << ptr2.use_count() << "\n";  // 2
        }  // ptr3 離開作用域
        std::cout << "引用計數: " << ptr2.use_count() << "\n";  // 1
    }  // ptr2 離開作用域，記憶體被釋放
}
```

### 2. 容器（Containers）

```cpp
void containerExample() {
    {
        std::vector<int> vec = {1, 2, 3, 4, 5};
        // vector 自動管理記憶體
    }  // 離開作用域時自動釋放所有元素和記憶體

    {
        std::string str = "Hello, RAII!";
        // string 自動管理字元陣列
    }  // 離開作用域時自動釋放記憶體
}
```

### 3. 鎖守衛（Lock Guards）

```cpp
#include <mutex>

std::mutex mtx;
int shared_data = 0;

void threadSafeIncrement() {
    std::lock_guard<std::mutex> lock(mtx);  // 自動上鎖

    ++shared_data;

    // 即使這裡拋出例外，lock 也會自動解鎖

}  // 離開作用域時自動解鎖
```

### 4. 檔案串流（File Streams）

```cpp
#include <fstream>

void fileStreamExample() {
    {
        std::ifstream file("data.txt");

        if (file) {
            std::string line;
            std::getline(file, line);
        }

    }  // 離開作用域時自動關閉檔案
}
```

### 5. unique_lock（更靈活的鎖）

```cpp
void uniqueLockExample() {
    std::mutex mtx;

    {
        std::unique_lock<std::mutex> lock(mtx);  // 上鎖

        // 可以手動解鎖
        lock.unlock();

        // 做一些不需要鎖的事情...

        // 再次上鎖
        lock.lock();

    }  // 如果還在鎖定狀態，會自動解鎖
}
```

## 自定義 RAII 類別的完整範例

### 範例：資料庫連線管理

```cpp
#include <iostream>
#include <string>
#include <stdexcept>

// 模擬資料庫 API
namespace DB {
    struct Connection {
        std::string connectionString;
        bool connected = false;
    };

    Connection* connect(const std::string& connStr) {
        std::cout << "連線到資料庫: " << connStr << "\n";
        Connection* conn = new Connection();
        conn->connectionString = connStr;
        conn->connected = true;
        return conn;
    }

    void disconnect(Connection* conn) {
        if (conn && conn->connected) {
            std::cout << "關閉連線: " << conn->connectionString << "\n";
            conn->connected = false;
            delete conn;
        }
    }

    void executeQuery(Connection* conn, const std::string& query) {
        if (!conn || !conn->connected) {
            throw std::runtime_error("連線無效");
        }
        std::cout << "執行查詢: " << query << "\n";
    }
}

// RAII 包裝器
class DatabaseConnection {
private:
    DB::Connection* conn;
    std::string connectionString;

public:
    // 建構子：建立連線
    explicit DatabaseConnection(const std::string& connStr)
        : connectionString(connStr), conn(nullptr)
    {
        conn = DB::connect(connStr);
        if (!conn) {
            throw std::runtime_error("無法連線到資料庫");
        }
    }

    // 解構子：關閉連線
    ~DatabaseConnection() {
        if (conn) {
            DB::disconnect(conn);
        }
    }

    // 禁止複製
    DatabaseConnection(const DatabaseConnection&) = delete;
    DatabaseConnection& operator=(const DatabaseConnection&) = delete;

    // 允許移動（C++11）
    DatabaseConnection(DatabaseConnection&& other) noexcept
        : conn(other.conn), connectionString(std::move(other.connectionString))
    {
        other.conn = nullptr;
    }

    DatabaseConnection& operator=(DatabaseConnection&& other) noexcept {
        if (this != &other) {
            // 釋放現有資源
            if (conn) {
                DB::disconnect(conn);
            }

            // 移動資源
            conn = other.conn;
            connectionString = std::move(other.connectionString);

            // 清空來源
            other.conn = nullptr;
        }
        return *this;
    }

    // 執行查詢
    void execute(const std::string& query) {
        DB::executeQuery(conn, query);
    }

    // 檢查連線是否有效
    bool isConnected() const {
        return conn && conn->connected;
    }
};

void databaseExample() {
    try {
        DatabaseConnection db("localhost:5432/mydb");

        db.execute("SELECT * FROM users");
        db.execute("INSERT INTO logs VALUES (...)");

        // 即使這裡拋出例外，連線也會自動關閉

    } catch (const std::exception& e) {
        std::cout << "錯誤: " << e.what() << "\n";
    }

    std::cout << "資料庫操作完成\n";
}

int main() {
    databaseExample();
    return 0;
}
```

### 執行結果

```
連線到資料庫: localhost:5432/mydb
執行查詢: SELECT * FROM users
執行查詢: INSERT INTO logs VALUES (...)
關閉連線: localhost:5432/mydb
資料庫操作完成
```

## RAII 的設計原則

### 1. 一個類別管理一種資源

```cpp
// 好：每個類別專注管理一種資源
class FileHandle {
    FILE* file;
public:
    explicit FileHandle(const char* name);
    ~FileHandle();
};

class MemoryBlock {
    void* memory;
public:
    explicit MemoryBlock(size_t size);
    ~MemoryBlock();
};

// 不好：一個類別管理多種資源（容易出錯）
class BadResource {
    FILE* file;
    void* memory;
    int socket;
public:
    BadResource();
    ~BadResource();  // 需要正確釋放所有資源，容易遺漏
};
```

### 2. 建構成功即資源取得

```cpp
class GoodResource {
public:
    GoodResource() {
        // 取得資源
        // 如果失敗，拋出例外
        if (!acquireResource()) {
            throw std::runtime_error("無法取得資源");
        }
    }

    // 解構子不應該失敗
    ~GoodResource() noexcept {
        releaseResource();
    }
};
```

### 3. 解構子不應該拋出例外

```cpp
class SafeResource {
public:
    ~SafeResource() noexcept {  // 標記為 noexcept
        try {
            // 釋放資源
            cleanup();
        } catch (...) {
            // 記錄錯誤但不拋出
            // 解構子拋出例外會導致程式終止
        }
    }
};
```

### 4. 考慮是否允許複製和移動

```cpp
// 選項 1：不可複製、不可移動（最簡單）
class NonCopyableResource {
public:
    NonCopyableResource(const NonCopyableResource&) = delete;
    NonCopyableResource& operator=(const NonCopyableResource&) = delete;
};

// 選項 2：可移動、不可複製（像 unique_ptr）
class MovableResource {
public:
    MovableResource(const MovableResource&) = delete;
    MovableResource& operator=(const MovableResource&) = delete;

    MovableResource(MovableResource&&) noexcept;
    MovableResource& operator=(MovableResource&&) noexcept;
};

// 選項 3：可複製、可移動（需要實作深複製）
class CopyableResource {
public:
    CopyableResource(const CopyableResource&);  // 深複製
    CopyableResource& operator=(const CopyableResource&);

    CopyableResource(CopyableResource&&) noexcept;
    CopyableResource& operator=(CopyableResource&&) noexcept;
};
```

## RAII vs 其他資源管理方式

### 1. RAII vs 手動管理

```cpp
// 手動管理（容易出錯）
void manualManagement() {
    Resource* res = acquireResource();

    try {
        useResource(res);
    } catch (...) {
        releaseResource(res);  // 必須記得釋放
        throw;
    }

    releaseResource(res);  // 必須記得釋放
}

// RAII（自動管理）
void raii() {
    RAIIResource res;

    useResource(res);  // 自動清理，即使拋出例外
}
```

### 2. RAII vs Garbage Collection

| 特性 | RAII | Garbage Collection |
|------|------|-------------------|
| 釋放時機 | 確定性（離開作用域） | 不確定性（GC 執行時） |
| 效能開銷 | 低（編譯期決定） | 高（執行期追蹤） |
| 適用資源 | 所有資源 | 主要是記憶體 |
| 例外安全 | 自動保證 | 無法管理非記憶體資源 |

```cpp
// C++：RAII 確保立即釋放
{
    std::ifstream file("large.dat");  // 開啟檔案
    // 使用檔案...
}  // 立即關閉檔案

// Java/C#：依賴 GC（檔案可能長時間不關閉）
{
    FileInputStream file = new FileInputStream("large.dat");
    // 使用檔案...
}  // file 何時被 finalize？不確定！
```

## 實務建議

### DO（應該做的）

1. **優先使用標準函式庫的 RAII 類別**
   ```cpp
   std::unique_ptr<int> ptr = std::make_unique<int>(42);
   std::vector<int> vec = {1, 2, 3};
   std::lock_guard<std::mutex> lock(mtx);
   ```

2. **自定義 RAII 類別時遵循單一職責**
   ```cpp
   class FileHandle {
       FILE* file;  // 只管理檔案
   };
   ```

3. **建構失敗時拋出例外**
   ```cpp
   Resource() {
       if (!init()) {
           throw std::runtime_error("初始化失敗");
       }
   }
   ```

4. **解構子標記 noexcept**
   ```cpp
   ~Resource() noexcept {
       cleanup();
   }
   ```

5. **明確定義或刪除複製/移動語意**
   ```cpp
   Resource(const Resource&) = delete;
   Resource(Resource&&) noexcept = default;
   ```

### DON'T（不應該做的）

1. **不要在解構子中拋出例外**
   ```cpp
   ~BadResource() {
       if (!cleanup()) {
           throw std::runtime_error("清理失敗");  // 危險！
       }
   }
   ```

2. **不要手動管理能用 RAII 的資源**
   ```cpp
   // 不好
   int* ptr = new int(42);
   // ... 使用 ptr
   delete ptr;

   // 好
   std::unique_ptr<int> ptr = std::make_unique<int>(42);
   ```

3. **不要將資源取得和初始化分離**
   ```cpp
   // 不好
   class BadResource {
       FILE* file = nullptr;
   public:
       BadResource() {}  // 沒有取得資源
       void init() { file = fopen(...); }  // 分離的初始化
   };

   // 好
   class GoodResource {
       FILE* file;
   public:
       GoodResource() : file(fopen(...)) {}  // 建構即取得
   };
   ```

4. **不要在建構子中使用裸指標管理多個資源**
   ```cpp
   // 危險
   class BadClass {
       int* ptr1;
       int* ptr2;
   public:
       BadClass() {
           ptr1 = new int(1);  // 成功
           ptr2 = new int(2);  // 如果這裡拋出例外，ptr1 會洩漏
       }
   };

   // 安全
   class GoodClass {
       std::unique_ptr<int> ptr1;
       std::unique_ptr<int> ptr2;
   public:
       GoodClass()
           : ptr1(std::make_unique<int>(1)),
             ptr2(std::make_unique<int>(2))
       {}  // 例外安全
   };
   ```

## 小結

**RAII 的核心概念：**

1. **定義**：將資源生命週期綁定到物件生命週期
   - 建構子取得資源
   - 解構子釋放資源

2. **優點**：
   - **自動化**：不需要手動釋放資源
   - **例外安全**：即使拋出例外也能正確清理
   - **確定性**：資源在離開作用域時立即釋放
   - **簡潔**：程式碼更簡潔易讀

3. **標準函式庫支援**：
   - 智慧指標（`unique_ptr`、`shared_ptr`）
   - 容器（`vector`、`string`）
   - 鎖守衛（`lock_guard`、`unique_lock`）
   - 檔案串流（`fstream`）

4. **設計原則**：
   - 一個類別管理一種資源
   - 建構成功即資源取得
   - 解構子不拋出例外
   - 明確定義複製/移動語意

5. **最佳實踐**：
   - 優先使用標準函式庫的 RAII 類別
   - 自定義 RAII 類別時遵循 Rule of Three/Five/Zero
   - 避免手動管理資源

**記住：RAII 是 C++ 中最重要的慣用法之一，它讓資源管理變得自動、安全且優雅。幾乎所有的資源管理都應該使用 RAII 原則！**
