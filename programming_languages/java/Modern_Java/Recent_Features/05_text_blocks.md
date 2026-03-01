# Text Blocks (Java 15)

Text blocks provide a way to write multi-line strings without messy concatenation and newline escape sequences. Ideally suited for SQL, JSON, or HTML strings embedded in Java code.

## 1. Syntax
Uses triple quotes `"""`.

```java
String json = """
              {
                  "name": "Alice",
                  "age": 30
              }
              """;
```

## 2. Indentation
The compiler automatically strips "incidental" indentation (whitespace common to all lines), keeping your code aligned but the string content clean.