# The File Class

The `java.io.File` class is the original representation of file and directory pathnames in Java. Although the newer NIO.2 API (Java 7+) offers a more robust `Path` class, `File` is still widely used in legacy code and for simple operations.

## 1. Concepts
*   **Abstract Representation:** A `File` object is an abstract representation of a file or directory path. It does **not** assume the file actually exists on the disk.
*   **Pathnames:** Can be absolute (full path from root) or relative (relative to the current working directory).

## 2. Creating a File Object
```java
import java.io.File;

// Does not create a file on disk, just the object
File file1 = new File("data.txt"); // Relative path
File file2 = new File("/home/user/docs/report.pdf"); // Absolute path (Linux/Mac)
File file3 = new File("C:\\Users\\User\\Documents\\image.png"); // Windows (note double backslashes)
```

## 3. Common Operations

### Checking Properties
```java
if (file1.exists()) {
    System.out.println("Name: " + file1.getName());
    System.out.println("Absolute Path: " + file1.getAbsolutePath());
    System.out.println("Writeable: " + file1.canWrite());
    System.out.println("Is Directory: " + file1.isDirectory());
    System.out.println("Size: " + file1.length() + " bytes");
} else {
    System.out.println("File does not exist.");
}
```

### Creating and Deleting
```java
try {
    // Create a new empty file
    boolean created = file1.createNewFile(); 
    if (created) System.out.println("File created.");
    
    // Create directories
    File dir = new File("logs/2023");
    dir.mkdirs(); // Creates parent directories if necessary
    
    // Delete
    // file1.delete(); 
} catch (IOException e) {
    e.printStackTrace();
}
```

### Listing Files
```java
File dir = new File(".");
String[] files = dir.list(); // Returns String array of names
File[] fileObjects = dir.listFiles(); // Returns File array

if (fileObjects != null) {
    for (File f : fileObjects) {
        System.out.println(f.getName());
    }
}
```

## 4. Limitations of `java.io.File`
*   **Error Handling:** Many methods return `boolean` (false) on failure instead of throwing exceptions, making it hard to diagnose *why* an operation failed.
*   **Performance:** Metadata access can be slow.
*   **Symbolic Links:** Poor support for symbolic links.

For modern development, prefer `java.nio.file.Path` and `java.nio.file.Files` (covered in the NIO chapter).