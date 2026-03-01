# Custom Annotations: Extending Metadata Capabilities

Building upon Java's built-in annotations, you can define your own **custom annotations**. This allows you to introduce metadata specific to your application's domain or framework, enabling powerful declarative programming paradigms. Custom annotations are particularly valuable when you want to externalize configuration, mark code for specific processing, or create domain-specific languages within Java.

## 1. Defining a Custom Annotation: The `@interface` Keyword

You define an annotation type using the `@interface` keyword, similar to an interface declaration.

```java
// Example: A simple custom annotation
// This annotation might mark a service method that needs logging
public @interface Loggable {
    // Annotation elements (optional):
    // These are like methods without parameters or implementations.
    // They define the data that can be passed to the annotation.

    String value() default "Default Log Message"; // An element named 'value'. If it's the only element, its name can be omitted when using the annotation.
    int level() default 1;                       // An element for logging level
    String[] tags() default {"general"};         // An array element
}
```

### Annotation Elements: Rules and Types
*   **Declaration:** Elements are declared like methods but without parameters or an implementation body.
*   **Default Values:** Elements can have default values using the `default` keyword.
*   **Return Types:** The return type for annotation elements is restricted to:
    *   All primitive types (`byte`, `short`, `int`, `long`, `float`, `double`, `boolean`, `char`).
    *   `String`.
    *   `Class` (e.g., `Class<?>`).
    *   `enum` types.
    *   Other annotation types.
    *   Arrays of any of the above types.

### Using the Custom Annotation
Once defined, you can apply your custom annotation to various program elements.

```java
// Applied to a class
@Loggable(value = "Service Class", level = 2, tags = {"service", "transaction"})
public class UserService {

    // Applied to a method, with value element abbreviated
    @Loggable("User creation triggered") 
    public void createUser(String username) {
        System.out.println("Creating user: " + username);
    }

    // Applied to a field
    @Loggable(level = 3)
    private String connectionString;

    // Applied to a parameter
    public void deleteUser(@Loggable(value = "User ID for deletion", level = 1) int userId) {
        System.out.println("Deleting user with ID: " + userId);
    }
}
```

## 2. Meta-Annotations: Controlling Annotation Behavior

Meta-annotations are annotations that are applied to custom annotation definitions themselves. They control fundamental properties of your custom annotation, such as where it can be applied, how long it's retained, and whether it's inherited.

### 2.1 `@Retention`
Specifies how long the annotation is to be retained (i.e., when it's available).
*   **`RetentionPolicy.SOURCE`:** The annotation is retained only in the source file (`.java`). It's discarded by the compiler and not included in the `.class` file. (e.g., `@Override`, `@SuppressWarnings`). Useful for compile-time checks or IDE processing.
*   **`RetentionPolicy.CLASS`:** The annotation is retained in the `.class` file but is discarded by the JVM at runtime. This is the default behavior if `@Retention` is not specified. Useful for post-compilation processing tools (e.g., bytecode manipulators).
*   **`RetentionPolicy.RUNTIME`:** The annotation is retained in the `.class` file and loaded by the JVM at runtime. This is crucial if you want to read and process the annotation using Java Reflection API. (e.g., Spring's `@Autowired`, JUnit's `@Test`).

```java
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

@Retention(RetentionPolicy.RUNTIME) // This annotation will be available at runtime via Reflection
public @interface MyRuntimeAnnotation {
    String name();
}
```

### 2.2 `@Target`
Specifies the contexts (types of program elements) in which the annotation is applicable. If `@Target` is not specified, the annotation can be applied to almost any program element.

*   **`ElementType.TYPE`:** Class, interface, enum, annotation.
*   **`ElementType.FIELD`:** Field (including enum constants).
*   **`ElementType.METHOD`:** Method.
*   **`ElementType.PARAMETER`:** Method parameter.
*   **`ElementType.CONSTRUCTOR`:** Constructor.
*   **`ElementType.LOCAL_VARIABLE`:** Local variable.
*   **`ElementType.ANNOTATION_TYPE`:** Another annotation definition.
*   **`ElementType.PACKAGE`:** Package declaration.
*   **`ElementType.TYPE_PARAMETER` (Java 8+):** Type parameter declaration (e.g., `<T>`).
*   **`ElementType.TYPE_USE` (Java 8+):** Any use of a type (e.g., `List<@NonNull String>`).

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Target;

@Target({ElementType.TYPE, ElementType.METHOD}) // This annotation can only be applied to classes AND methods
@Retention(RetentionPolicy.RUNTIME)
public @interface ServiceEndpoint {
    String path();
}
```

### 2.3 `@Inherited`
Indicates that an annotation type is automatically inherited by subclasses.
*   If a superclass is annotated with an `@Inherited` annotation, then its subclasses will also implicitly have that annotation (unless they override or explicitly apply a different one).
*   **Applies only to class declarations.** Annotations on methods or fields are NOT inherited by default.

```java
import java.lang.annotation.Inherited;

@Inherited // Subclasses of classes annotated with @Author will also be considered "authored"
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

@Documented // Annotation will appear in Javadoc documentation
@Retention(RetentionPolicy.RUNTIME)
public @interface Important {
}
```

## 3. Processing Custom Annotations (Reflection API)

For annotations with `RetentionPolicy.RUNTIME`, you can read and process them at runtime using Java's Reflection API. This is the mechanism by which frameworks inspect your code and configure their behavior.

```java
import java.lang.reflect.Method;

public class AnnotationProcessor {
    public static void analyzeClass(Class<?> clazz) throws NoSuchMethodException {
        System.out.println("Analyzing class: " + clazz.getName());

        // Check if class itself has the @ServiceEndpoint annotation
        if (clazz.isAnnotationPresent(ServiceEndpoint.class)) {
            ServiceEndpoint classAnno = clazz.getAnnotation(ServiceEndpoint.class);
            System.out.println("  Class is a Service Endpoint at path: " + classAnno.path());
        }

        // Check methods for @Loggable annotations
        for (Method method : clazz.getDeclaredMethods()) {
            if (method.isAnnotationPresent(Loggable.class)) {
                Loggable logAnno = method.getAnnotation(Loggable.class);
                System.out.println("  Method '" + method.getName() + "' is Loggable:");
                System.out.println("    Message: " + logAnno.value() + ", Level: " + logAnno.level());
                System.out.println("    Tags: " + String.join(", ", logAnno.tags()));
            }
        }
    }

    public static void main(String[] args) throws NoSuchMethodException {
        // Example Class using annotations
        @ServiceEndpoint(path = "/users")
        class MyUserService {
            @Loggable(value = "Fetching user details", level = 1, tags = {"read", "user"})
            public void getUserById(int id) {
                System.out.println("Fetching user with ID: " + id);
            }

            @Loggable(value = "Creating new user", level = 2, tags = {"write", "admin"})
            public void createUser(String username) {
                System.out.println("Creating user: " + username);
            }
            public void deleteUser(int id) { /* Not logged by @Loggable */ }
        }
        
        analyzeClass(MyUserService.class);
    }
}
// Expected Output:
// Analyzing class: AnnotationProcessor$1MyUserService
//   Class is a Service Endpoint at path: /users
//   Method 'getUserById' is Loggable:
//     Message: Fetching user details, Level: 1
//     Tags: read, user
//   Method 'createUser' is Loggable:
//     Message: Creating new user, Level: 2
//     Tags: write, admin
```

## 4. Summary: Annotations as Declarative Power

Custom annotations are a remarkably powerful feature that enables declarative programming, reduces boilerplate code, and facilitates the integration of frameworks and tools. By allowing you to embed metadata directly into your source code, annotations provide a flexible way to drive application behavior, externalize configuration, and create highly extensible software systems. They are a cornerstone of modern Java development, particularly in the Spring ecosystem and testing frameworks.

---

### Links to Topics:
*   [Enum Types](01_enum_types.md)
*   [Enum Methods and Constructors](02_enum_methods_and_constructors.md)
*   [Built-in Annotations](03_built_in_annotations.md)
*   [Custom Annotations](04_custom_annotations.md)
