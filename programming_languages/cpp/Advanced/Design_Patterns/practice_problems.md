# Practice Problems: Design Patterns

## Problem 1: Simple Logger Singleton

Implement a thread-safe singleton logger class. It should have a `log` method that writes a message to the console. Demonstrate that it is a singleton by getting the instance from two different places and showing that they are the same object.

### Solution

```cpp
#include <iostream>
#include <string>
#include <mutex>

class Logger {
private:
    Logger() {} // Private constructor
    static std::mutex mtx;

public:
    // Delete copy/move operations
    Logger(const Logger&) = delete;
    Logger& operator=(const Logger&) = delete;
    Logger(Logger&&) = delete;
    Logger& operator=(Logger&&) = delete;

    static Logger& getInstance() {
        static Logger instance;
        return instance;
    }

    void log(const std::string& message) {
        // Lock for thread safety, although cout is already thread-safe in C++11
        std::lock_guard<std::mutex> lock(mtx);
        std::cout << "[LOG] " << message << std::endl;
    }
};

std::mutex Logger::mtx;

void function1() {
    Logger::getInstance().log("Message from function1.");
}

void function2() {
    Logger::getInstance().log("Message from function2.");
}

int main() {
    std::cout << "Address of logger in main: " << &Logger::getInstance() << std::endl;
    
    std::thread t1(function1);
    std::thread t2(function2);

    t1.join();
    t2.join();

    return 0;
}
```

## Problem 2: Document Exporter (Factory Method)

You are building a document editor that can export documents in different formats (e.g., PDF, HTML). Use the Factory Method pattern to create an exporter system.

*   Create a `DocumentExporter` abstract class with a `createExporter()` factory method.
*   Create concrete creator classes like `PdfExporterCreator` and `HtmlExporterCreator`.
*   Create a `Exporter` interface with an `export` method.
*   Create concrete products like `PdfExporter` and `HtmlExporter`.

### Solution

```cpp
#include <iostream>
#include <string>
#include <memory>

// Product interface
class Exporter {
public:
    virtual ~Exporter() = default;
    virtual void exportDoc(const std::string& content) = 0;
};

// Concrete Products
class PdfExporter : public Exporter {
public:
    void exportDoc(const std::string& content) override {
        std::cout << "Exporting to PDF: " << content << std::endl;
    }
};

class HtmlExporter : public Exporter {
public:
    void exportDoc(const std::string& content) override {
        std::cout << "Exporting to HTML: " << content << std::endl;
    }
};

// Creator (Factory)
class DocumentExporter {
public:
    virtual ~DocumentExporter() = default;
    virtual std::unique_ptr<Exporter> createExporter() = 0;

    void exportDocument(const std::string& content) {
        auto exporter = createExporter();
        exporter->exportDoc(content);
    }
};

// Concrete Creators
class PdfExporterCreator : public DocumentExporter {
public:
    std::unique_ptr<Exporter> createExporter() override {
        return std::make_unique<PdfExporter>();
    }
};

class HtmlExporterCreator : public DocumentExporter {
public:
    std::unique_ptr<Exporter> createExporter() override {
        return std::make_unique<HtmlExporter>();
    }
};


int main() {
    std::unique_ptr<DocumentExporter> creator;
    std::string format = "pdf"; // This could come from user input

    if (format == "pdf") {
        creator = std::make_unique<PdfExporterCreator>();
    } else {
        creator = std::make_unique<HtmlExporterCreator>();
    }

    creator->exportDocument("This is the document content.");

    return 0;
}
```

## Problem 3: Coffee with Add-ons (Decorator)

You are creating an app for a coffee shop. A coffee can have several add-ons (milk, sugar, etc.), and each add-on has a cost. Use the Decorator pattern to calculate the cost of a coffee with various add-ons.

*   Create a `Coffee` interface with a `cost()` method.
*   Create a `SimpleCoffee` concrete component.
*   Create a `CoffeeDecorator` base class.
*   Create concrete decorators like `WithMilk` and `WithSugar`.

### Solution

```cpp
#include <iostream>
#include <string>
#include <memory>

// Component
class Coffee {
public:
    virtual ~Coffee() = default;
    virtual double cost() = 0;
};

// Concrete Component
class SimpleCoffee : public Coffee {
public:
    double cost() override {
        return 5.0;
    }
};

// Decorator
class CoffeeDecorator : public Coffee {
protected:
    std::unique_ptr<Coffee> decoratedCoffee;
public:
    CoffeeDecorator(std::unique_ptr<Coffee> coffee) : decoratedCoffee(std::move(coffee)) {}
    double cost() override {
        return decoratedCoffee->cost();
    }
};

// Concrete Decorators
class WithMilk : public CoffeeDecorator {
public:
    WithMilk(std::unique_ptr<Coffee> coffee) : CoffeeDecorator(std::move(coffee)) {}
    double cost() override {
        return CoffeeDecorator::cost() + 0.5;
    }
};

class WithSugar : public CoffeeDecorator {
public:
    WithSugar(std::unique_ptr<Coffee> coffee) : CoffeeDecorator(std::move(coffee)) {}
    double cost() override {
        return CoffeeDecorator::cost() + 0.2;
    }
};

int main() {
    std::unique_ptr<Coffee> myCoffee = std::make_unique<SimpleCoffee>();
    std::cout << "Cost of simple coffee: " << myCoffee->cost() << std::endl;

    myCoffee = std::make_unique<WithMilk>(std::move(myCoffee));
    std::cout << "Cost with milk: " << myCoffee->cost() << std::endl;

    myCoffee = std::make_unique<WithSugar>(std::move(myCoffee));
    std::cout << "Cost with milk and sugar: " << myCoffee->cost() << std::endl;

    return 0;
}
```
