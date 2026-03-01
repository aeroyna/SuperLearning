# Makefiles

`make` is a classic build automation tool. It uses a file called `Makefile` to define dependencies and build instructions.

## Basic Structure

```makefile
target: dependencies
    command
```
*   **Target**: The file you want to generate (e.g., `main.o`).
*   **Dependencies**: Files needed to generate the target (e.g., `main.cpp`).
*   **Command**: The shell command to run (Must be indented with **TAB**, not spaces).

## Example

```makefile
# Variables
CXX = g++
CXXFLAGS = -Wall -g

# The Final Executable
my_app: main.o utils.o
    $(CXX) $(CXXFLAGS) -o my_app main.o utils.o

# Object Files
main.o: main.cpp utils.h
    $(CXX) $(CXXFLAGS) -c main.cpp

utils.o: utils.cpp utils.h
    $(CXX) $(CXXFLAGS) -c utils.cpp

# Phony target (not a real file)
clean:
    rm -f *.o my_app
```

## Why use Make?
It only recompiles files that have changed (based on timestamps). If `utils.cpp` changes, only `utils.o` (and things that depend on it) are rebuilt.
