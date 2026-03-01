# Lifecycles and Phases

Maven uses a standard lifecycle to build projects. You don't tell Maven *how* to build; you tell it *what phase* you want to reach.

## The Default Lifecycle
The main lifecycle has sequential phases. Running a phase executes all preceding phases.

1.  **validate**: Check if project is correct and all info is available.
2.  **compile**: Compile source code (`src/main/java`).
3.  **test**: Run unit tests (`src/test/java`) using a testing framework.
4.  **package**: Package compiled code in distributable format (JAR/WAR).
5.  **verify**: Run integration tests.
6.  **install**: Install the package into the **local repository** (for use as a dependency in other local projects).
7.  **deploy**: Copy the package to the **remote repository** (Nexus/Artifactory) for sharing with others.

## Other Lifecycles
*   **clean**: Deletes the `target` directory (build artifacts).
*   **site**: Generates project documentation.

## Plugins and Goals
Maven is a plugin execution framework. A **Phase** is just a placeholder. Real work is done by **Plugins** bound to those phases.
*   `compile` phase maps to `maven-compiler-plugin:compile` goal.
*   `test` phase maps to `maven-surefire-plugin:test` goal.
