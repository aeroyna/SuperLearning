# JPMS Basics: Escaping Classpath Hell

The Module System addresses the fundamental flaws of the Classpath mechanism.

## 1. The Classpath Problem
*   **Brittle:** If a jar is missing, the app crashes only when that specific class is accessed (runtime).
*   **Shadowing:** If two jars contain `com.example.Util`, the one loaded first "wins". The other is shadowed.
*   **No Encapsulation:** Any public class in any jar is visible to every other jar. Internal APIs (like `sun.misc.Unsafe`) were accessible.

## 2. The Module Solution
*   **Resolved Graph:** The JVM verifies the graph of modules at startup. If a dependency is missing, the app fails to start (Reliable Configuration).
*   **Strong Encapsulation:** A module explicitly declares what it exposes. `public` inside a module does not mean "public to the world," only "public to modules that read me."

## 3. The Three Types of Modules
1.  **Application Modules:** Explicitly named, contain `module-info.java`.
2.  **Automatic Modules:** Regular JARs placed on the `--module-path`. Their name is derived from the filename. They export everything. This bridges the gap for legacy libraries.
3.  **Unnamed Module:** Everything on the classic `-classpath`. It reads everything, exports everything. It exists for backward compatibility.
