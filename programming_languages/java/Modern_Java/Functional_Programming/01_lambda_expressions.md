# Lambda Expressions: Under the Hood

Lambda expressions are more than just syntactic sugar for anonymous classes. Their implementation in the JVM allows for significant optimization and performance benefits.

## 1. Syntax vs. Implementation

### The Anonymous Class Approach (Legacy)
When you compile an anonymous inner class:
```java
Runnable r = new Runnable() { public void run() { ... } };
```
The compiler generates a separate physical file on disk: `MyClass$1.class`.
*   **Cost:** Extra disk IO, extra class loading, memory metadata usage in Metaspace.

### The Lambda Approach (Modern)
```java
Runnable r = () -> { ... };
```
The compiler does **NOT** generate a separate class file. Instead, it generates an **`invokedynamic`** instruction in the bytecode.
*   **Mechanism:** At runtime, the first time the lambda is encountered, the JVM calls a "Bootstrap Method" (LambdaMetafactory). This generates the class *in memory* on the fly.
*   **Benefit:** Smaller JARs, faster startup (less IO), and the JVM can optimize the strategy in future versions without you recompiling your code.

## 2. Variable Capture (Closures)

Lambdas can "capture" variables from their enclosing scope.

```java
int factor = 10;
Function<Integer, Integer> f = n -> n * factor; // Capturing 'factor'
```

### The Cost of Capture
*   **Non-Capturing Lambda:** If your lambda uses *no* external variables (`x -> x * 2`), the JVM creates a **Singleton** instance. It allocates memory *once*. It is extremely fast.
*   **Capturing Lambda:** If your lambda uses external variables, the JVM must create a **new instance** every time the lambda definition is executed to store the captured state (`factor`).
*   **Performance Tip:** Prefer non-capturing lambdas in hot paths to avoid allocation pressure.

### "Effectively Final"
Why must captured variables be final?
*   **Concurrency:** If local variables were mutable, you could have complex race conditions where the lambda sees a changing variable from a stack frame that might have already disappeared (if the lambda runs in a different thread). Java enforces immutability to ensure thread safety.

## 3. Scope and `this`
In an anonymous class, `this` refers to the anonymous class instance itself.
In a lambda, `this` refers to the **enclosing class** instance. Lambdas do not have their own identity (shadowing) in the same way.
