# Custom Annotations

Beyond the built-in annotations, Java allows you to define your own custom annotations. These are incredibly powerful for adding metadata to your code, which can then be processed by tools, frameworks, or your own runtime logic (using Reflection).

## 1. Defining a Custom Annotation

You define an annotation using the `@interface` keyword.

```java
// Example: A simple custom annotation
public @interface MyCustomAnnotation {
    // Annotation elements (optional)
    String value() default "Default Value"; // A single element, often named 'value'
    int count() default 1;
    String[] tags() default {}; // Array of strings
}
```

### Annotation Elements
*   Elements are declared like methods but without parameters or implementation.
*   They can have default values.
*   Return types for elements must be:
    *   Primitive types (int, long, boolean, etc.)
    *   `String`
    *   `Class`
    *   `enum` type
    *   Other annotation types
    *   Arrays of any of the above types

### Using the Custom Annotation
```java
// On a class
@MyCustomAnnotation(value = "Class Level", count = 2, tags = {"tag1", "tag2"})
public class AnnotatedClass {

    // On a method
    @MyCustomAnnotation("Method Level") // 'value' can be omitted if it's the only element
    public void annotatedMethod(@MyCustomAnnotation(count = 3) String param) { // On a parameter
        // ...
    }

    // On a field
    @MyCustomAnnotation(count = 4)
    private String annotatedField;
}
```

## 2. Meta-Annotations (Annotations for Annotations)

Meta-annotations are annotations that are applied to custom annotation definitions. They control how your custom annotation behaves.

### 2.1 `@Retention`
Specifies how long the annotation is to be retained.
*   `RetentionPolicy.SOURCE`: Retained only in the source file, discarded by the compiler. (e.g., `@Override`, `@SuppressWarnings`)
*   `RetentionPolicy.CLASS`: Retained in the `.class` file, but discarded by the JVM at runtime. Default behavior.
*   `RetentionPolicy.RUNTIME`: Retained in the `.class` file and loaded by the JVM at runtime. This allows reflection to read the annotation (e.g., Spring, JUnit).

```java
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

@Retention(RetentionPolicy.RUNTIME) // Important for runtime processing
public @interface MyRuntimeAnnotation {
    String value();
}
```

### 2.2 `@Target`
Specifies the contexts in which the annotation is applicable.
*   `ElementType.TYPE`: Class, interface, enum, annotation.
*   `ElementType.FIELD`: Field.
*   `ElementType.METHOD`: Method.
*   `ElementType.ElementType.PARAMETER`: Method parameter.
*   `ElementType.CONSTRUCTOR`: Constructor.
*   `ElementType.LOCAL_VARIABLE`: Local variable.
*   `ElementType.ANNOTATION_TYPE`: Another annotation.
*   `ElementType.PACKAGE`: Package declaration.
*   `ElementType.TYPE_PARAMETER` (Java 8+): Type parameter (e.g., `<T>`).
*   `ElementType.TYPE_USE` (Java 8+): Any type use (e.g., `List<@NonNull String>`).

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Target;

@Target({ElementType.TYPE, ElementType.METHOD}) // Can be applied to classes AND methods
@Retention(RetentionPolicy.RUNTIME)
public @interface CustomAction {
    String name();
}
```

### 2.3 `@Inherited`
Indicates that an annotation type is automatically inherited by subclasses.
*   If a class is annotated with an `@Inherited` annotation, then its subclasses will also implicitly have that annotation (unless they override or explicitly remove it).
*   Applies only to class declarations.

```java
import java.lang.annotation.Inherited;

@Inherited // Subclasses will also have this annotation
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Author {
    String name();
}
```

### 2.4 `@Documented`
Indicates that annotations with this type are to be documented by Javadoc.

```java
import java.lang.annotation.Documented;

@Documented // Annotation will appear in Javadoc
@Retention(RetentionPolicy.RUNTIME)
public @interface Important {
}
```

## 3. Processing Custom Annotations (Reflection)

Custom annotations with `RetentionPolicy.RUNTIME` can be read and processed at runtime using Java Reflection API.

```java
// Example: Processing the CustomAction annotation
public class AnnotationProcessor {
    public static void process(Object obj) throws NoSuchMethodException {
        Class<?> clazz = obj.getClass();
        
        // Check class-level annotation
        if (clazz.isAnnotationPresent(CustomAction.class)) {
            CustomAction classAction = clazz.getAnnotation(CustomAction.class);
            System.out.println("Class Action: " + classAction.name());
        }

        // Check method-level annotation
        java.lang.reflect.Method method = clazz.getMethod("doSomething");
        if (method.isAnnotationPresent(CustomAction.class)) {
            CustomAction methodAction = method.getAnnotation(CustomAction.class);
            System.out.println("Method Action: " + methodAction.name());
        }
    }

    public static void main(String[] args) throws NoSuchMethodException {
        @CustomAction(name = "Class Level Task")
        class MyTask {
            @CustomAction(name = "Method Level Operation")
            public void doSomething() {
                System.out.println("Executing doSomething...");
            }
        }
        process(new MyTask());
    }
}
// Output:
// Class Action: Class Level Task
// Method Action: Method Level Operation
```

Custom annotations are a powerful feature that enables declarative programming, reduces boilerplate, and facilitates the integration of frameworks and tools by allowing metadata to drive application behavior.