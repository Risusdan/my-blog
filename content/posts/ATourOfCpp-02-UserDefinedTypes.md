+++ 
draft = false
date = 2025-09-03T07:33:53+08:00
title = "A Tour of C++ 02: User-Defined Types"
categories = ['A Tour of C++']
tags = ['C++']
+++

## Table of Contents

- [Table of Contents](#table-of-contents)
- [Introduction](#introduction)
- [Structures](#structures)
- [Classes](#classes)
- [Enumerations](#enumerations)
- [Unions](#unions)

## Introduction

- Types built out of other using C++'s abstraction mechanisms are called *user-defined types*.

> Abstraction mechanisms are language features that let you define and manipulate complex types and behaviors by building higher-level concepts from simpler ones.

- User-defined typs are often preferred over built-in types because they are easier to use, less error-prone, and typically as efficent for what they do as direct use of built-in types, or even more efficient.

## Structures

- To defined a new type `Vector` which is similar to the standard-library `vector`, first of all we define a structure.

```cpp
struct Vector {
    double* element;
    int size;
}

void vectorInit(Vector& v, int s) {
    v.element = new double[s];
    v.size = s;
}

Vector v;
vectorInit(v, 5);
```

- We use *.(dot)* to access `struct` members through a name and through a reference.
- We use *->* to access `struct` members through a pointer.

```cpp
void f(Vector v, Vector& rv, Vector* pv) {
    int i1 = v.size;
    int i2 = rv.size;
    int i3 = pv->size;
}
```

## Classes

- A class has a set of *members*, which can be data, function, or type members.

> Type members are types declared within a class, such as nested classes/structs or typedefs/using aliases. They help group related types under the class’s scope and can be controlled via access specifiers.

- The interface of a class is defined by its `public` members, and its `private` members are accessible only through that interface.
- Conventionally we place the `public` declarations first and the `private` declarations later, except when we want to emphasize the representation.

```cpp
class Vector {
public:
    // construct a Vector
    Vector(int s) : element_{new doubles[s]}, size_{s} {}
    // element access
    double& operator[](int i) {
        return element_[i];
    }
    // get size
    int size() {
        return size_;
    }
private:
    double* element_;
    int size_;
}
```

- A member function with the same name as its class is called a *constructor*, that is, a function used to construct object of a class.
- The constructor initializes the members using a *member initializer list*. For example, in `element_{new doubles[s]}, size_{s}`, we first initialize `element_` with a pointer to `s` elements of type double obtained from the free store. Then, we initialize `size_` to `s`.
- Access to elements is provided by a *subscript function*, called `operator[]`.
- Obviously, error handling is missing, and we did not provide a mechanism to use *destructor* to "give back" the array of doubles acquired by `new`.
- There is no fundamental difference between a *struct* and a *class*; <mark>a *struct* is simply a *class* with members public by default</mark>.

## Enumerations

- C++ supports a simple form of user-defined type for which we can enuerate the values.

```cpp
enum class Color {red, blue, green};
enum class TrafficLight {green, yellow, red};

Color col = Color::red;
TrafficLight light = TrafficLight::red;
```

- Note that while using enum class, the enumerators (e.g., red) are in the scope of their enum class, so that they help prevent acciential misuse of constants.

```cpp
auto x1 = Color::red;
Color y2 = TrafficLight::red;
Color z3 = Color::red;
int i = Color::red; // error: Color::red is not an int
Color c = 2; // error: 2 is not a Color
```

- It is allowed to initialize an enum class with a value from its underlying type (by default, that is `int`).

```cpp
Color x = Color{5};
Color y {6};
```

- We can explicitly convert an enum value to its underlying type.

```cpp
int x = int(Color::red);
```

- We can define operators foran enumeration.

```cpp
TrafficLight& operator++(TrafficLight& t) {
    using enum TrafficLight;

    switch(t) {
        case green: return t = yellow;
        case yellow: return t = red;
        case red: return t = green;
    }
}

auto signal = TrafficLight::red;
TrafficLight next = ++signal;
```

- If you don't want to explicitly qualify enumerator names and want enumerator values to be ints, you can remove the "class" from "enum class" to get a "plain" enum.
- The enumerators from a "plain" enum are entered into the same scope as the name of their enum and implicitly convert to their integer values.

```cpp
enum Color {red, green, blue};
int col = green; // col gets value 1
```

## Unions

- A union is a struct in which all members are allocated at the same address so that the union occupies only as much space as its largest member.

```cpp
union Value {
    Node* p;
    int i;
}
```

> `Value::p` and `Value::i` are placed at the same address of memory of each `Value` object.

- The programmer must keep track of which kind of value is hold by a union.

```cpp
union Value {
    Node* p;
    int i;
}

struct Entry {
    string Name;
    Type t;
    Value v;
}

void f(Entry* pe)
{
    if(pe->t == Type::num)
    {
        cout << pe->v.i;
    }
}
```

- A standard library type called `variant` can be used to eliminate most direct uses of unions.
- A variant stores a value of one of a set of alternative types.

```cpp
struct Entry {
    string Name;
    variant<Node*, int> v;
}

void f(Entry* pe)
{
    if(holds_alternative<int>(pe->v)) // Does `*pe` holds an int?
    {
        cout << pe->v.i;
    }
}
```
