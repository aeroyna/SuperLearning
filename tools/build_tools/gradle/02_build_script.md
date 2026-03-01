# Build Script Structure

The `build.gradle` file defines the build logic.

## Plugins
Gradle core is tiny. Functionality is added via plugins.

```groovy
plugins {
    id 'java'
    id 'application'
}
```
The `java` plugin adds tasks like `compileJava` and `test`.

## Repositories
Where to find dependencies.

```groovy
repositories {
    mavenCentral()
}
```

## Tasks
The unit of work.

```groovy
tasks.register('hello') {
    doLast {
        println 'Hello, World!'
    }
}
```
Run with `./gradlew hello`.

## The Wrapper (`gradlew`)
Always use the wrapper scripts (`gradlew`, `gradlew.bat`) committed to the repo. They download the exact correct version of Gradle for the project, ensuring reproducibility without requiring manual installation.
