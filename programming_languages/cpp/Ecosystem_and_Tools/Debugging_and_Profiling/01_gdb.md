# GDB (GNU Debugger)

GDB allows you to inspect your program while it is executing.

## Preparation
Compile with `-g` to include debug symbols.
```bash
g++ -g main.cpp -o app
```

## Basic Commands

### Starting
```bash
gdb ./app
```

### Flow Control
*   `run` (r): Start the program.
*   `break main` (b): Set breakpoint at function.
*   `break 15`: Set breakpoint at line 15.
*   `next` (n): Execute next line (step *over* functions).
*   `step` (s): Execute next line (step *into* functions).
*   `continue` (c): Continue running until next breakpoint.

### Inspection
*   `print x` (p): Print value of variable `x`.
*   `info locals`: Print all local variables.
*   `backtrace` (bt): Show the call stack (Crucial for crashes/segfaults!).
*   `list`: Show source code.

### TUI Mode
Run with `gdb -tui ./app` or press `Ctrl+X, A` inside GDB to see a visual split view of code and commands.
