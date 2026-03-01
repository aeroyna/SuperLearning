# Sanitizers

Modern compilers (GCC, Clang) have built-in runtime instrumentation tools called Sanitizers. They are faster than Valgrind and often catch errors Valgrind misses.

## AddressSanitizer (ASan)
Detects memory errors (buffer overflow, use-after-free).

```bash
g++ -fsanitize=address -g main.cpp -o app
./app
```
If an error occurs, it prints a colorful report and aborts immediately.

## UndefinedBehaviorSanitizer (UBSan)
Detects undefined behavior (integer overflow, null pointer dereference, divide by zero).

```bash
g++ -fsanitize=undefined -g main.cpp -o app
```

## ThreadSanitizer (TSan)
Detects data races in multithreaded code.

```bash
g++ -fsanitize=thread -g main.cpp -o app
```

*Note: You usually cannot act TSan and ASan at the same time.*
