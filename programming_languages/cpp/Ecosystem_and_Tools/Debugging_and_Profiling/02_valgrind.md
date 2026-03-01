# Valgrind

Valgrind is a suite of tools for dynamic analysis. The most useful tool is **Memcheck**, which detects memory errors.

## What it detects
*   Memory leaks (forgetting to `delete`).
*   Invalid reads/writes (buffer overflows, use-after-free).
*   Uninitialized variable usage.

## Usage

```bash
g++ -g main.cpp -o app
valgrind --leak-check=full ./app
```

## Output Analysis

### Memory Leak
```
definitely lost: 40 bytes in 1 blocks
```
Means you allocated memory (`new`) and lost the pointer without freeing it.

### Invalid Write
```
Invalid write of size 4
   at 0x4005D5: main (main.cpp:10)
```
Means you are writing to memory you don't own (e.g., writing to `arr[10]` when `arr` is size 10).
