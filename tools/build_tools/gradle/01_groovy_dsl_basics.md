# Groovy DSL Basics

Gradle build scripts are code. Unlike Maven's XML configuration, Gradle uses a Domain Specific Language (DSL) based on Groovy (or Kotlin).

## Syntax Primer

### Method Calls
Parentheses are optional in Groovy for top-level method calls.
```groovy
// These are equivalent
println("Hello")
println "Hello"
```

### Closures
Code blocks in curly braces `{}` are **Closures**. They are like lambdas that can accept a "delegate" object.
```groovy
dependencies {
    // Inside this block, method calls are delegated to the DependencyHandler object
    implementation 'com.google.guava:guava:31.1-jre'
}
```

### Properties
Getters and Setters are generated automatically.
```groovy
// Accessing 'version' property of the Project object
version = '1.0' 
println project.version
```

## Kotlin DSL
Ideally, new projects should use Kotlin DSL (`build.gradle.kts`) for better IDE support and type safety, but Groovy remains widely used in legacy builds.
