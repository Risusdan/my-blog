+++
date = '2025-12-16T23:53:15+08:00'
draft = false
title = '[C++] Operator Overloading（運算子多載）'
categories = ['C++ Notes']
tags = ['C++']
+++

# 目錄

- [目錄](#目錄)
  - [什麼是運算子多載？](#什麼是運算子多載)
    - [基本概念](#基本概念)
  - [為什麼需要運算子多載？](#為什麼需要運算子多載)
    - [提升程式碼可讀性](#提升程式碼可讀性)
  - [可以多載的運算子](#可以多載的運算子)
    - [可多載的運算子](#可多載的運算子)
    - [不可多載的運算子](#不可多載的運算子)
  - [成員函式 vs 非成員函式](#成員函式-vs-非成員函式)
    - [成員函式（Member Function）](#成員函式member-function)
    - [非成員函式（Non-member Function）](#非成員函式non-member-function)
    - [如何選擇？](#如何選擇)
    - [為什麼要區分成員函式和非成員函式？](#為什麼要區分成員函式和非成員函式)
      - [1. C++ 語言規定（強制要求）](#1-c-語言規定強制要求)
      - [2. 修改左運算元的運算子 → 成員函式](#2-修改左運算元的運算子--成員函式)
      - [3. 對稱性和隱式轉換 → 非成員函式](#3-對稱性和隱式轉換--非成員函式)
      - [4. 串流運算子必須是非成員函式](#4-串流運算子必須是非成員函式)
      - [實際範例：對比兩種方式](#實際範例對比兩種方式)
      - [總結：選擇準則](#總結選擇準則)
  - [常用運算子多載範例](#常用運算子多載範例)
    - [1. 算術運算子（+, -, \*, /）](#1-算術運算子----)
    - [2. 複合賦值運算子（+=, -=, \*=, /=）](#2-複合賦值運算子----)
    - [3. 比較運算子（==, !=, \<, \>, \<=, \>=）](#3-比較運算子-----)
    - [4. 串流運算子（\<\<, \>\>）](#4-串流運算子-)
    - [5. 下標運算子（\[\]）](#5-下標運算子)
    - [6. 遞增/遞減運算子（++, --）](#6-遞增遞減運算子---)
    - [7. 函式呼叫運算子（()）- Functor](#7-函式呼叫運算子--functor)
    - [8. 型別轉換運算子](#8-型別轉換運算子)
  - [運算子多載的最佳實踐](#運算子多載的最佳實踐)
    - [DO（應該做的）](#do應該做的)
    - [DON'T（不應該做的）](#dont不應該做的)
  - [完整範例：複數類別](#完整範例複數類別)
  - [小結](#小結)

## 什麼是運算子多載？

**運算子多載（Operator Overloading）** 是 C++ 的一個強大特性，允許我們為自定義類別定義運算子的行為，讓自定義類別能像內建型別一樣使用運算子。

### 基本概念

```cpp
// 內建型別可以使用運算子
int a = 5, b = 3;
int c = a + b;  // 使用 + 運算子

// 自定義類別也可以使用運算子（透過多載）
Complex c1(3, 4);
Complex c2(1, 2);
Complex c3 = c1 + c2;  // 使用多載的 + 運算子
```

運算子多載本質上是**用特殊名稱的函式來實作運算子的功能**：

```cpp
// a + b 實際上呼叫
operator+(a, b)

// std::cout << x 實際上呼叫
operator<<(std::cout, x)
```

## 為什麼需要運算子多載？

### 提升程式碼可讀性

**不使用運算子多載：**

```cpp
class Vector2D {
public:
    double x, y;

    Vector2D add(const Vector2D& other) const {
        return Vector2D(x + other.x, y + other.y);
    }

    Vector2D multiply(double scalar) const {
        return Vector2D(x * scalar, y * scalar);
    }
};

// 使用時需要呼叫方法
Vector2D v1(1, 2);
Vector2D v2(3, 4);
Vector2D v3 = v1.add(v2).multiply(2);  // 不直觀
```

**使用運算子多載：**

```cpp
class Vector2D {
public:
    double x, y;

    Vector2D operator+(const Vector2D& other) const {
        return Vector2D(x + other.x, y + other.y);
    }

    Vector2D operator*(double scalar) const {
        return Vector2D(x * scalar, y * scalar);
    }
};

// 使用時直觀清楚
Vector2D v1(1, 2);
Vector2D v2(3, 4);
Vector2D v3 = (v1 + v2) * 2;  // 清楚易懂！
```

## 可以多載的運算子

### 可多載的運算子

```cpp
// 算術運算子
+  -  *  /  %  +=  -=  *=  /=  %=

// 比較運算子
==  !=  <  >  <=  >=  <=>  (C++20)

// 邏輯運算子
!  &&  ||

// 位元運算子
&  |  ^  ~  <<  >>  &=  |=  ^=  <<=  >>=

// 遞增/遞減
++  --

// 存取運算子
[]  ->  ->*  ()

// 其他
=  ,  new  delete  new[]  delete[]
```

### 不可多載的運算子

```cpp
::   // 作用域解析
.    // 成員存取
.*   // 成員指標存取
?:   // 三元運算子
sizeof
typeid
```

## 成員函式 vs 非成員函式

### 成員函式（Member Function）

```cpp
class Complex {
private:
    double real, imag;

public:
    Complex(double r = 0, double i = 0) : real(r), imag(i) {}

    // 成員函式：左運算元是 this
    Complex operator+(const Complex& other) const {
        return Complex(real + other.real, imag + other.imag);
    }

    // 使用：c1.operator+(c2) 或 c1 + c2
};

Complex c1(1, 2);
Complex c2(3, 4);
Complex c3 = c1 + c2;  // c1.operator+(c2)
```

### 非成員函式（Non-member Function）

```cpp
class Complex {
private:
    double real, imag;

public:
    Complex(double r = 0, double i = 0) : real(r), imag(i) {}

    // 宣告為 friend 以存取私有成員
    friend Complex operator+(const Complex& lhs, const Complex& rhs);

    double getReal() const { return real; }
    double getImag() const { return imag; }
};

// 非成員函式：兩個運算元都是參數
Complex operator+(const Complex& lhs, const Complex& rhs) {
    return Complex(lhs.getReal() + rhs.getReal(),
                   lhs.getImag() + rhs.getImag());
}

Complex c1(1, 2);
Complex c2(3, 4);
Complex c3 = c1 + c2;  // operator+(c1, c2)
```

### 如何選擇？

| 使用成員函式 | 使用非成員函式 |
|------------|---------------|
| `=` `[]` `()` `->` | 需要隱式轉換左運算元時 |
| `+=` `-=` 等複合運算子 | 串流運算子 `<<` `>>` |
| 一元運算子（`++`, `--`, `!`） | 對稱的二元運算子（`+`, `-`, `*`） |

**經驗法則：**
- **必須是成員函式**：`=`、`[]`、`()`、`->`
- **建議非成員函式**：`<<`、`>>`、對稱的二元運算子
- **可以是任一種**：其他運算子

### 為什麼要區分成員函式和非成員函式？

理解何時選擇成員函式或非成員函式是運算子多載中最重要的設計考量之一。主要有三個關鍵原因：

#### 1. C++ 語言規定（強制要求）

某些運算子**必須**實作為成員函式，這是 C++ 語言的強制要求：

```cpp
class MyClass {
public:
    // 這些運算子必須是成員函式
    MyClass& operator=(const MyClass& other);    // 賦值
    MyClass& operator[](size_t index);           // 下標
    MyClass& operator()();                       // 函式呼叫
    MyClass* operator->();                       // 成員存取
};
```

**原因**：這些運算子與物件的內部狀態緊密相關，且需要直接修改左運算元（`this` 指標）。

#### 2. 修改左運算元的運算子 → 成員函式

如果運算子會修改左運算元，應該實作為成員函式：

```cpp
class Number {
    int value;
public:
    Number(int v) : value(v) {}

    // 會修改 this 物件，使用成員函式
    Number& operator+=(const Number& rhs) {
        value += rhs.value;  // 修改自己
        return *this;
    }

    // 不修改任何運算元，使用非成員函式
    friend Number operator+(const Number& lhs, const Number& rhs) {
        return Number(lhs.value + rhs.value);
    }
};

Number a(5), b(3);
a += b;  // a 被修改了
Number c = a + b;  // a 和 b 都不會被修改
```

#### 3. 對稱性和隱式轉換 → 非成員函式

這是**最關鍵**的設計考量。讓我們看看為什麼對稱的二元運算子應該使用非成員函式。

**問題：成員函式無法對稱處理隱式轉換**

```cpp
class Number {
    int value;
public:
    Number(int v) : value(v) {}

    // 錯誤示範：使用成員函式
    Number operator+(const Number& rhs) const {
        return Number(value + rhs.value);
    }
};

Number a(5);
Number result1 = a + 10;   // 可以！10 被隱式轉換成 Number(10)
Number result2 = 10 + a;   // 編譯錯誤！int 沒有 operator+ 可以接受 Number
```

**為什麼 `10 + a` 會失敗？**

- `a + 10` 實際上是 `a.operator+(10)`
  - 編譯器可以將 `10` 隱式轉換成 `Number(10)`
  - 呼叫成功：`a.operator+(Number(10))`

- `10 + a` 會變成 `10.operator+(a)`
  - 但 `int` 型別沒有這個運算子！
  - 編譯失敗

**解決方案：使用非成員函式實現對稱性**

```cpp
class Number {
    int value;
public:
    Number(int v) : value(v) {}

    // 正確做法：使用非成員函式
    friend Number operator+(const Number& lhs, const Number& rhs) {
        return Number(lhs.value + rhs.value);
    }
};

Number a(5);
Number result1 = a + 10;   // 可以！operator+(a, Number(10))
Number result2 = 10 + a;   // 可以！operator+(Number(10), a)
```

現在兩個參數都可以進行隱式轉換，實現了**對稱性**！

#### 4. 串流運算子必須是非成員函式

```cpp
class Number {
    int value;
public:
    Number(int v) : value(v) {}

    // 正確：非成員函式
    friend std::ostream& operator<<(std::ostream& os, const Number& num) {
        os << num.value;
        return os;
    }
};

Number a(42);
std::cout << a;  // 呼叫 operator<<(std::cout, a)
```

**為什麼不能是成員函式？**

如果寫成成員函式：

```cpp
class Number {
    // 如果是成員函式
    std::ostream& operator<<(std::ostream& os) const {
        os << value;
        return os;
    }
};

// 使用時會變成：
a << std::cout;  // 這樣很不自然！
```

我們無法修改 `std::ostream` 類別來加入對我們自定義類別的支援，所以必須使用非成員函式，讓 `std::cout` 作為左運算元。

#### 實際範例：對比兩種方式

```cpp
class Point {
    int x, y;
public:
    Point(int x = 0, int y = 0) : x(x), y(y) {}

    // 成員函式版本（不對稱）
    Point operator+(const Point& other) const {
        return Point(x + other.x, y + other.y);
    }
};

Point p1(1, 2);
Point p2(3, 4);
Point p3 = p1 + p2;      // 可以
// Point p4 = p1 + 10;   // 無法編譯（即使有轉換建構子）
```

```cpp
class Point {
    int x, y;
public:
    Point(int x = 0, int y = 0) : x(x), y(y) {}

    // 非成員函式版本（對稱）
    friend Point operator+(const Point& lhs, const Point& rhs) {
        return Point(lhs.x + rhs.x, lhs.y + rhs.y);
    }

    friend Point operator+(const Point& p, int offset) {
        return Point(p.x + offset, p.y + offset);
    }

    friend Point operator+(int offset, const Point& p) {
        return Point(p.x + offset, p.y + offset);
    }
};

Point p1(1, 2);
Point p2 = p1 + 10;      // 可以
Point p3 = 10 + p1;      // 可以（對稱）
```

#### 總結：選擇準則

| 情況 | 選擇 | 原因 |
|------|------|------|
| `=` `[]` `()` `->` | 成員函式 | C++ 強制規定 |
| `+=` `-=` `*=` `++` `--` | 成員函式 | 會修改左運算元 |
| 一元運算子 `!` `~` `-` | 成員函式 | 通常只操作一個物件 |
| `+` `-` `*` `/` `==` `!=` | 非成員函式 | 需要對稱的隱式轉換 |
| `<<` `>>` (串流) | 非成員函式 | 左運算元不是我們的類別 |

**核心原則**：
- 如果運算子會**修改物件狀態** → 成員函式
- 如果需要**兩側對稱**的隱式轉換 → 非成員函式
- 如果**左運算元不是你的類別** → 非成員函式
- 如果 **C++ 語言規定** → 成員函式

## 常用運算子多載範例

### 1. 算術運算子（+, -, *, /）

```cpp
class Fraction {
private:
    int numerator;    // 分子
    int denominator;  // 分母

    // 簡化分數
    void simplify() {
        int gcd = std::gcd(numerator, denominator);
        numerator /= gcd;
        denominator /= gcd;
    }

public:
    Fraction(int num = 0, int denom = 1)
        : numerator(num), denominator(denom)
    {
        if (denom == 0) {
            throw std::invalid_argument("分母不能為零");
        }
        simplify();
    }

    // 加法：非成員函式（對稱）
    friend Fraction operator+(const Fraction& lhs, const Fraction& rhs) {
        return Fraction(
            lhs.numerator * rhs.denominator + rhs.numerator * lhs.denominator,
            lhs.denominator * rhs.denominator
        );
    }

    // 減法
    friend Fraction operator-(const Fraction& lhs, const Fraction& rhs) {
        return Fraction(
            lhs.numerator * rhs.denominator - rhs.numerator * lhs.denominator,
            lhs.denominator * rhs.denominator
        );
    }

    // 乘法
    friend Fraction operator*(const Fraction& lhs, const Fraction& rhs) {
        return Fraction(
            lhs.numerator * rhs.numerator,
            lhs.denominator * rhs.denominator
        );
    }

    // 除法
    friend Fraction operator/(const Fraction& lhs, const Fraction& rhs) {
        return Fraction(
            lhs.numerator * rhs.denominator,
            lhs.denominator * rhs.numerator
        );
    }

    // 串流輸出
    friend std::ostream& operator<<(std::ostream& os, const Fraction& f) {
        os << f.numerator << "/" << f.denominator;
        return os;
    }
};

// 使用範例
int main() {
    Fraction f1(1, 2);  // 1/2
    Fraction f2(1, 3);  // 1/3

    Fraction sum = f1 + f2;      // 5/6
    Fraction diff = f1 - f2;     // 1/6
    Fraction prod = f1 * f2;     // 1/6
    Fraction quot = f1 / f2;     // 3/2

    std::cout << f1 << " + " << f2 << " = " << sum << "\n";
    // 輸出: 1/2 + 1/3 = 5/6

    return 0;
}
```

### 2. 複合賦值運算子（+=, -=, *=, /=）

```cpp
class Vector2D {
private:
    double x, y;

public:
    Vector2D(double x = 0, double y = 0) : x(x), y(y) {}

    // 複合賦值運算子：成員函式（修改 this）
    Vector2D& operator+=(const Vector2D& other) {
        x += other.x;
        y += other.y;
        return *this;  // 回傳自己的參考，支援串接
    }

    Vector2D& operator-=(const Vector2D& other) {
        x -= other.x;
        y -= other.y;
        return *this;
    }

    Vector2D& operator*=(double scalar) {
        x *= scalar;
        y *= scalar;
        return *this;
    }

    // 普通算術運算子：用複合賦值實作
    Vector2D operator+(const Vector2D& other) const {
        Vector2D result = *this;  // 複製
        result += other;          // 使用 +=
        return result;
    }

    Vector2D operator-(const Vector2D& other) const {
        Vector2D result = *this;
        result -= other;
        return result;
    }

    Vector2D operator*(double scalar) const {
        Vector2D result = *this;
        result *= scalar;
        return result;
    }

    friend std::ostream& operator<<(std::ostream& os, const Vector2D& v) {
        os << "(" << v.x << ", " << v.y << ")";
        return os;
    }
};

// 使用範例
int main() {
    Vector2D v1(1, 2);
    Vector2D v2(3, 4);

    v1 += v2;  // v1 變成 (4, 6)
    std::cout << v1 << "\n";

    // 串接使用
    v1 += v2 += Vector2D(1, 1);
    // v2 變成 (4, 5)
    // v1 變成 (8, 11)

    return 0;
}
```

### 3. 比較運算子（==, !=, <, >, <=, >=）

```cpp
class Point {
private:
    int x, y;

public:
    Point(int x = 0, int y = 0) : x(x), y(y) {}

    // 相等運算子
    bool operator==(const Point& other) const {
        return x == other.x && y == other.y;
    }

    // 不等運算子（用 == 實作）
    bool operator!=(const Point& other) const {
        return !(*this == other);
    }

    // 小於運算子（定義排序順序）
    bool operator<(const Point& other) const {
        if (x != other.x) {
            return x < other.x;
        }
        return y < other.y;
    }

    // 其他比較運算子可以用 < 和 == 實作
    bool operator>(const Point& other) const {
        return other < *this;
    }

    bool operator<=(const Point& other) const {
        return !(other < *this);
    }

    bool operator>=(const Point& other) const {
        return !(*this < other);
    }

    friend std::ostream& operator<<(std::ostream& os, const Point& p) {
        os << "(" << p.x << ", " << p.y << ")";
        return os;
    }
};

// C++20：三向比較運算子（Spaceship Operator）
class ModernPoint {
private:
    int x, y;

public:
    ModernPoint(int x = 0, int y = 0) : x(x), y(y) {}

    // 只需定義一個運算子，自動產生其他比較運算子
    auto operator<=>(const ModernPoint& other) const = default;
    bool operator==(const ModernPoint& other) const = default;
};
```

### 4. 串流運算子（<<, >>）

```cpp
class Time {
private:
    int hours, minutes, seconds;

public:
    Time(int h = 0, int m = 0, int s = 0)
        : hours(h), minutes(m), seconds(s) {}

    // 輸出運算子：必須是非成員函式
    friend std::ostream& operator<<(std::ostream& os, const Time& t) {
        os << std::setfill('0') << std::setw(2) << t.hours << ":"
           << std::setw(2) << t.minutes << ":"
           << std::setw(2) << t.seconds;
        return os;  // 支援串接
    }

    // 輸入運算子
    friend std::istream& operator>>(std::istream& is, Time& t) {
        char colon;
        is >> t.hours >> colon >> t.minutes >> colon >> t.seconds;

        // 驗證輸入
        if (t.hours < 0 || t.hours > 23 ||
            t.minutes < 0 || t.minutes > 59 ||
            t.seconds < 0 || t.seconds > 59) {
            is.setstate(std::ios::failbit);
        }

        return is;
    }
};

// 使用範例
int main() {
    Time t1(14, 30, 45);
    std::cout << "時間: " << t1 << "\n";
    // 輸出: 時間: 14:30:45

    Time t2;
    std::cout << "輸入時間 (HH:MM:SS): ";
    std::cin >> t2;
    std::cout << "你輸入的時間: " << t2 << "\n";

    return 0;
}
```

### 5. 下標運算子（[]）

```cpp
class Array {
private:
    int* data;
    size_t size;

public:
    explicit Array(size_t n) : size(n), data(new int[n]()) {}

    ~Array() {
        delete[] data;
    }

    // 非 const 版本：可修改
    int& operator[](size_t index) {
        if (index >= size) {
            throw std::out_of_range("索引超出範圍");
        }
        return data[index];
    }

    // const 版本：唯讀
    const int& operator[](size_t index) const {
        if (index >= size) {
            throw std::out_of_range("索引超出範圍");
        }
        return data[index];
    }

    size_t getSize() const { return size; }
};

// 使用範例
int main() {
    Array arr(5);

    // 使用 [] 設定值
    arr[0] = 10;
    arr[1] = 20;

    // 使用 [] 讀取值
    std::cout << arr[0] << "\n";  // 10

    // const 物件使用 const 版本
    const Array& constArr = arr;
    std::cout << constArr[0] << "\n";  // 10
    // constArr[0] = 30;  // 編譯錯誤！const 版本回傳 const int&

    return 0;
}
```

### 6. 遞增/遞減運算子（++, --）

```cpp
class Counter {
private:
    int value;

public:
    explicit Counter(int v = 0) : value(v) {}

    // 前置遞增：++counter
    Counter& operator++() {
        ++value;
        return *this;  // 回傳遞增後的物件
    }

    // 後置遞增：counter++
    // 參數 int 只是用來區分前置和後置
    Counter operator++(int) {
        Counter temp = *this;  // 儲存舊值
        ++value;               // 遞增
        return temp;           // 回傳舊值
    }

    // 前置遞減：--counter
    Counter& operator--() {
        --value;
        return *this;
    }

    // 後置遞減：counter--
    Counter operator--(int) {
        Counter temp = *this;
        --value;
        return temp;
    }

    int getValue() const { return value; }

    friend std::ostream& operator<<(std::ostream& os, const Counter& c) {
        os << c.value;
        return os;
    }
};

// 使用範例
int main() {
    Counter c(10);

    std::cout << "初始值: " << c << "\n";          // 10

    std::cout << "前置遞增: " << ++c << "\n";      // 11
    std::cout << "現在的值: " << c << "\n";        // 11

    std::cout << "後置遞增: " << c++ << "\n";      // 11
    std::cout << "現在的值: " << c << "\n";        // 12

    // 效能差異：
    ++c;   // 前置：效率高，直接修改並回傳參考
    c++;   // 後置：效率低，需要建立臨時物件

    return 0;
}
```

### 7. 函式呼叫運算子（()）- Functor

```cpp
class Adder {
private:
    int offset;

public:
    explicit Adder(int offset) : offset(offset) {}

    // 函式呼叫運算子：讓物件像函式一樣使用
    int operator()(int value) const {
        return value + offset;
    }
};

class Multiplier {
private:
    double factor;

public:
    explicit Multiplier(double factor) : factor(factor) {}

    double operator()(double value) const {
        return value * factor;
    }
};

// 使用範例
int main() {
    Adder add5(5);
    std::cout << add5(10) << "\n";  // 15
    std::cout << add5(20) << "\n";  // 25

    Multiplier times2(2.0);
    std::cout << times2(3.5) << "\n";  // 7.0

    // 與 STL 演算法搭配使用
    std::vector<int> vec = {1, 2, 3, 4, 5};
    std::transform(vec.begin(), vec.end(), vec.begin(), Adder(10));
    // vec 變成 {11, 12, 13, 14, 15}

    return 0;
}
```

### 8. 型別轉換運算子

```cpp
class Celsius {
private:
    double temp;

public:
    explicit Celsius(double t) : temp(t) {}

    // 轉換成 double（隱式）
    operator double() const {
        return temp;
    }

    // 轉換成 int（顯式）
    explicit operator int() const {
        return static_cast<int>(temp);
    }

    double getTemp() const { return temp; }
};

class Fahrenheit {
private:
    double temp;

public:
    explicit Fahrenheit(double t) : temp(t) {}

    // 從 Celsius 轉換（轉換建構子）
    Fahrenheit(const Celsius& c)
        : temp(c.getTemp() * 9.0 / 5.0 + 32.0) {}

    double getTemp() const { return temp; }

    friend std::ostream& operator<<(std::ostream& os, const Fahrenheit& f) {
        os << f.temp << "°F";
        return os;
    }
};

// 使用範例
int main() {
    Celsius c(100);  // 100°C

    // 隱式轉換成 double
    double d = c;
    std::cout << d << "\n";  // 100

    // 顯式轉換成 int
    int i = static_cast<int>(c);
    std::cout << i << "\n";  // 100

    // 轉換成 Fahrenheit
    Fahrenheit f = c;
    std::cout << f << "\n";  // 212°F

    return 0;
}
```

## 運算子多載的最佳實踐

### DO（應該做的）

1. **保持運算子的直覺意義**
   ```cpp
   // 好：+ 用於加法
   Complex operator+(const Complex& lhs, const Complex& rhs);

   // 不好：+ 用於減法（違反直覺）
   Complex operator+(const Complex& lhs, const Complex& rhs) {
       return Complex(lhs.real - rhs.real, lhs.imag - rhs.imag);  // 錯誤！
   }
   ```

2. **用複合運算子實作普通運算子**
   ```cpp
   class Vector {
   public:
       // 先實作 +=（效率高）
       Vector& operator+=(const Vector& other) {
           x += other.x;
           y += other.y;
           return *this;
       }

       // 用 += 實作 +
       Vector operator+(const Vector& other) const {
           Vector result = *this;
           result += other;
           return result;
       }
   };
   ```

3. **對稱的運算子使用非成員函式**
   ```cpp
   // 好：兩個運算元地位平等
   Complex operator+(const Complex& lhs, const Complex& rhs);

   // 不好：左運算元特殊
   Complex Complex::operator+(const Complex& rhs);
   ```

4. **串流運算子回傳串流參考**
   ```cpp
   std::ostream& operator<<(std::ostream& os, const MyClass& obj) {
       os << /* ... */;
       return os;  // 支援串接：cout << obj1 << obj2
   }
   ```

5. **提供 const 和非 const 版本**
   ```cpp
   class Array {
   public:
       int& operator[](size_t index);              // 非 const：可修改
       const int& operator[](size_t index) const;  // const：唯讀
   };
   ```

### DON'T（不應該做的）

1. **不要多載 &&、||、,（逗號）運算子**
   ```cpp
   // 不好：失去短路求值
   bool operator&&(const MyBool& lhs, const MyBool& rhs) {
       return lhs.value && rhs.value;  // 兩個運算元都會求值
   }
   ```

2. **不要改變運算子的優先順序和結合性**
   ```cpp
   // 多載不會改變優先順序
   // * 永遠比 + 優先，無論你如何多載
   ```

3. **不要濫用運算子多載**
   ```cpp
   // 不好：<< 用於加入元素（不直覺）
   class List {
   public:
       List& operator<<(int value) {
           push_back(value);
           return *this;
       }
   };

   // 好：用有意義的函式名稱
   class List {
   public:
       void push_back(int value);
   };
   ```

4. **不要讓運算子有副作用（除了賦值類）**
   ```cpp
   // 不好：+ 改變左運算元
   Vector operator+(const Vector& other) {
       x += other.x;  // 錯誤！不應該修改 this
       y += other.y;
       return *this;
   }

   // 好：+ 不改變運算元
   Vector operator+(const Vector& other) const {
       return Vector(x + other.x, y + other.y);
   }
   ```

5. **不要多載 . 或 :: 或 ?:**
   ```cpp
   // 不可能：這些運算子無法多載
   ```

## 完整範例：複數類別

```cpp
#include <iostream>
#include <cmath>

class Complex {
private:
    double real, imag;

public:
    // 建構子
    Complex(double r = 0, double i = 0) : real(r), imag(i) {}

    // Getters
    double getReal() const { return real; }
    double getImag() const { return imag; }

    // 算術運算子（非成員函式）
    friend Complex operator+(const Complex& lhs, const Complex& rhs) {
        return Complex(lhs.real + rhs.real, lhs.imag + rhs.imag);
    }

    friend Complex operator-(const Complex& lhs, const Complex& rhs) {
        return Complex(lhs.real - rhs.real, lhs.imag - rhs.imag);
    }

    friend Complex operator*(const Complex& lhs, const Complex& rhs) {
        return Complex(
            lhs.real * rhs.real - lhs.imag * rhs.imag,
            lhs.real * rhs.imag + lhs.imag * rhs.real
        );
    }

    // 複合賦值運算子（成員函式）
    Complex& operator+=(const Complex& other) {
        real += other.real;
        imag += other.imag;
        return *this;
    }

    Complex& operator-=(const Complex& other) {
        real -= other.real;
        imag -= other.imag;
        return *this;
    }

    // 一元運算子
    Complex operator-() const {
        return Complex(-real, -imag);
    }

    // 比較運算子
    bool operator==(const Complex& other) const {
        return real == other.real && imag == other.imag;
    }

    bool operator!=(const Complex& other) const {
        return !(*this == other);
    }

    // 串流運算子
    friend std::ostream& operator<<(std::ostream& os, const Complex& c) {
        os << c.real;
        if (c.imag >= 0) {
            os << " + " << c.imag << "i";
        } else {
            os << " - " << -c.imag << "i";
        }
        return os;
    }

    friend std::istream& operator>>(std::istream& is, Complex& c) {
        is >> c.real >> c.imag;
        return is;
    }

    // 輔助函式
    double abs() const {
        return std::sqrt(real * real + imag * imag);
    }
};

int main() {
    Complex c1(3, 4);
    Complex c2(1, 2);

    std::cout << "c1 = " << c1 << "\n";  // 3 + 4i
    std::cout << "c2 = " << c2 << "\n";  // 1 + 2i

    Complex sum = c1 + c2;
    std::cout << "c1 + c2 = " << sum << "\n";  // 4 + 6i

    Complex diff = c1 - c2;
    std::cout << "c1 - c2 = " << diff << "\n";  // 2 + 2i

    Complex prod = c1 * c2;
    std::cout << "c1 * c2 = " << prod << "\n";  // -5 + 10i

    c1 += c2;
    std::cout << "c1 += c2: " << c1 << "\n";  // 4 + 6i

    Complex neg = -c1;
    std::cout << "-c1 = " << neg << "\n";  // -4 - 6i

    std::cout << "|c1| = " << c1.abs() << "\n";  // 7.21...

    if (c1 == Complex(4, 6)) {
        std::cout << "c1 等於 4 + 6i\n";
    }

    return 0;
}
```

## 小結

**運算子多載的核心概念：**

1. **定義**：為自定義類別定義運算子的行為
   - 讓自定義類別像內建型別一樣使用運算子
   - 提升程式碼可讀性和直覺性

2. **實作方式**：
   - **成員函式**：適合修改左運算元的運算子（`+=`、`[]`、`()`）
   - **非成員函式**：適合對稱的運算子（`+`、`-`、`<<`）

3. **常用運算子**：
   - 算術：`+`、`-`、`*`、`/`
   - 複合賦值：`+=`、`-=`、`*=`、`/=`
   - 比較：`==`、`!=`、`<`、`>`、`<=`、`>=`
   - 串流：`<<`、`>>`
   - 存取：`[]`、`()`
   - 遞增/遞減：`++`、`--`

4. **最佳實踐**：
   - 保持運算子的直覺意義
   - 用複合運算子實作普通運算子
   - 對稱的運算子使用非成員函式
   - 提供 const 和非 const 版本
   - 不要濫用運算子多載

5. **注意事項**：
   - 不能改變優先順序和結合性
   - 不能多載 `.`、`::`、`?:`
   - 避免多載 `&&`、`||`、`,`
   - 某些運算子必須是成員函式（`=`、`[]`、`()`、`->`）
