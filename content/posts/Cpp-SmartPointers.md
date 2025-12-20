+++
date = '2025-12-17T00:18:56+08:00'
draft = false
title = '[C++] 智慧指標：從 auto_ptr 到現代 C++'
categories = ['C++ Notes']
tags = ['C++', 'Pointers']
+++

# 目錄

- [目錄](#目錄)
  - [什麼是智慧指標？](#什麼是智慧指標)
    - [為什麼需要智慧指標？](#為什麼需要智慧指標)
  - [智慧指標的演進歷史](#智慧指標的演進歷史)
    - [1. 史前時代：手動管理（C++98 以前）](#1-史前時代手動管理c98-以前)
    - [2. C++98：auto\_ptr（已廢棄）](#2-c98auto_ptr已廢棄)
    - [3. C++11：現代智慧指標](#3-c11現代智慧指標)
  - [unique\_ptr：獨占所有權](#unique_ptr獨占所有權)
    - [基本使用](#基本使用)
    - [unique\_ptr 不可複製，但可移動](#unique_ptr-不可複製但可移動)
    - [管理陣列](#管理陣列)
    - [自定義刪除器](#自定義刪除器)
    - [完整範例：管理類別資源](#完整範例管理類別資源)
  - [shared\_ptr：共享所有權](#shared_ptr共享所有權)
    - [基本使用](#基本使用-1)
    - [shared\_ptr 可以複製](#shared_ptr-可以複製)
    - [完整範例：共享資源](#完整範例共享資源)
    - [shared\_ptr 的開銷](#shared_ptr-的開銷)
    - [make\_shared 的優點](#make_shared-的優點)
  - [weak\_ptr：打破循環參考](#weak_ptr打破循環參考)
    - [循環參考問題](#循環參考問題)
    - [使用 weak\_ptr 解決](#使用-weak_ptr-解決)
    - [weak\_ptr 的使用](#weak_ptr-的使用)
    - [實用範例：觀察者模式](#實用範例觀察者模式)
  - [智慧指標的選擇](#智慧指標的選擇)
    - [決策樹](#決策樹)
    - [使用場景](#使用場景)
    - [效能考量](#效能考量)
  - [實用範例](#實用範例)
    - [範例 1：工廠模式](#範例-1工廠模式)
    - [範例 2：資源池](#範例-2資源池)
    - [範例 3：PIMPL 慣用法](#範例-3pimpl-慣用法)
  - [最佳實踐](#最佳實踐)
    - [DO（應該做的）](#do應該做的)
    - [DON'T（不應該做的）](#dont不應該做的)
  - [常見陷阱](#常見陷阱)
    - [陷阱 1：忘記定義解構子（PIMPL）](#陷阱-1忘記定義解構子pimpl)
    - [陷阱 2：循環參考](#陷阱-2循環參考)
    - [陷阱 3：使用 get() 後建立新的智慧指標](#陷阱-3使用-get-後建立新的智慧指標)
  - [小結](#小結)

## 什麼是智慧指標？

**智慧指標（Smart Pointer）** 是 C++ 中用來自動管理動態記憶體的 RAII 類別。它們像一般指標一樣使用，但會在適當的時機自動釋放記憶體，避免記憶體洩漏和懸空指標等問題。

### 為什麼需要智慧指標？

**不使用智慧指標的問題：**

```cpp
void problematicFunction() {
    int* ptr = new int(42);

    // 做一些事情...

    if (errorCondition) {
        return;  // 記憶體洩漏！忘記 delete
    }

    riskyOperation();  // 如果拋出例外，記憶體洩漏！

    delete ptr;  // 可能永遠執行不到
}
```

**使用智慧指標：**

```cpp
void safeFunction() {
    std::unique_ptr<int> ptr = std::make_unique<int>(42);

    // 做一些事情...

    if (errorCondition) {
        return;  // 安全！自動釋放記憶體
    }

    riskyOperation();  // 即使拋出例外，記憶體也會被釋放

}  // 離開作用域時自動釋放
```

## 智慧指標的演進歷史

### 1. 史前時代：手動管理（C++98 以前）

```cpp
// 完全手動管理記憶體
void oldSchool() {
    MyClass* obj = new MyClass();

    try {
        obj->doSomething();
        delete obj;  // 必須記得刪除
    } catch (...) {
        delete obj;  // 例外處理也要記得刪除
        throw;
    }
}
```

**問題：**
- 容易忘記 delete
- 例外安全性難以保證
- 程式碼冗長且容易出錯

### 2. C++98：auto_ptr（已廢棄）

C++98 引入了第一個標準智慧指標 `auto_ptr`，但有嚴重的設計缺陷。

```cpp
#include <memory>

void autoPointerExample() {
    std::auto_ptr<int> ptr1(new int(42));

    // 危險！複製會轉移所有權
    std::auto_ptr<int> ptr2 = ptr1;  // ptr1 變成 nullptr！

    // *ptr1;  // 崩潰！ptr1 已經是 nullptr
    std::cout << *ptr2 << "\n";  // 42
}
```

**auto_ptr 的問題：**
- **複製語意不直覺**：複製會轉移所有權，原指標變 nullptr
- **無法用於 STL 容器**：容器需要可複製的物件
- **無法管理陣列**：沒有 `delete[]` 版本
- **不支援自定義刪除器**

**結果：**
- C++11 廢棄（deprecated）
- C++17 移除

### 3. C++11：現代智慧指標

C++11 引入了三個新的智慧指標，取代了 `auto_ptr`：

| 智慧指標 | 所有權模式 | 特性 |
|---------|----------|------|
| `unique_ptr` | 獨占所有權 | 不可複製，可移動，零開銷 |
| `shared_ptr` | 共享所有權 | 參考計數，可複製，有開銷 |
| `weak_ptr` | 非擁有參考 | 不增加參考計數，避免循環參考 |

## unique_ptr：獨占所有權

`unique_ptr` 是最常用的智慧指標，代表**獨占所有權**：同一時間只能有一個 `unique_ptr` 擁有資源。

### 基本使用

```cpp
#include <memory>
#include <iostream>

void uniquePtrBasics() {
    // 建立方式 1：使用 make_unique（推薦，C++14）
    auto ptr1 = std::make_unique<int>(42);

    // 建立方式 2：直接建構（不推薦）
    std::unique_ptr<int> ptr2(new int(100));

    // 使用方式：像一般指標
    std::cout << *ptr1 << "\n";  // 42

    // 取得裸指標（但不轉移所有權）
    int* raw = ptr1.get();

    // 檢查是否為空
    if (ptr1) {
        std::cout << "ptr1 不是空的\n";
    }

    // 釋放所有權並取得裸指標
    int* released = ptr1.release();
    // ptr1 現在是 nullptr，需要手動 delete released
    delete released;

    // 重置指標（釋放舊資源，指向新資源）
    ptr2.reset(new int(200));

}  // ptr2 自動釋放記憶體
```

### unique_ptr 不可複製，但可移動

```cpp
#include <memory>
#include <utility>

void uniquePtrMove() {
    auto ptr1 = std::make_unique<int>(42);

    // 不能複製
    // auto ptr2 = ptr1;  // 編譯錯誤！

    // 可以移動
    auto ptr2 = std::move(ptr1);  // 轉移所有權

    // 現在 ptr1 是 nullptr
    if (!ptr1) {
        std::cout << "ptr1 已經是空的\n";
    }

    std::cout << *ptr2 << "\n";  // 42
}
```

### 管理陣列

```cpp
#include <memory>

void uniquePtrArray() {
    // 管理陣列（注意使用 []）
    auto arr = std::make_unique<int[]>(10);

    // 可以像陣列一樣使用
    for (int i = 0; i < 10; ++i) {
        arr[i] = i * 10;
    }

    std::cout << arr[5] << "\n";  // 50

}  // 自動呼叫 delete[]
```

### 自定義刪除器

```cpp
#include <memory>
#include <iostream>

// 自定義刪除器（用於特殊資源）
void closeFile(FILE* fp) {
    if (fp) {
        std::cout << "關閉檔案\n";
        fclose(fp);
    }
}

void customDeleter() {
    // 使用自定義刪除器
    std::unique_ptr<FILE, decltype(&closeFile)> file(
        fopen("test.txt", "w"),
        closeFile
    );

    if (file) {
        fprintf(file.get(), "Hello, World!\n");
    }

}  // 自動呼叫 closeFile
```

### 完整範例：管理類別資源

```cpp
#include <memory>
#include <iostream>
#include <string>

class Resource {
private:
    std::string name;

public:
    explicit Resource(const std::string& n) : name(n) {
        std::cout << "建立資源: " << name << "\n";
    }

    ~Resource() {
        std::cout << "釋放資源: " << name << "\n";
    }

    void use() {
        std::cout << "使用資源: " << name << "\n";
    }
};

void demonstrateUniquePtr() {
    std::cout << "--- 開始 ---\n";

    {
        auto res1 = std::make_unique<Resource>("Resource 1");
        res1->use();

        // 轉移所有權
        auto res2 = std::move(res1);
        res2->use();

        std::cout << "--- 離開作用域 ---\n";
    }  // res2 自動釋放

    std::cout << "--- 結束 ---\n";
}

int main() {
    demonstrateUniquePtr();
    return 0;
}
```

**輸出：**
```
--- 開始 ---
建立資源: Resource 1
使用資源: Resource 1
使用資源: Resource 1
--- 離開作用域 ---
釋放資源: Resource 1
--- 結束 ---
```

## shared_ptr：共享所有權

`shared_ptr` 允許**多個指標共享同一個資源**，使用參考計數來追蹤有多少個 `shared_ptr` 指向該資源。當最後一個 `shared_ptr` 被銷毀時，資源才會被釋放。

### 基本使用

```cpp
#include <memory>
#include <iostream>

void sharedPtrBasics() {
    // 建立方式 1：使用 make_shared（推薦）
    auto ptr1 = std::make_shared<int>(42);

    std::cout << "值: " << *ptr1 << "\n";
    std::cout << "引用計數: " << ptr1.use_count() << "\n";  // 1

    {
        // 複製會增加引用計數
        auto ptr2 = ptr1;
        std::cout << "引用計數: " << ptr1.use_count() << "\n";  // 2

        auto ptr3 = ptr1;
        std::cout << "引用計數: " << ptr1.use_count() << "\n";  // 3

    }  // ptr2 和 ptr3 離開作用域

    std::cout << "引用計數: " << ptr1.use_count() << "\n";  // 1

}  // ptr1 離開作用域，資源被釋放
```

### shared_ptr 可以複製

```cpp
#include <memory>
#include <vector>

void sharedPtrCopy() {
    auto ptr1 = std::make_shared<int>(100);

    // 可以複製（與 unique_ptr 不同）
    auto ptr2 = ptr1;  // OK！共享所有權

    // 可以放入容器
    std::vector<std::shared_ptr<int>> vec;
    vec.push_back(ptr1);
    vec.push_back(ptr2);

    std::cout << "引用計數: " << ptr1.use_count() << "\n";  // 4
    // (ptr1 + ptr2 + vec[0] + vec[1])
}
```

### 完整範例：共享資源

```cpp
#include <memory>
#include <iostream>
#include <vector>

class SharedResource {
private:
    int id;

public:
    explicit SharedResource(int i) : id(i) {
        std::cout << "建立資源 " << id << "\n";
    }

    ~SharedResource() {
        std::cout << "釋放資源 " << id << "\n";
    }

    void display() const {
        std::cout << "資源 " << id << "\n";
    }
};

void demonstrateSharedPtr() {
    std::cout << "--- 建立第一個 shared_ptr ---\n";
    auto ptr1 = std::make_shared<SharedResource>(1);
    std::cout << "引用計數: " << ptr1.use_count() << "\n";  // 1

    {
        std::cout << "\n--- 建立第二個 shared_ptr ---\n";
        auto ptr2 = ptr1;  // 共享
        std::cout << "引用計數: " << ptr1.use_count() << "\n";  // 2

        {
            std::cout << "\n--- 建立第三個 shared_ptr ---\n";
            auto ptr3 = ptr1;  // 共享
            std::cout << "引用計數: " << ptr1.use_count() << "\n";  // 3

            ptr3->display();

            std::cout << "\n--- ptr3 離開作用域 ---\n";
        }

        std::cout << "引用計數: " << ptr1.use_count() << "\n";  // 2
        std::cout << "\n--- ptr2 離開作用域 ---\n";
    }

    std::cout << "引用計數: " << ptr1.use_count() << "\n";  // 1
    std::cout << "\n--- ptr1 離開作用域 ---\n";
}

int main() {
    demonstrateSharedPtr();
    std::cout << "\n--- 程式結束 ---\n";
    return 0;
}
```

**輸出：**
```
--- 建立第一個 shared_ptr ---
建立資源 1
引用計數: 1

--- 建立第二個 shared_ptr ---
引用計數: 2

--- 建立第三個 shared_ptr ---
引用計數: 3
資源 1

--- ptr3 離開作用域 ---
引用計數: 2

--- ptr2 離開作用域 ---
引用計數: 1

--- ptr1 離開作用域 ---
釋放資源 1

--- 程式結束 ---
```

### shared_ptr 的開銷

`shared_ptr` 比 `unique_ptr` 有額外開銷：

```cpp
// 記憶體開銷
sizeof(std::unique_ptr<int>)  // 8 bytes（一個指標）
sizeof(std::shared_ptr<int>)  // 16 bytes（兩個指標：資源 + 控制塊）

// 效能開銷
// - 參考計數的增減（原子操作）
// - 額外的記憶體分配（控制塊）
```

### make_shared 的優點

```cpp
// 不好：兩次記憶體分配
std::shared_ptr<int> ptr1(new int(42));
// 1. new int(42) - 分配物件
// 2. shared_ptr 建構子 - 分配控制塊

// 好：一次記憶體分配（推薦）
auto ptr2 = std::make_shared<int>(42);
// 一次分配：物件 + 控制塊
```

## weak_ptr：打破循環參考

`weak_ptr` 是一個**不擁有資源的智慧指標**，它指向 `shared_ptr` 管理的資源，但不增加引用計數。主要用於**避免循環參考**。

### 循環參考問題

```cpp
#include <memory>
#include <iostream>

class B;  // 前向宣告

class A {
public:
    std::shared_ptr<B> ptrB;
    ~A() { std::cout << "~A()\n"; }
};

class B {
public:
    std::shared_ptr<A> ptrA;  // 循環參考！
    ~B() { std::cout << "~B()\n"; }
};

void circularReference() {
    auto a = std::make_shared<A>();
    auto b = std::make_shared<B>();

    a->ptrB = b;  // A 指向 B
    b->ptrA = a;  // B 指向 A（循環！）

    std::cout << "a 引用計數: " << a.use_count() << "\n";  // 2
    std::cout << "b 引用計數: " << b.use_count() << "\n";  // 2

}  // 記憶體洩漏！A 和 B 都不會被釋放
   // 因為彼此的引用計數都不為 0

int main() {
    circularReference();
    std::cout << "程式結束\n";
    // 解構子不會被呼叫！
    return 0;
}
```

**輸出：**
```
a 引用計數: 2
b 引用計數: 2
程式結束
```

注意：解構子沒有被呼叫！

### 使用 weak_ptr 解決

```cpp
#include <memory>
#include <iostream>

class B;

class A {
public:
    std::shared_ptr<B> ptrB;
    ~A() { std::cout << "~A()\n"; }
};

class B {
public:
    std::weak_ptr<A> ptrA;  // 使用 weak_ptr 打破循環
    ~B() { std::cout << "~B()\n"; }
};

void noCircularReference() {
    auto a = std::make_shared<A>();
    auto b = std::make_shared<B>();

    a->ptrB = b;
    b->ptrA = a;  // weak_ptr 不增加引用計數

    std::cout << "a 引用計數: " << a.use_count() << "\n";  // 1
    std::cout << "b 引用計數: " << b.use_count() << "\n";  // 2

}  // A 和 B 都會被正確釋放

int main() {
    noCircularReference();
    std::cout << "程式結束\n";
    return 0;
}
```

**輸出：**
```
a 引用計數: 1
b 引用計數: 2
~A()
~B()
程式結束
```

### weak_ptr 的使用

```cpp
#include <memory>
#include <iostream>

void weakPtrUsage() {
    std::weak_ptr<int> weak;

    {
        auto shared = std::make_shared<int>(42);
        weak = shared;  // weak_ptr 指向 shared_ptr

        std::cout << "shared 引用計數: " << shared.use_count() << "\n";  // 1
        std::cout << "weak 引用計數: " << weak.use_count() << "\n";      // 1

        // 使用 weak_ptr：必須先轉換成 shared_ptr
        if (auto locked = weak.lock()) {  // 嘗試取得 shared_ptr
            std::cout << "值: " << *locked << "\n";  // 42
            std::cout << "引用計數: " << locked.use_count() << "\n";  // 2
        }

    }  // shared 離開作用域，資源被釋放

    // 資源已經被釋放
    if (weak.expired()) {
        std::cout << "weak_ptr 已失效\n";
    }

    if (auto locked = weak.lock()) {
        std::cout << "這行不會執行\n";
    } else {
        std::cout << "無法取得 shared_ptr（資源已釋放）\n";
    }
}
```

### 實用範例：觀察者模式

```cpp
#include <memory>
#include <iostream>
#include <vector>

class Observer {
public:
    virtual ~Observer() = default;
    virtual void update(const std::string& message) = 0;
};

class ConcreteObserver : public Observer {
private:
    std::string name;

public:
    explicit ConcreteObserver(const std::string& n) : name(n) {}

    void update(const std::string& message) override {
        std::cout << name << " 收到訊息: " << message << "\n";
    }
};

class Subject {
private:
    std::vector<std::weak_ptr<Observer>> observers;

public:
    void attach(std::shared_ptr<Observer> observer) {
        observers.push_back(observer);  // 儲存 weak_ptr
    }

    void notify(const std::string& message) {
        // 清理已失效的 weak_ptr
        observers.erase(
            std::remove_if(observers.begin(), observers.end(),
                [](const std::weak_ptr<Observer>& wp) {
                    return wp.expired();
                }),
            observers.end()
        );

        // 通知所有觀察者
        for (auto& wp : observers) {
            if (auto observer = wp.lock()) {
                observer->update(message);
            }
        }
    }
};

int main() {
    Subject subject;

    {
        auto obs1 = std::make_shared<ConcreteObserver>("觀察者1");
        auto obs2 = std::make_shared<ConcreteObserver>("觀察者2");

        subject.attach(obs1);
        subject.attach(obs2);

        std::cout << "--- 第一次通知 ---\n";
        subject.notify("Hello");

    }  // obs1 和 obs2 離開作用域

    std::cout << "\n--- 第二次通知（觀察者已釋放）---\n";
    subject.notify("World");

    return 0;
}
```

## 智慧指標的選擇

### 決策樹

```
需要管理動態記憶體？
├─ 否 → 不需要智慧指標，使用區域變數
└─ 是 → 繼續
    │
    ├─ 需要共享所有權？
    │  ├─ 是 → 使用 shared_ptr
    │  └─ 否 → 使用 unique_ptr
    │
    └─ 需要觀察但不擁有？
       └─ 是 → 使用 weak_ptr
```

### 使用場景

| 智慧指標 | 使用時機 | 典型場景 |
|---------|---------|---------|
| `unique_ptr` | 獨占所有權 | 工廠函式回傳值、類別成員變數 |
| `shared_ptr` | 共享所有權 | 多個物件需要存取同一資源 |
| `weak_ptr` | 觀察資源 | 快取、觀察者模式、打破循環參考 |

### 效能考量

```cpp
// 效能排序（從快到慢）
1. unique_ptr    // 零開銷，與裸指標相同
2. shared_ptr    // 參考計數開銷（原子操作）
3. weak_ptr      // 需要 lock() 轉換

// 記憶體使用
unique_ptr: 8 bytes
shared_ptr: 16 bytes（指標 + 控制塊指標）
weak_ptr:   16 bytes
```

## 實用範例

### 範例 1：工廠模式

```cpp
#include <memory>
#include <iostream>
#include <string>

class Shape {
public:
    virtual ~Shape() = default;
    virtual void draw() const = 0;
};

class Circle : public Shape {
public:
    void draw() const override {
        std::cout << "繪製圓形\n";
    }
};

class Rectangle : public Shape {
public:
    void draw() const override {
        std::cout << "繪製矩形\n";
    }
};

// 工廠函式回傳 unique_ptr
class ShapeFactory {
public:
    static std::unique_ptr<Shape> createShape(const std::string& type) {
        if (type == "circle") {
            return std::make_unique<Circle>();
        } else if (type == "rectangle") {
            return std::make_unique<Rectangle>();
        }
        return nullptr;
    }
};

int main() {
    auto shape1 = ShapeFactory::createShape("circle");
    auto shape2 = ShapeFactory::createShape("rectangle");

    if (shape1) shape1->draw();
    if (shape2) shape2->draw();

    return 0;
}
```

### 範例 2：資源池

```cpp
#include <memory>
#include <iostream>
#include <vector>
#include <string>

class Connection {
private:
    std::string id;

public:
    explicit Connection(const std::string& id) : id(id) {
        std::cout << "建立連線: " << id << "\n";
    }

    ~Connection() {
        std::cout << "關閉連線: " << id << "\n";
    }

    void execute(const std::string& query) {
        std::cout << "[" << id << "] 執行: " << query << "\n";
    }
};

class ConnectionPool {
private:
    std::vector<std::shared_ptr<Connection>> pool;

public:
    std::shared_ptr<Connection> getConnection() {
        if (pool.empty()) {
            // 建立新連線
            auto conn = std::make_shared<Connection>("conn-" + std::to_string(pool.size() + 1));
            pool.push_back(conn);
            return conn;
        }

        // 重用現有連線
        return pool.back();
    }

    size_t size() const { return pool.size(); }
};

int main() {
    ConnectionPool pool;

    {
        auto conn1 = pool.getConnection();
        conn1->execute("SELECT * FROM users");

        auto conn2 = pool.getConnection();  // 重用
        conn2->execute("INSERT INTO logs ...");

        std::cout << "連線池大小: " << pool.size() << "\n";
    }

    std::cout << "連線仍在池中，未釋放\n";
    return 0;
}
```

### 範例 3：PIMPL 慣用法

```cpp
// Widget.h
#include <memory>

class Widget {
private:
    class Impl;  // 前向宣告
    std::unique_ptr<Impl> pImpl;  // PIMPL

public:
    Widget();
    ~Widget();

    void doSomething();
};

// Widget.cpp
#include <iostream>

class Widget::Impl {
public:
    void doSomething() {
        std::cout << "實作細節\n";
    }
};

Widget::Widget() : pImpl(std::make_unique<Impl>()) {}

// 必須在 .cpp 中定義，因為需要知道 Impl 的完整定義
Widget::~Widget() = default;

void Widget::doSomething() {
    pImpl->doSomething();
}
```

## 最佳實踐

### DO（應該做的）

1. **優先使用 make_unique 和 make_shared**
   ```cpp
   // 好：例外安全、效率高
   auto ptr = std::make_unique<int>(42);
   auto sptr = std::make_shared<int>(100);

   // 不好：可能有例外安全問題
   std::unique_ptr<int> ptr(new int(42));
   ```

2. **使用 unique_ptr 作為預設選擇**
   ```cpp
   // 預設使用 unique_ptr
   std::unique_ptr<MyClass> obj = std::make_unique<MyClass>();

   // 只有在真正需要共享時才用 shared_ptr
   ```

3. **透過值回傳 unique_ptr**
   ```cpp
   std::unique_ptr<Widget> createWidget() {
       return std::make_unique<Widget>();
   }

   // 自動移動，不需要 std::move
   auto widget = createWidget();
   ```

4. **使用 weak_ptr 打破循環參考**
   ```cpp
   class Node {
       std::shared_ptr<Node> next;      // 擁有
       std::weak_ptr<Node> prev;        // 不擁有
   };
   ```

5. **明確所有權語意**
   ```cpp
   // 清楚表達：函式取得所有權
   void takeOwnership(std::unique_ptr<Widget> widget);

   // 清楚表達：函式共享所有權
   void shareOwnership(std::shared_ptr<Widget> widget);

   // 清楚表達：函式只是使用
   void useWidget(Widget* widget);  // 或 Widget&
   ```

### DON'T（不應該做的）

1. **不要使用 auto_ptr**
   ```cpp
   // 不要
   std::auto_ptr<int> ptr(new int(42));  // C++17 已移除

   // 好
   auto ptr = std::make_unique<int>(42);
   ```

2. **不要從裸指標建立多個 shared_ptr**
   ```cpp
   // 危險！會造成雙重刪除
   int* raw = new int(42);
   std::shared_ptr<int> ptr1(raw);
   std::shared_ptr<int> ptr2(raw);  // 錯誤！

   // 好
   auto ptr1 = std::make_shared<int>(42);
   auto ptr2 = ptr1;  // 正確的共享
   ```

3. **不要儲存 weak_ptr 指向的資源而不檢查**
   ```cpp
   // 危險
   std::weak_ptr<int> weak = getWeakPtr();
   // ... 時間過去 ...
   auto value = *weak.lock();  // 可能已失效！

   // 好
   if (auto locked = weak.lock()) {
       auto value = *locked;
   }
   ```

4. **不要過度使用 shared_ptr**
   ```cpp
   // 不好：不需要共享卻用 shared_ptr
   std::shared_ptr<int> ptr = std::make_shared<int>(42);

   // 好：獨占所有權用 unique_ptr
   auto ptr = std::make_unique<int>(42);
   ```

5. **不要用 shared_ptr 管理非動態記憶體**
   ```cpp
   int x = 42;

   // 危險！x 不是動態分配的
   std::shared_ptr<int> ptr(&x);  // 會嘗試 delete &x！

   // 如果必須，使用自定義刪除器
   std::shared_ptr<int> ptr(&x, [](int*){});  // 空刪除器
   ```

## 常見陷阱

### 陷阱 1：忘記定義解構子（PIMPL）

```cpp
// Widget.h
class Widget {
    class Impl;
    std::unique_ptr<Impl> pImpl;
public:
    Widget();
    // 缺少 ~Widget();  // 錯誤！
};

// 編譯錯誤：無法刪除不完整的型別
```

**解決方法：**
```cpp
// Widget.h
~Widget();  // 在 .h 宣告

// Widget.cpp
Widget::~Widget() = default;  // 在 .cpp 定義
```

### 陷阱 2：循環參考

```cpp
// 問題
class Node {
    std::shared_ptr<Node> parent;  // 造成循環
    std::shared_ptr<Node> child;
};

// 解決
class Node {
    std::weak_ptr<Node> parent;    // 使用 weak_ptr
    std::shared_ptr<Node> child;
};
```

### 陷阱 3：使用 get() 後建立新的智慧指標

```cpp
// 危險
auto ptr1 = std::make_shared<int>(42);
int* raw = ptr1.get();
std::shared_ptr<int> ptr2(raw);  // 雙重刪除！

// 好
auto ptr1 = std::make_shared<int>(42);
auto ptr2 = ptr1;  // 正確共享
```

## 小結

**智慧指標的核心概念：**

1. **歷史演進**：
   - **C++98**：`auto_ptr`（有缺陷，已廢棄）
   - **C++11**：`unique_ptr`、`shared_ptr`、`weak_ptr`（現代標準）

2. **三種智慧指標**：
   - **unique_ptr**：獨占所有權，不可複製，零開銷
   - **shared_ptr**：共享所有權，參考計數，可複製
   - **weak_ptr**：非擁有參考，打破循環參考

3. **選擇原則**：
   - 預設使用 `unique_ptr`
   - 需要共享時使用 `shared_ptr`
   - 需要觀察時使用 `weak_ptr`

4. **最佳實踐**：
   - 使用 `make_unique` 和 `make_shared`
   - 明確所有權語意
   - 避免從裸指標建立多個 `shared_ptr`
   - 使用 `weak_ptr` 打破循環參考

5. **效能考量**：
   - `unique_ptr`：零開銷（推薦）
   - `shared_ptr`：參考計數開銷
   - 不要過度使用 `shared_ptr`

**記住：智慧指標是現代 C++ 的核心特性，它們讓記憶體管理變得安全、簡單且自動化。正確使用智慧指標可以完全避免記憶體洩漏和懸空指標問題！**
