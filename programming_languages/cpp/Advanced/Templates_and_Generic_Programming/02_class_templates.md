# Class Templates

Just like function templates, you can also have class templates. A class template is a blueprint for creating a class. The STL containers like `std::vector`, `std::list`, and `std::map` are all class templates.

## Syntax

```cpp
template <typename T>
class MyContainer {
private:
    T data;
public:
    MyContainer(T d) : data(d) {}
    T getData() const { return data; }
};
```

## Instantiation

To use a class template, you must explicitly specify the template argument.

```cpp
MyContainer<int> c1(10);
std::cout << c1.getData() << std::endl; // 10

MyContainer<std::string> c2("hello");
std::cout << c2.getData() << std::endl; // "hello"
```

## Example: A simple Pair class

Let's create a class template that can store a pair of values of any type.

```cpp
#include <iostream>

template <typename T, typename U>
class Pair {
private:
    T first;
    U second;
public:
    Pair(T f, U s) : first(f), second(s) {}

    T getFirst() const { return first; }
    U getSecond() const { return second; }
};

int main() {
    Pair<int, double> p1(10, 3.14);
    std::cout << "p1: " << p1.getFirst() << ", " << p1.getSecond() << std::endl;

    Pair<std::string, int> p2("age", 30);
    std::cout << "p2: " << p2.getFirst() << ", " << p2.getSecond() << std::endl;

    return 0;
}
```
This is essentially a simplified version of `std::pair`.

## Member Functions of a Class Template

When you define a member function outside of the class template definition, you need to provide the template prefix again.

```cpp
template <typename T>
class MyClass {
public:
    void myMethod();
};

template <typename T>
void MyClass<T>::myMethod() {
    // ...
}
```

## Non-Type Template Parameters

Template parameters don't have to be types. They can also be non-type parameters, such as `int`s. A common use case for this is to have a fixed-size array class.

```cpp
#include <iostream>
#include <array> // for comparison

template <typename T, int Size>
class FixedArray {
private:
    T data[Size];
public:
    int getSize() const { return Size; }
    // ... other methods like operator[]
};

int main() {
    FixedArray<int, 10> arr;
    std::cout << "Size: " << arr.getSize() << std::endl;

    // This is similar to std::array
    std::array<int, 10> std_arr;
    std::cout << "Size: " << std_arr.size() << std::endl;

    return 0;
}
```
Class templates are a fundamental part of the C++ language and are used extensively in the Standard Library.
