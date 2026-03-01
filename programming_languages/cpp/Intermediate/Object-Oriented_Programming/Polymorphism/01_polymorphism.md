# Polymorphism

Polymorphism is a core concept of object-oriented programming that means "many forms". It allows objects of different classes to be treated as objects of a common base class.

There are two types of polymorphism in C++:

1.  **Compile-time Polymorphism (Static Polymorphism):** This is achieved through function overloading and operator overloading. The compiler decides which function to call at compile time.

2.  **Runtime Polymorphism (Dynamic Polymorphism):** This is achieved through virtual functions. The compiler decides which function to call at runtime. This is the more powerful type of polymorphism.

## Runtime Polymorphism

Runtime polymorphism is achieved by using a base class pointer to refer to a derived class object. When a member function is called through the base class pointer, the compiler determines which version of the function to call based on the actual type of the object being pointed to.

### Example

```cpp
#include <iostream>

class Animal {
public:
    void speak() {
        std::cout << "Animal speaks" << std::endl;
    }
};

class Dog : public Animal {
public:
    void speak() {
        std::cout << "Dog barks" << std::endl;
    }
};

class Cat : public Animal {
public:
    void speak() {
        std::cout << "Cat meows" << std::endl;
    }
};

int main() {
    Animal* a1 = new Dog();
    Animal* a2 = new Cat();

    a1->speak(); // Expected: "Dog barks", Actual: "Animal speaks"
    a2->speak(); // Expected: "Cat meows", Actual: "Animal speaks"

    delete a1;
    delete a2;
    return 0;
}
```

In the example above, even though `a1` points to a `Dog` object, `a1->speak()` calls the `speak()` method of the `Animal` class. This is because the decision of which function to call is made at compile time based on the type of the pointer (`Animal*`).

To achieve runtime polymorphism, we need to use **virtual functions**.
