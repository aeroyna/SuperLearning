# Dependency Management

Gradle uses **Configurations** (buckets of dependencies) instead of simple scopes.

## Modern Configurations (Java Plugin)

1.  **implementation**: Implementation details. Available to local code, but **NOT** exposed to consumers (other projects depending on this one). Faster builds.
2.  **api** (requires `java-library` plugin): Exposed to consumers. Use for transitive dependencies that are part of your public API.
3.  **compileOnly**: Required at compile time but not at runtime (like Maven `provided`).
4.  **testImplementation**: Available only for tests.
5.  **runtimeOnly**: Available only at runtime.

## Defining Dependencies

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web:3.0.0'
    testImplementation 'org.junit.jupiter:junit-jupiter:5.9.2'
}
```

## Resolution
Gradle has powerful conflict resolution capabilities, typically picking the **newest** version by default (unlike Maven's "nearest" strategy).
