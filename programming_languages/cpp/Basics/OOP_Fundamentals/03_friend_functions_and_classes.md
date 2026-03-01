# Friend Functions and Classes

The `friend` keyword grants access to a class's private and protected members. It's a controlled way to break encapsulation when needed.

## What Does Friend Do?

A friend can access **all** members (private, protected, public) of the class that declared it as a friend.

```cpp
class Secret {
private:
    int privateData = 42;

    friend void revealSecret(const Secret& s);  // Grant access
};

void revealSecret(const Secret& s) {
    std::cout << s.privateData << std::endl;  // OK! We're a friend
}

int main() {
    Secret s;
    revealSecret(s);  // 42
    // std::cout << s.privateData;  // Error! Not a friend
}
```

## Types of Friends

### 1. Friend Functions

```cpp
class Box {
private:
    double width, height, depth;

public:
    Box(double w, double h, double d) : width(w), height(h), depth(d) {}

    // Friend function declaration
    friend double calculateVolume(const Box& b);
};

// Friend function definition (NOT a member function!)
double calculateVolume(const Box& b) {
    return b.width * b.height * b.depth;  // Access private members
}
```

### 2. Friend Classes

```cpp
class Engine {
private:
    int horsepower = 300;
    void start() { std::cout << "Engine started\n"; }

    friend class Car;  // Car can access all Engine members
};

class Car {
    Engine engine;

public:
    void displayPower() {
        std::cout << "HP: " << engine.horsepower << std::endl;  // OK
    }

    void startEngine() {
        engine.start();  // OK - accessing private method
    }
};
```

### 3. Friend Member Functions

```cpp
class B;  // Forward declaration

class A {
public:
    void showB(const B& b);
};

class B {
private:
    int secret = 99;

    friend void A::showB(const B& b);  // Only A::showB is a friend
};

void A::showB(const B& b) {
    std::cout << b.secret << std::endl;  // OK
}
```

## Common Use Cases

### 1. Operator Overloading

The most common use - `operator<<` can't be a member:

```cpp
class Complex {
private:
    double real, imag;

public:
    Complex(double r, double i) : real(r), imag(i) {}

    // Friend for stream output
    friend std::ostream& operator<<(std::ostream& os, const Complex& c);

    // Friend for symmetric + operation
    friend Complex operator+(const Complex& a, const Complex& b);
};

std::ostream& operator<<(std::ostream& os, const Complex& c) {
    os << c.real << " + " << c.imag << "i";
    return os;
}

Complex operator+(const Complex& a, const Complex& b) {
    return Complex(a.real + b.real, a.imag + b.imag);
}
```

### 2. Factory Classes

```cpp
class Widget {
private:
    Widget() {}  // Private constructor
    int id;

    friend class WidgetFactory;
};

class WidgetFactory {
public:
    static Widget create(int id) {
        Widget w;     // Can call private constructor
        w.id = id;    // Can access private member
        return w;
    }
};
```

### 3. Testing Private Methods

```cpp
class MyClass {
private:
    int internalComputation(int x) { return x * 2; }

    friend class MyClassTest;  // Test class can access internals
};

class MyClassTest {
public:
    static void testInternal() {
        MyClass obj;
        assert(obj.internalComputation(5) == 10);
    }
};
```

### 4. Iterator Access to Container

```cpp
template<typename T>
class Container {
private:
    T* data;
    size_t size;

public:
    class Iterator {
        T* ptr;
    public:
        Iterator(T* p) : ptr(p) {}
        // ...
    };

    friend class Iterator;  // Iterator can access Container's private members

    Iterator begin() { return Iterator(data); }
    Iterator end() { return Iterator(data + size); }
};
```

## Important Properties

### Friendship is NOT:

1. **Symmetric**: If A is B's friend, B is NOT automatically A's friend
```cpp
class A {
    friend class B;  // B can access A's private
};

class B {
    // A cannot access B's private (unless B declares A as friend)
};
```

2. **Transitive**: Friend of friend is NOT a friend
```cpp
class A {
    friend class B;
};

class B {
    friend class C;
};
// C is NOT a friend of A
```

3. **Inherited**: Friends are not inherited
```cpp
class Base {
    friend class Helper;
};

class Derived : public Base {
    // Helper is NOT a friend of Derived
};
```

## Friend vs Public Getter

```cpp
// Using friend
class Data {
private:
    int value;
    friend void process(const Data& d);
};

void process(const Data& d) {
    std::cout << d.value;  // Direct access
}

// Using getter
class Data {
    int value;
public:
    int getValue() const { return value; }
};

void process(const Data& d) {
    std::cout << d.getValue();  // Through getter
}
```

| Approach | Pros | Cons |
|----------|------|------|
| Friend | Direct access, can be more efficient | Tighter coupling |
| Getter | Maintains encapsulation | Extra function call, exposes to ALL |

## When to Use Friend

**Good uses:**
- Operator overloading (especially `<<`, `>>`, binary operators)
- Factory patterns
- Closely related classes that share implementation details
- Testing frameworks

**Avoid when:**
- You can use public methods instead
- It creates unnecessary coupling
- Just for convenience (lazy encapsulation)

## Key Takeaways

- Friend grants access to private/protected members
- Friendship is declared by the class granting access
- Not symmetric, transitive, or inherited
- Most common use: operator overloading
- Use sparingly—it breaks encapsulation

## Common Interview Questions

> [!question]- Is friendship inherited?
> No. If class `Base` declares `Helper` as friend, derived classes don't automatically have `Helper` as a friend.

> [!question]- Can friend functions be virtual?
> No. Friend functions are not members, so they can't be virtual. Only member functions can be virtual.

> [!question]- Why is `operator<<` usually a friend?
> Because the left operand is `std::ostream`, not our class. We can't add members to `ostream`, so we use a friend non-member function.

> [!question]- Does friend violate encapsulation?
> It's a controlled violation. The class explicitly chooses who can access its internals. It's better than making members public.

## Related Topics

- [[../../Intermediate/Operator_Overloading/03_stream_operators|Stream Operators]]
- [[02_static_members|Static Members]]
