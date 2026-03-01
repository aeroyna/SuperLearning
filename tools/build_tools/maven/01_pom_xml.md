# The Project Object Model (POM)

The `pom.xml` is the core of a Maven project. It defines the project's identity, dependencies, and build configuration.

## Key Elements

```xml
<project>
    <!-- Identity (GAV) -->
    <groupId>com.example</groupId>
    <artifactId>my-app</artifactId>
    <version>1.0.0-SNAPSHOT</version>

    <!-- Properties -->
    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <!-- Dependencies -->
    <dependencies>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>5.9.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

## GAV Coordinates
*   **GroupId**: Unique identifier of the organization (e.g., `org.springframework`).
*   **ArtifactId**: Name of the project (e.g., `spring-core`).
*   **Version**: Specific release (e.g., `6.0.0`). `SNAPSHOT` means "under development".

## Super POM
All POMs inherit from the **Super POM**, which defines defaults (like `src/main/java` directory structure, default plugins). This is why a minimal POM works out of the box.
