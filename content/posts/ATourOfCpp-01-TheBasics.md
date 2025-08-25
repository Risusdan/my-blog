+++
date = '2025-08-22T10:00:00-07:00'
draft = false
title = 'A Tour of C++ 01: The Basics'
categories = ['A Tour of C++']
tags = ['C++']
+++

# A Tour of C++ 01: The Basics

![](/images/IMG_7844.jpeg)

Terms:

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
