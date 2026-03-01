# Encapsulation

Encapsulation is the bundling of data (attributes) and methods that operate on the data into a single unit (a class). It also involves restricting access to some of the object's components, which is known as **data hiding**.

## How to achieve Encapsulation?

In C++, encapsulation is achieved by:

1.  Making the member variables of a class `private`.
2.  Providing `public` member functions (known as **getters** and **setters**) to access and modify the member variables.

### Example

```cpp
#include <iostream>
#include <string>

class Employee {
private:
    int salary;

public:
    // Setter
    void setSalary(int s) {
        if (s > 0) {
            salary = s;
        } else {
            salary = 0;
        }
    }

    // Getter
    int getSalary() {
        return salary;
    }
};

int main() {
    Employee emp;
    // emp.salary = 50000; // Error: salary is private

    emp.setSalary(50000);
    std::cout << "Salary: " << emp.getSalary() << std::endl;

    emp.setSalary(-1000); // invalid salary
    std::cout << "Salary: " << emp.getSalary() << std::endl;

    return 0;
}
```

## Benefits of Encapsulation

*   **Data Hiding:** The internal state of an object is hidden from the outside. This prevents other parts of the program from modifying the data in unexpected ways.
*   **Increased Flexibility:** You can change the implementation of the class without affecting the code that uses it. For example, you could change the type of the `salary` variable, and you would only need to update the getter and setter methods.
*   **Validation:** You can add validation logic to the setter methods to ensure that the data is in a valid state. In the example above, we prevent the salary from being set to a negative value.
*   **Maintainability:** Encapsulation makes the code easier to understand and maintain because the data and the methods that operate on it are grouped together.
