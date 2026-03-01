# Multi-Module Projects

Maven supports aggregating multiple projects under a single parent to manage them as a unit.

## Parent POM
The root folder contains a POM with packaging type `pom`.

```xml
<packaging>pom</packaging>
<modules>
    <module>core-service</module>
    <module>web-api</module>
</modules>
```

## The Reactor
The **Reactor** is the mechanism that analyzes the project DAG (Directed Acyclic Graph) to determine the correct build order.
*   Builds dependencies before dependents.
*   Allows building the entire system with one `mvn install` command from the root.

## Inheritance
Child POMs inherit configuration from the Parent:
*   `dependencies`
*   `dependencyManagement`
*   `pluginManagement`
*   `properties`

This drastically reduces duplication in microservices or layered architectures.
