# Constructors

A constructor is a special member function that is automatically called when an object of a class is created. It is used to initialize the object's attributes.

## Properties of Constructors

*   A constructor has the same name as the class.
*   A constructor does not have a return type.
*   A constructor can be overloaded.

## Default Constructor

A default constructor is a constructor that takes no arguments. If you do not define any constructors, the compiler will generate a default constructor for you.

```cpp
class MyClass {
public:
    MyClass() {
        std::cout << "Constructor called!" << std::endl;
    }
};

int main() {
    MyClass obj; // an object of MyClass is created, and the constructor is called
    return 0;
}
```

## Parameterized Constructor

A constructor that takes arguments is called a parameterized constructor.

```cpp
class Dog {
public:
    std::string name;
    int age;

    Dog(std::string n, int a) {
        name = n;
        age = a;
    }

    void print_info() {
        std::cout << "Name: " << name << ", Age: " << age << std::endl;
    }
};

int main() {
    Dog myDog("Buddy", 3);
    myDog.print_info();
    return 0;
}
```

## Initializer Lists

Initializer lists are a more efficient way to initialize member variables. They are used in the constructor's definition.

```cpp
class Dog {
public:
    std::string name;
    int age;

    Dog(std::string n, int a) : name(n), age(a) {
        // constructor body can be empty
    }
};
```

It is recommended to use initializer lists for a few reasons:

*   **Efficiency:** For class member variables, it avoids an extra step of creating the object and then assigning to it.
*   **const members:** `const` member variables can only be initialized using an initializer list.
*   **Reference members:** Reference member variables must be initialized using an initializer list.

## Copy Constructor

A copy constructor is a constructor that creates an object by initializing it with an object of the same class, which has been created previously.

```cpp
class MyClass {
public:
    int x;
    MyClass(int val) : x(val) {}
    MyClass(const MyClass& other) {
        x = other.x;
    }
};

int main() {
    MyClass obj1(10);
    MyClass obj2 = obj1; // copy constructor is called
    MyClass obj3(obj1);  // copy constructor is also called
}
```
If you don't define a copy constructor, the compiler will create a default one for you that performs a member-wise copy.
