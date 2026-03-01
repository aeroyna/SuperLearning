# Build Systems

In modern Java development, managing the classpath manually is impossible. We use build systems to manage dependencies, compile code, and package artifacts.

## Maven

Maven is the industry standard. It uses an XML configuration (`pom.xml`) and favors convention over configuration.

> [!NOTE]
> For a comprehensive guide to Maven, including the POM structure, Lifecycles, and Multi-module projects, see the dedicated [**Maven Tools Section**](../../../../tools/build_tools/maven/00_overview.md).

## Gradle

Gradle is a more flexible build system that offers a Groovy or Kotlin DSL. It is the default for Android development and is widely used in microservices.

> [!NOTE]
> For a comprehensive guide to Gradle, including the DSL syntax and task configuration, see the dedicated [**Gradle Tools Section**](../../../../tools/build_tools/gradle/00_overview.md).
