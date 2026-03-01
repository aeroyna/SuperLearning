# Practice Problems: Object-Oriented Programming

## Problem 1: Bank Account

Create a `BankAccount` class that has the following attributes: `accountNumber` (string) and `balance` (double).

*   The `balance` should be private.
*   Provide a constructor to initialize the account number and balance.
*   Provide `deposit` and `withdraw` methods. The `withdraw` method should not allow the balance to go below zero.
*   Provide a `getBalance` method.

### Solution

```cpp
#include <iostream>
#include <string>

class BankAccount {
private:
    std::string accountNumber;
    double balance;

public:
    BankAccount(std::string accNum, double initialBalance)
        : accountNumber(accNum), balance(initialBalance) {}

    void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }

    void withdraw(double amount) {
        if (amount > 0 && balance >= amount) {
            balance -= amount;
        } else {
            std::cout << "Insufficient funds or invalid amount." << std::endl;
        }
    }

    double getBalance() {
        return balance;
    }

    std::string getAccountNumber() {
        return accountNumber;
    }
};

int main() {
    BankAccount myAccount("123456789", 1000.0);
    std::cout << "Initial balance: " << myAccount.getBalance() << std::endl;

    myAccount.deposit(500.0);
    std::cout << "Balance after deposit: " << myAccount.getBalance() << std::endl;

    myAccount.withdraw(200.0);
    std::cout << "Balance after withdrawal: " << myAccount.getBalance() << std::endl;

    myAccount.withdraw(1500.0); // should fail
    std::cout << "Final balance: " << myAccount.getBalance() << std::endl;

    return 0;
}
```

## Problem 2: Shapes (Polymorphism)

Create an abstract class `Shape` with a pure virtual function `getArea()`.
Create two derived classes, `Rectangle` and `Circle`, that implement the `getArea()` function.
In your `main` function, create an array of `Shape` pointers and store pointers to `Rectangle` and `Circle` objects. Iterate through the array and print the area of each shape.

### Solution

```cpp
#include <iostream>
#include <vector>

// Abstract base class
class Shape {
public:
    virtual double getArea() = 0;
    virtual ~Shape() {} // Virtual destructor
};

class Rectangle : public Shape {
private:
    double width, height;
public:
    Rectangle(double w, double h) : width(w), height(h) {}
    double getArea() override {
        return width * height;
    }
};

class Circle : public Shape {
private:
    double radius;
public:
    Circle(double r) : radius(r) {}
    double getArea() override {
        return 3.14159 * radius * radius;
    }
};

int main() {
    std::vector<Shape*> shapes;
    shapes.push_back(new Rectangle(5, 4));
    shapes.push_back(new Circle(3));
    shapes.push_back(new Rectangle(2, 8));

    for (Shape* shape : shapes) {
        std::cout << "Area: " << shape->getArea() << std::endl;
    }

    // Clean up the dynamically allocated memory
    for (Shape* shape : shapes) {
        delete shape;
    }
    shapes.clear();

    return 0;
}
```
This problem demonstrates polymorphism and the importance of virtual destructors.
