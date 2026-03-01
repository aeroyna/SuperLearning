# Exception Handling

Exception handling provides a systematic way to deal with runtime errors, ensuring that your program can maintain a consistent state or fail gracefully. In C++, exceptions allow you to separate error-handling code from regular code. This chapter explores the mechanism of throwing and catching exceptions, the hierarchy of standard exceptions provided by the library, and the critical concept of exception safety—ensuring that no resources are leaked and invariants are maintained even when things go wrong.

You will learn about:
- **Try, Catch, Throw:** The syntax and flow control of exception handling.
- **Stack Unwinding:** How automatic objects are destroyed when an exception propagates.
- **Standard Exceptions:** The `std::exception` hierarchy and common error types.
- **Exception Safety Guarantees:** Basic, Strong, and No-throw guarantees.

## In this chapter

- **[Try Catch Throw](01_try_catch_throw.md)**
- **[Standard Exceptions](02_standard_exceptions.md)**
- **[Exception Safety](03_exception_safety.md)**
- **[Practice Problems](practice_problems.md)**
