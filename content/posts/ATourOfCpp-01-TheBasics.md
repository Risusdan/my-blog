+++
draft = false
date = '2025-08-22T10:00:00-07:00'
title = 'A Tour of C++ 01: The Basics'
categories = ['A Tour of C++']
tags = ['C++']
+++

## Table of Contents

- [Table of Contents](#table-of-contents)
- [Terms](#terms)
- [Programs](#programs)
- [Hello, World!](#hello-world)
- [Functions](#functions)
- [Types, Variables, and Arithmetic](#types-variables-and-arithmetic)
- [Initialization](#initialization)
- [Scope and Lifetime](#scope-and-lifetime)
- [Constants](#constants)
- [Pointers, Arrays, and References](#pointers-arrays-and-references)
  - [for, range-for](#for-range-for)
- [The Null Pointer](#the-null-pointer)
- [Tests](#tests)
- [Mapping to Hardware](#mapping-to-hardware)
  - [Assignment](#assignment)
  - [Initialization](#initialization-1)

## Terms

- `{}` is called *curly braces*
- `[]` is called *square brackets*
- `()` is called *parentheses*
- `<>` is called *angle brackets* or *chevrons*
- `""` is called *double quotes* or *quotation marks*
- `''` is called *single quotes* or *apostrophes*
- `||` is called *pipes* or *vertical bars*
- `//` is called *forward slashes* or *double slash*
- `\\` is called *backslashes*
- `<<` is called *put to*
- `>>` is called *get from*

## Programs

- The ISO C++ standard defines two kinds of entities:
    1. **Core language features**, such as built-in types (e.g., `char`, `int`) and loops (e.g., `for`, `while`)
    2. **Standard-library component**, such as containers (e.g., `vector`, `map`) and I/O operations (e.g., `<<`, `getline()`)
- C++ is a statically typed language. That is, the type of every entity (e.g., object, value, expression) must be known to the compiler at its point to use.

## Hello, World!

```cpp
import std;

int main() {
    std::cout << "Hello World!\n";
}
```

- The line `import std;` instructs the compiler to make the decalrations of the standard library available.
- The operator `<<` writes its second argument to the first.
- The `std::` specifies that the name `cout` is to be found in the standard library namespace.
- The `import` directive is new in C++20, you can also try the old-fasioned one:

```cpp
#include <iostream>

int main() {
    std::cout << "Hello World!\n";
}
```

## Functions

- A function declaration may contain argument names. This can be help to the reader of a program, but unless the delcaration is also a function definition, the compiler simply ignores such names.

```cpp
double sqrt(double);
double sqrt(double d);
```

- A function can be a member of a class, called *member function*. For a member function, the name of its class is also part of the function type.

```cpp
char& String::operator[](int);
```

- If two function are defined with the same name, but with different argument types, the compiler will choose the most apprepriate function to invoke fot each call. This is known as *function overloading*.

```cpp
void print(int);
void print(double);
void print(string);

void use() {
    print(2);
    print(8.3);
    print("Good");
}
```

- If two alternative functions could be called, but neither is better than the other, the call is deemed ambiguous and the compiler gives an error.

```cpp
void print(int, double);
void print(double, int);

void use() {
    print(0, 0);
}
```

## Types, Variables, and Arithmetic

- A *decalration* is a statement that introduces an entity into the program and specifies its type.
- A *type* defines a set of possiable values and a set of operations (for an object)
- An *object* is some memory that holds a value of some type.
- A *value* is a set of bits interpreted according to a type.
- A *variable* is a named project.

- The size of a type can vary among different machines. When we want a type of a specific size, we use a standard-library type alias, such as `int32_t`.
- A `0b` prefix indicates a binary integer literal (e.g., `0b10101010`).
- A `0x` prefix indicates a hexadecimal integer literal (e.g., `0x89C3`).
- A `0` prefix indicates a octal integer literal (e.g., `0334`).
- To make a lone literals more readable for humans, we can use a single quote as a diit seperator (e.g., `3.14159'26535'89793`).

- Note that `=` is the assignmeent operator, and `==` tests equality.

## Initialization

- C++ offers a variety of notations for expressing initialization, such as `=`, and a universal form based on curly-brace-delimited initializer lists:

```cpp
double d1 = 2.4;
double d2 {2.4};
double d3 = {2.4};

vector<int> v {1, 2, 4, 6};
```

- Use the general list form can save you from conversions that lose information:

```cpp
int i1 = 8.3; // i1 becomes 8 :(
int i2 {8.3}; // will raise an error
```

- Narrowing conversions, such as `double` to `int` and `int` to `char`, are allowed and implicitly applied when you use `=` (but not when you use `{}`).
- We use `auto` where we don't have a specific reason to mention the type explicitly. "Specific reasons" include:
  - The definition is in a large scope where we want to make the type clearly visible to readers of our code.
  - The type of the initializer isn't obvious.
  - We want to be exolicit about the variable's range or precision (e.r., `double` rather than `float`).

## Scope and Lifetime

- A declaration introduces its name into a scope:
  - *Local scope*: 
    - A name decalre in a function or lambda is called a *local name*.
    - Its scope extends from its point of declaration to the end of the block in which its declaration occurs.
    - A *block* is delimited by a `{}` pair.
    - Function argment names are considered local names.
  - *Class scope*:
    - A name is called a *member name* if it is defined in a class, outside any function, lambda, or enum class.
    - Its scope extends from the opening *{* of its enclosing declaration to the matching *}*.
  - *Namespcae scope*:
    - A name is called a *namespace member name* if it is defined in a namespace outside any function, lambda, class, or enum class.
    - Its scope extends from the point of declaration to the end of its namespace.
  - A name is not declared inside any other construct is called a *global name* and is said to be in the global namespace.
- In addition, we can have objects without names, such as temporaries and objects created using `new`.
- An object created by `new` lives until destroyed by `delete`.

```cpp
auto p = new Record("Home"); // p points to a unnamed Record that created by `new`
```

## Constants

- C++ supports two notions of immutability(an object with an unchangeable state).
- `const` is about immutability, `constexpr` is about compile-time evaluation.
- Compile-time evaluation is important for performance.
- `const`: 
  - Meaning roughly "I promise not to change this valie"
  - Runtime or compile-time: Value can be determined at runtime
  - Immutability: Variable cannot be modified after initialization
  - Usage: Read-only variables, function parameters, return types

```cpp
const int x = 10;           // Compile-time constant
const int y = rand();       // Runtime constant - OK
const std::string s = "hi"; // Runtime constant
```

- `constexpr`:
  - Meaning roughly "to be evaluated at compile time"
  - Compile-time only: Value must be computable at compile-time
  - Constant expressions: Can be used in contexts requiring compile-time constants
  - Functions: Can be evaluated at compile-time if arguments are compile-time constants

```cpp
constexpr int x = 10;       // Must be compile-time constant
constexpr int y = rand();   // ERROR - rand() not compile-time
constexpr int z = x + 5;    // OK - computed at compile-time

constexpr int square(int n) { return n * n; }
constexpr int result = square(5); // Evaluated at compile-time
```

## Pointers, Arrays, and References

- An array of elements of type `char` can be declared like this: `char v[6];`.
- `[]` means "array of"
- A pointer can be decalred like this: `char* p;`.
- `*` means "pointer to".

```cpp
char* p = &v[3]; // p points to v's 4th element
char x = *p; // *p is the object that p points to
```

- In an expression, prefix unary `*` means "contents of" and prefix unary `&` means "address of".

### for, range-for

- The following for-statement can be read as "set i to zero, while i is not 10, print the ith element and increment i".

```cpp
void print1() {
    int v1[10] = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
    
    for(auto i=0; i!=10; i++) {
        cout << v[i] << "\n";
    }
}
```

- C++ also offers a simpler for-statement called a range-for-statement.
- The following range-for-statement can be read as "for every element of v, place a copy in x and print it".

```cpp
void print2() {
    int v1[10] = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
    
    for(auto x: v) {
        cout << x << "\n";
    }
}
```

- If we didn't want to copy the values from `v` into the variable `x`, but rather than just have `x` refer to an element, we could write:

```cpp
void print3() {
    int v1[10] = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
    
    for(auto& x: v) { // x is a reference to each element of v
        cout << x << "\n";
    }
}
```

- A *reference* is similar to a pointer, except that you don't need to use a prefix `*` to access to the value referred to by the reference.

```cpp
int val = 42;

int* ptr = &val;
cout << *ptr << endl; // Need * to get value

int& ref = val;
cout << ref << endl; // Direct access
```

- Also, a reference can not be made to refer to a different object after its initialization.

```cpp
int a = 10;
int b = 20;

// Pointer can be reassigned
int* ptr = &a;
ptr = &b;

// Reference can not be reassigned
int& ref = a;
ref = b; // This copies b's VALUE into a, doesn't reassign reference
```

- References are particularly useful for specifying function arguments. For example, `void sort(vector<double>& v);`, by using a reference, we ensure that for a call such as `sort(myVec)`, we do not copy the argument. It really is `myVec` that is sorted and not a copy of it.
- When we <mark>don't want to modify an argument but still don't want the cost of copying</mark>, we use a *const reference*, that is , a reference to a constant. Function taking const references are very common.

## The Null Pointer

- When we don't have an object to point to or if we need to represent the notion of "no object available" (e.g., for an end of a list), we give the pointer the value `nullptr`.
- It is often wise to check that a pointer argument actually points to something.
- In older code, `0` or `NULL` is typically used instead of `nullptr`. However, using `nullptr` eliminates potential confusion between integers (such as `0` or `NULL`) and pointers (such as `nullptr`).
- A test of a numeric value (e.g., `while(*ptr)`) is equivalent to comapring the value to `0` (that is, `while(*ptr != 0)`).
- A test of a pointer value (e.g., `if(ptr)`) is equivalent to comapring the value to `nullptr` (that is, `if(ptr != nullptr)`).
- There are no "null reference". A referenece must refer to a valid object.

## Tests

- Like a for-statement, an if-statement can introduce a variable and test it.
- A name declared in a condition is in scope on both branches of the if-statement.

```cpp
void doSomething(vector<int>& v) {
    if(auto n = v.size(); n!=0) {
        // we get here if n!=0 ...
    }
}
```

## Mapping to Hardware

### Assignment

- If we want different objects to refer to the same value, we must say so.

```cpp
int x = 2;
int y = 3;
int* xPtr = &x;
int* yPtr = &y;

xPtr = yPtr; // xPtr becomes &y, now xPtr==yPtr, so *xPtr==*yPtr
// Now both pointers point to y.
```

- Assignment to a reference doesn't change what the reference refers to but assigns to the referenced object.

```cpp
int x = 2;
int y = 3;
int& xRef = x;
int& yRef = y;

xRef = yRef; // read through yRef, write through xRef, x becoms 3
// Now xRef still points to x, yRef still points to y, but the value of x has changed to 3.
```

- To access the value pointed to by a pointer, you use `*`; <mark>that is implicitly done for a reference</mark>.

### Initialization

- We can not have an uninitialized reference.
- You can use both `{}` and `=` to initialize a reference but please don't let that confuse you.

```cpp
int& r {x}; // bind r to x (r refers to x)
int& r = x; // bind r to x (r refers to x), not any value copy
```
