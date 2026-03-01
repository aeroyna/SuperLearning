# Setup and Hello World

## 1. Choosing a JDK Distribution
Unlike the old days where Oracle was the only option, today there are many high-quality, free distributions of the OpenJDK.

**Recommended Distributions:**
1.  **Eclipse Temurin (Adoptium):** Highly recommended, community-driven, stable.
2.  **Amazon Corretto:** Great for AWS environments, long-term support.
3.  **Azul Zulu:** Good for varying architecture support.
4.  **Oracle JDK:** Free for dev, but licensing can be complex for production.

**Version to Install:**
*   **Java 21 (LTS):** The current standard for new projects.
*   **Java 17 (LTS):** Still very common in enterprise.

## 2. Installation & Environment Variables

### Windows
1.  Download the `.msi` or `.zip`.
2.  **Set `JAVA_HOME`:**
    *   Right-click *This PC* -> *Properties* -> *Advanced System Settings* -> *Environment Variables*.
    *   New System Variable: `JAVA_HOME` -> `C:\Program Files\Eclipse Adoptium\jdk-21...`
3.  **Update `Path`:**
    *   Find `Path` variable -> Edit.
    *   Add `%JAVA_HOME%\bin`.
    *   *Why?* This lets you type `javac` and `java` in any command prompt window.

### macOS / Linux
Use a package manager like `sdkman` (highly recommended) or `brew`.

**Using SDKMAN:**
```bash
curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"
 sdk list java
sdk install java 21.0.1-tem
```

**Verification:**
Open a terminal and type:
```bash
java -version
javac -version
```

---

## 3. Hello World: The Anatomy
Create a file named `HelloWorld.java`.

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, Java!");
    }
}
```

### Line-by-Line Dissection

#### 1. `public class HelloWorld`
*   **`class`:** The keyword to define a blueprint for objects. Java is pure OOP; code must live inside a class.
*   **`HelloWorld`:** The class name.
    *   *Rule:* If the class is `public`, the filename **must** match exactly (`HelloWorld.java`).
    *   *Convention:* Use **PascalCase** for class names.

#### 2. `public static void main(String[] args)`
This is the **Entry Point** of any Java application.
*   **`public`:** Access modifier. The JVM (which is outside your code) needs access to call this method.
*   **`static`:** Means this method belongs to the *class*, not an instance of the class. The JVM can call this without creating an object (`new HelloWorld()`).
*   **`void`:** Return type. This method returns nothing.
*   **`main`:** The specific name the JVM looks for.
*   **`String[] args`:** Command-line arguments passed to the program. `args[0]` would be the first argument.

#### 3. `System.out.println("...")`
*   **`System`:** A built-in class in `java.lang` package that provides access to the system.
*   **`out`:** A static variable in `System` representing the "Standard Output" (console).
*   **`println`:** A method that prints the text and adds a new line (`\n`).

#### 4. Blocks `{ }`
Curly braces define the **scope**. The method lives inside the class scope. The statement lives inside the method scope.

#### 5. Semicolons `;`
Every statement in Java must end with a semicolon.

---

## 4. Running the Code

### The Manual Way (Two Steps)
1.  **Compile:**
    ```bash
    javac HelloWorld.java
    ```
    *   This creates `HelloWorld.class` (Bytecode).

2.  **Run:**
    ```bash
    java HelloWorld
    ```
    *   Note: Do **not** type `.class` or `.java`. You are running the *Class Name*.
    *   Output: `Hello, Java!`

### The Modern Way (Java 11+)
For single-file source code, you can run it directly without explicit compilation:
```bash
java HelloWorld.java
```
*   The JVM compiles it in memory and executes it immediately. Great for scripting and learning.