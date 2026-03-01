# Bytecode

CPython does not execute source code directly. It compiles it into **Bytecode**, which is a set of instructions for the Python Virtual Machine (PVM).

## `.pyc` Files
Bytecode is cached in `__pycache__` directories to speed up loading.

## Inspecting Bytecode (`dis`)

```python
import dis

def add(a, b):
    return a + b

dis.dis(add)
```

**Output:**
```
  2           0 LOAD_FAST                0 (a)
              2 LOAD_FAST                1 (b)
              4 BINARY_ADD
              6 RETURN_VALUE
```

## The PVM
The PVM is a giant infinite loop (in `Python/ceval.c`) that reads the next opcode and executes a C function corresponding to it. It is a **Stack Machine**, meaning operations work by pushing and popping values from a stack.
