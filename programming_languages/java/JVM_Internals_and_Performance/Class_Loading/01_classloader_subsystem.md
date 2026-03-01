# ClassLoader Subsystem: The Hierarchy

## 1. The Delegation Model
ClassLoaders are hierarchical. When a loader is asked to load a class, it delegates the request to its **Parent** first. Only if the parent fails does it try to load itself.

1.  **Bootstrap ClassLoader:** (Root). Loads core Java classes (`java.*`) from JDK internals. Written in C++.
2.  **Platform (Extension) ClassLoader:** Loads platform extensions.
3.  **Application (System) ClassLoader:** Loads your code from the `-classpath`.

**Why Delegation?**
Security. It prevents you from creating your own `java.lang.String` and tricking the JVM into loading it. The Bootstrap loader will always find the real one first.

## 2. Class Uniqueness
A class in Java is uniquely identified by **(ClassName + ClassLoader)**.
*   `MyClass` loaded by Loader A is **NOT** the same as `MyClass` loaded by Loader B.
*   They cannot be cast to each other. This is a common source of `ClassCastException` in complex containers (Tomcat, OSGi).

## 3. Custom ClassLoaders
You can write your own ClassLoader to:
*   Load classes from a database or network.
*   Decrypt encrypted class files on the fly.
*   Isolate modules (plugins) so they don't conflict (e.g., Plugin A uses Lib v1, Plugin B uses Lib v2).