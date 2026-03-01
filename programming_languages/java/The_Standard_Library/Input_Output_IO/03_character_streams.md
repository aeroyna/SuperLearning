# Character Streams

While Byte Streams handle raw 8-bit bytes, **Character Streams** are designed to handle 16-bit Unicode characters. They automatically handle **Character Encoding** (translating between bytes and characters), making them ideal for text files.

## 1. The Abstract Classes
*   **`Reader`:** Superclass for reading character streams.
*   **`Writer`:** Superclass for writing character streams.

## 2. File Readers and Writers

*   `FileReader`: Convenience class for reading character files.
*   `FileWriter`: Convenience class for writing character files.

**Note:** `FileReader` and `FileWriter` use the **system default encoding**, which can be dangerous for portability. For specific encoding (like UTF-8), use `InputStreamReader` and `OutputStreamWriter` (see below).

## 3. Buffered Character Streams

Just like byte streams, buffering is crucial for performance.

*   **`BufferedReader`:** Reads text efficiently. Provides the very useful `readLine()` method.
*   **`BufferedWriter`:** Writes text efficiently. Provides `newLine()`.

### Example: Copying a Text File line by line
```java
import java.io.*;

public class CopyLines {
    public static void main(String[] args) {
        try (BufferedReader reader = new BufferedReader(new FileReader("input.txt"));
             BufferedWriter writer = new BufferedWriter(new FileWriter("output.txt"))) {
            
            String line;
            while ((line = reader.readLine()) != null) {
                writer.write(line);
                writer.newLine(); // Write system-dependent newline
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## 4. Bridge Streams (Byte to Char)

Sometimes you have a Byte Stream (e.g., from network or standard input `System.in`) but want to treat it as characters.

*   **`InputStreamReader`:** Reads bytes and decodes them into characters using a specified charset.
*   **`OutputStreamWriter`:** Encodes characters into bytes using a specified charset.

### Example: Reading Console Input
```java
// System.in is an InputStream (bytes)
// Wrap it in InputStreamReader to handle chars
// Wrap that in BufferedReader for line-by-line reading
BufferedReader console = new BufferedReader(new InputStreamReader(System.in));

System.out.print("Enter your name: ");
String name = console.readLine();
System.out.println("Hello, " + name);
```

### Specifying Encoding (Best Practice)
```java
// Writing UTF-8 text explicitly
try (Writer w = new BufferedWriter(
        new OutputStreamWriter(new FileOutputStream("utf8.txt"), "UTF-8"))) {
    w.write("こんにちは"); // Writing Japanese characters
}
```

## 5. `PrintWriter`
Often used for writing formatted text. `System.out` is a `PrintStream` (byte-based version of `PrintWriter`).
*   Methods: `print()`, `println()`, `printf()`.
*   Auto-flushing capability.