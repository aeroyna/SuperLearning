# Setup and Hello World: Practical Foundations

This chapter bridges the gap from theoretical understanding of the JVM to the practical steps of setting up your development environment and running your first Java program. We will demystify the installation process and perform a deep dissection of the "Hello World" example.

## 1. Choosing a JDK Distribution

The Java Development Kit (JDK) is where all the tools for compiling and running Java reside. Since Oracle made changes to its JDK licensing, the landscape of JDK distributions has diversified.

### OpenJDK vs. Oracle JDK
*   **OpenJDK:** This is the upstream, open-source reference implementation of Java. Most modern JDK distributions are based on OpenJDK. It's free and open-source.
*   **Oracle JDK:** Oracle's branded distribution. Free for development and personal use, but requires a commercial license for production use in enterprises.
*   **Other Vendors:** Many companies provide their own builds of OpenJDK with varying levels of support, optimizations, and pricing models. These are generally safe and often recommended for production.

### Recommended Distributions for Development & Production
1.  **Eclipse Temurin (Adoptium):** Highly recommended, community-driven, stable, and widely adopted for production. Builds are available for various platforms and architectures.
2.  **Amazon Corretto:** Amazon's free, production-ready distribution of OpenJDK. Ideal if you're deploying on AWS but also works universally.
3.  **Azul Zulu:** Another popular choice, offering builds with different garbage collectors and support options.

### Choosing the Right Version
*   **LTS (Long-Term Support) Releases:** These are maintained for several years and are the standard for stable production environments.
    *   **Java 21 (LTS):** The latest LTS, recommended for new projects.
    *   **Java 17 (LTS):** Still very common in enterprise, excellent stability.
    *   **Java 8 (LTS):** Older, but some legacy systems are still tied to it. Avoid for new development if possible.
*   **Feature Releases:** Versions between LTS releases (e.g., Java 18-20, Java 22) are released every 6 months and are supported only for 6 months. Use these for experimenting with new features or if you can update frequently.

## 2. Installation & Environment Variables: The Path to Execution

Properly setting up your environment variables is crucial for the Java tools to be accessible from any command line.

### `JAVA_HOME`
This environment variable points to the root directory of your JDK installation. It's widely used by build tools (Maven, Gradle), application servers (Tomcat, JBoss), and many Java applications to locate the correct JDK.
*   **Example (Windows):** `C:\Program Files\Java\jdk-21`
*   **Example (Linux/macOS):** `/usr/lib/jvm/jdk-21`

### `Path` (or `PATH` on Linux/macOS)
This system variable tells your operating system where to look for executable programs when you type a command (like `java` or `javac`) without specifying its full path.
*   You typically add `%JAVA_HOME%\bin` (Windows) or `$JAVA_HOME/bin` (Linux/macOS) to your `Path`.

### Installation Steps (General)
1.  **Download:** Get the appropriate JDK installer for your OS (e.g., from adoptium.net).
2.  **Install:** Follow the installer instructions.
3.  **Set `JAVA_HOME`:** Configure `JAVA_HOME` to point to the installation directory.
4.  **Update `Path`:** Add `%JAVA_HOME%\bin` to your system's `Path` variable.

### Verification
After installation, open a **new** terminal or command prompt and run:
```bash
java -version    # Shows installed JRE version
javac -version   # Shows installed JDK compiler version
echo %JAVA_HOME% # Windows
echo $JAVA_HOME  # Linux/macOS
```
If these commands return the expected versions and paths, your setup is correct.

## 3. Hello World: Deconstructing Your First Program

The "Hello World" program is the traditional first step in any language. Let's dissect its structure.

```java
// HelloWorld.java
public class HelloWorld { // 1. Class Declaration
    public static void main(String[] args) { // 2. Main Method - The Entry Point
        System.out.println("Hello, Java!"); // 3. The Print Statement
    }
}
```

### Line-by-Line Deep Dissection

#### 1. `public class HelloWorld`
*   **`class`:** This keyword declares a new class. In Java, all executable code must reside within a class. It's the blueprint for objects.
*   **`HelloWorld`:** This is the name of your class.
    *   **Rule:** If a class is declared `public` (as `HelloWorld` is), its filename **must** exactly match the class name, including case (e.g., `HelloWorld.java`). This is a strict compiler rule.
    *   **Convention:** Class names should always be in **PascalCase** (first letter of each word capitalized).
*   **`public`:** This is an **access modifier**. It means this class is visible and accessible from any other class.

#### 2. `public static void main(String[] args)`
This specific method signature is the universal **entry point** for all standalone Java applications. When you run `java MyClass`, the JVM searches for and executes this `main` method.
*   **`public`:** The JVM, which is external to your program, needs to be able to call this method. Hence, it must be `public`.
*   **`static`:** This keyword means the `main` method belongs to the `HelloWorld` *class* itself, not to any specific *object* of `HelloWorld`. The JVM doesn't need to create an instance (`new HelloWorld()`) to call `main`. This is why `static` methods cannot directly access non-static (instance) variables or methods without an object reference.
*   **`void`:** This is the method's **return type**. `void` means the method does not return any value after its execution.
*   **`main`:** This is a special, predefined method name that the JVM looks for as the starting point of the program.
*   **`String[] args`:** This is the **parameter list**. It's an array of `String` objects, allowing you to pass command-line arguments to your program when it starts. Each element in the `args` array represents one argument.
    *   Example: `java HelloWorld arg1 arg2` would make `args[0]` = "arg1" and `args[1]` = "arg2".

#### 3. `System.out.println("Hello, Java!");`
This line performs the actual output to the console.
*   **`System`:** A `final` class in the `java.lang` package (implicitly imported). It provides access to system resources.
*   **`out`:** A `static` `final` field within the `System` class, which is an instance of `PrintStream`. It represents the standard output stream (usually your console).
*   **`println()`:** A method of the `PrintStream` class. It prints the given string to the output stream and then appends a new line character (`\n`).

#### 4. Braces and Semicolons
*   **`{ ... }`:** Curly braces define **blocks of code** or scopes. The `main` method's code is enclosed in the `main` method's block, which is itself inside the `HelloWorld` class's block.
*   **`;`:** Every complete statement in Java (like variable declarations, method calls) must be terminated by a semicolon.

## 4. Running the Code: Compilation vs. Execution

### The Traditional Two-Step Process (Pre-Java 11)
This process clearly separates compilation from execution, demonstrating the role of `javac` and `java` commands.

1.  **Compile the Source Code (`javac`):**
    ```bash
    # From your terminal, navigate to the directory containing HelloWorld.java
    javac HelloWorld.java
    ```
    *   **`javac` (Java Compiler):** Reads your human-readable Java source code (`.java` files).
    *   **Output:** If successful, `javac` produces a `HelloWorld.class` file in the same directory. This `.class` file contains **Java Bytecode**.
    *   **Errors:** If there are syntax errors, `javac` will report them (compile-time errors) and will *not* produce a `.class` file.

2.  **Execute the Bytecode (`java`):**
    ```bash
    # From your terminal, in the same directory
    java HelloWorld
    ```
    *   **`java` (Java Application Launcher):** Invokes the JVM.
    *   The JVM loads the `HelloWorld.class` file.
    *   It verifies the bytecode.
    *   It then finds and executes the `public static void main(String[] args)` method within that class.
    *   **Important:** You specify the class name (`HelloWorld`), not the filename (`HelloWorld.java` or `HelloWorld.class`).

### The Modern One-Step Process (Java 11+)
For simple, single-file source code programs, Java 11 introduced a convenience feature.

```bash
# From your terminal, in the directory containing HelloWorld.java
java HelloWorld.java
```
*   **Process:** When you use `java` directly on a `.java` source file, the JVM implicitly compiles the source code in memory (not creating a `.class` file on disk by default) and then executes it.
*   **Benefit:** Great for quick scripts, learning, and reducing friction during development.
*   **Limitation:** Primarily for single-file source code programs; more complex projects with multiple source files still require explicit compilation via `javac` or a build tool.

This comprehensive setup process and deep understanding of "Hello World" provide the essential foundation for building any Java application.

---

### Links to Topics:
*   [History & Philosophy](01_java_history_and_philosophy.md)
*   [JVM, JDK, JRE Architecture](02_jvm_jdk_jre_architecture.md)
*   [Setup and Hello World](03_setup_and_hello_world.md)
*   [Compilation and Execution Process](04_compilation_and_execution_process.md)
