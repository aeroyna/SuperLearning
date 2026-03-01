# The Interpreter Architecture

## Components

1.  **Parser/Lexer**: Converts source code text into a Parse Tree (CST) and then an Abstract Syntax Tree (AST).
2.  **Compiler**: Traverses the AST and emits Bytecode (`.pyc`).
3.  **Evaluator (PVM)**: Executes the bytecode.

## GIL
The Global Interpreter Lock exists here, in the Evaluator loop. See [Concurrency/GIL](../Concurrency/02_gil.md).

## Value Stack vs Block Stack
*   **Value Stack**: Holds data (variables, intermediate results) for operations.
*   **Block Stack**: Keeps track of control flow constructs (loops, try/except blocks) to know where to jump if `break` or `exception` occurs.

## Frame Objects
Each function call creates a **Stack Frame** (`PyFrameObject`). This frame contains:
*   Local variables
*   Instruction pointer (what line is running)
*   Reference to global variables
*   Reference to the caller frame (forming the call stack)

This is why Recursion Depth is limited—each call consumes a frame object.
