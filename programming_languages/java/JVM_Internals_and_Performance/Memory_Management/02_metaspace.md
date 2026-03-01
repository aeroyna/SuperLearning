# Metaspace: Where Classes Live

Prior to Java 8, class metadata was stored in a special Heap area called **PermGen** (Permanent Generation). It had a fixed size, often leading to `java.lang.OutOfMemoryError: PermGen space`. Java 8 replaced this with **Metaspace**.

## 1. What is Metaspace?
Metaspace acts as the storage area for the JVM's internal representation of classes. When you load a class (e.g., `String.class`), the JVM parses the `.class` file and stores its structure here.

### Contents
*   Method Bytecodes.
*   Internal representation of classes and interfaces.
*   The Runtime Constant Pool.
*   Static variables (references are here, objects they point to are in Heap).

## 2. Key Characteristics
*   **Native Memory:** It is NOT part of the Java Heap. It uses native OS memory.
*   **Dynamic Sizing:** Unlike PermGen, Metaspace can grow automatically to accommodate metadata.
*   **GC:** Dead classes (those loaded by a ClassLoader that is no longer reachable) are garbage collected from Metaspace.

## 3. Configuration Flags
*   `-XX:MetaspaceSize`: The initial size.
*   `-XX:MaxMetaspaceSize`: The maximum limit. If not set, it can potentially use all available system RAM, causing OS-level swapping or OOM killer. **Best Practice:** Always set a limit in production containers.

## 4. The "Class Leak" Problem
Frameworks that generate classes dynamically (Spring, Hibernate, Mockito) create many temporary classes. If the ClassLoaders for these classes are not collected, Metaspace grows indefinitely until the process crashes.