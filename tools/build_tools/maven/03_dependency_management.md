# Dependency Management

Maven automatically downloads libraries and their transitive dependencies.

## Standard Scopes

*   **compile** (default): Available in all classpaths (compilation, test, run). Transitive.
*   **test**: Only available for test compilation and execution (e.g., JUnit). Not transitive.
*   **provided**: Expected to be provided by the runtime environment (e.g., Servlet API provided by Tomcat).
*   **runtime**: Required for execution but not compilation (e.g., JDBC driver).
*   **system**: Similar to provided but requires explicit path (Avoid using this).

## Transitive Dependencies
If Project A depends on B, and B depends on C, then A automatically depends on C.

### Dependency Mediation
If multiple versions of the same artifact appear in the tree, Maven picks the **"nearest definition"** (shallowest in the tree).

### Exclusions
You can exclude unwanted transitive dependencies.

```xml
<dependency>
    <groupId>com.example</groupId>
    <artifactId>lib-a</artifactId>
    <exclusions>
        <exclusion>
            <groupId>com.example</groupId>
            <artifactId>lib-b</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

## Dependency Management Section
`<dependencyManagement>` in a parent POM defines versions *without* adding dependencies. Child modules can then add the dependency without specifying a version, ensuring consistency.
