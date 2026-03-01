# Path and Files (NIO.2)

Java 7 introduced NIO.2 (JSR 203), which brought a vastly improved API for file system manipulation. The core of this API is the `java.nio.file` package.

## 1. The `Path` Interface

`Path` is the modern replacement for `java.io.File`. It represents a hierarchical path to a file or directory in the file system.

### Creating a Path
```java
import java.nio.file.Path;
import java.nio.file.Paths;

Path p1 = Paths.get("/home/user/logs/app.log");
Path p2 = Paths.get("C:\\Users\\Admin\\docs");
Path p3 = Paths.get("data", "input.txt"); // Joins parts
```

### Path Operations
`Path` objects are immutable.
```java
System.out.println("Filename: " + p1.getFileName());
System.out.println("Parent: " + p1.getParent());
System.out.println("Root: " + p1.getRoot());
System.out.println("Normalized: " + p1.normalize()); // Removes . and ..
```

## 2. The `Files` Utility Class

The `java.nio.file.Files` class contains static methods for operating on files and directories using `Path` objects.

### 2.1 Checking Existence
```java
import java.nio.file.Files;

if (Files.exists(p1)) {
    // ...
}
if (Files.notExists(p1)) {
    // ...
}
```

### 2.2 Copying, Moving, Deleting
```java
try {
    Path source = Paths.get("source.txt");
    Path dest = Paths.get("dest.txt");

    // Copy
    Files.copy(source, dest, StandardCopyOption.REPLACE_EXISTING);

    // Move / Rename
    Files.move(dest, Paths.get("renamed.txt"));

    // Delete
    Files.delete(Paths.get("renamed.txt"));
    // Files.deleteIfExists(path); // Safer
} catch (IOException e) {
    e.printStackTrace();
}
```

### 2.3 Reading and Writing (Small Files)
`Files` provides easy methods for reading/writing entire files at once.
*   **Warning:** Only use these for small files, as they load the entire content into memory.

```java
try {
    // Read all lines
    List<String> lines = Files.readAllLines(p1);
    
    // Write string
    String content = "Hello NIO.2";
    Files.write(p1, content.getBytes());
} catch (IOException e) {
    e.printStackTrace();
}
```

### 2.4 Streaming Directory Content
You can efficiently iterate over directories.
```java
try (Stream<Path> stream = Files.list(Paths.get("."))) {
    stream.forEach(System.out::println);
}
```

### 2.5 Attributes
`Files` allows easy access to file metadata.
```java
long size = Files.size(p1);
FileTime lastModified = Files.getLastModifiedTime(p1);
boolean isHidden = Files.isHidden(p1);
```

## 3. Summary
For all new file system operations in Java, prefer `Path` and `Files` over `java.io.File`. They offer better error handling, better support for large directories, and access to more file attributes.

```