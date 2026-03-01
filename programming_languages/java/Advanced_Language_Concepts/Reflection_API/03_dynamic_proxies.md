# Dynamic Proxies

Dynamic proxies in Java Reflection provide a way to create proxy objects at runtime that implement a specified list of interfaces. This is incredibly powerful for implementing cross-cutting concerns (like logging, security, transactions) without modifying the original code, a concept often seen in Aspect-Oriented Programming (AOP).

## 1. What is a Proxy?

A **proxy** is an object that acts as a substitute for another object (the "real" or "subject" object). It controls access to the subject and can add extra functionality before or after forwarding calls to the subject.

## 2. The `java.lang.reflect.Proxy` Class

The `Proxy` class provides static methods for creating dynamic proxy classes and instances.

## 3. The `java.lang.reflect.InvocationHandler` Interface

This is the core component for defining the proxy's behavior. When a method is called on the proxy instance, the `invoke()` method of its associated `InvocationHandler` is called.

### `InvocationHandler` Interface
```java
public interface InvocationHandler {
    public Object invoke(Object proxy, Method method, Object[] args)
        throws Throwable;
}
```
*   **`proxy`:** The proxy instance itself.
*   **`method`:** The `java.lang.reflect.Method` object corresponding to the method invoked on the proxy.
*   **`args`:** An array of `Object` containing the arguments passed to the proxy method.

## 4. Creating a Dynamic Proxy

The process involves these steps:
1.  **Define an Interface:** The subject class must implement an interface. The proxy will implement this same interface.
2.  **Create an `InvocationHandler`:** Implement the `InvocationHandler` interface, defining the logic that the proxy will execute.
3.  **Use `Proxy.newProxyInstance()`:** Call this static method to create the proxy instance.

### Example: Simple Logging Proxy

Let's say we have a `Calculator` interface and an implementation. We want to log every method call without changing `CalculatorImpl`.

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.util.Arrays;

// 1. Define an Interface
interface Calculator {
    int add(int a, int b);
    int subtract(int a, int b);
}

// Concrete Implementation (the "Subject")
class CalculatorImpl implements Calculator {
    @Override
    public int add(int a, int b) {
        return a + b;
    }

    @Override
    public int subtract(int a, int b) {
        return a - b;
    }
}

// 2. Create an InvocationHandler
class LoggingInvocationHandler implements InvocationHandler {
    private final Object target; // The actual object to delegate to

    public LoggingInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // Log before the method call
        System.out.println("Calling method: " + method.getName() + " with args: " + Arrays.toString(args));

        // Invoke the method on the target object
        Object result = method.invoke(target, args);

        // Log after the method call
        System.out.println("Method " + method.getName() + " returned: " + result);

        return result;
    }
}

public class DynamicProxyExample {
    public static void main(String[] args) {
        Calculator realCalculator = new CalculatorImpl();

        // 3. Use Proxy.newProxyInstance()
        // Parameters:
        //   - ClassLoader: ClassLoader of the target/proxy
        //   - interfaces: Array of interfaces the proxy should implement
        //   - InvocationHandler: The handler that defines the proxy's behavior
        Calculator proxyCalculator = (Calculator) Proxy.newProxyInstance(
            Calculator.class.getClassLoader(),       // ClassLoader
            new Class<?>[] { Calculator.class }, // Interfaces to implement
            new LoggingInvocationHandler(realCalculator) // Handler
        );

        // Now, call methods on the proxy.
        // The invoke() method of LoggingInvocationHandler will be called for each.
        int sum = proxyCalculator.add(10, 5);
        System.out.println("Sum from proxy: " + sum);

        int diff = proxyCalculator.subtract(10, 5);
        System.out.println("Difference from proxy: " + diff);
    }
}
// Expected Output:
// Calling method: add with args: [10, 5]
// Method add returned: 15
// Sum from proxy: 15
// Calling method: subtract with args: [10, 5]
// Method subtract returned: 5
// Difference from proxy: 5
```

## 5. Use Cases for Dynamic Proxies

*   **AOP (Aspect-Oriented Programming):** Frameworks like Spring AOP extensively use dynamic proxies (and other byte-code manipulation libraries like CGLIB) to inject cross-cutting concerns (e.g., transactions, security, logging) into business logic without modifying the core code.
*   **Mocking Frameworks:** Testing frameworks (e.g., Mockito) use proxies to create mock objects for unit testing.
*   **Remote Method Invocation (RMI):** Proxies are used to represent remote objects locally.
*   **Security:** To restrict access to certain methods or objects.
*   **Lazy Loading:** To defer the creation or loading of an object until it's actually needed.

Dynamic proxies are a powerful, albeit advanced, feature of Java Reflection, enabling highly modular and extensible software architectures.