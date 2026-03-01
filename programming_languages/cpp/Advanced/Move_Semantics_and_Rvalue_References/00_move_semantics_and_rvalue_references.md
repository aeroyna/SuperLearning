# Move Semantics and Rvalue References

Move semantics, introduced in C++11, is a major performance optimization that allows resources to be transferred (moved) from one object to another rather than copied. This is particularly important for managing heavy resources like dynamic memory or file handles. This chapter explains the difference between lvalues and rvalues, the syntax of rvalue references (`&&`), and how to implement move constructors and move assignment operators to make your classes efficient.

You will learn about:
- **Value Categories:** Understanding Lvalues (identifiable) vs. Rvalues (temporary).
- **Rvalue References:** Binding to temporary objects.
- **Move Semantics:** Stealing resources to avoid deep copies.
- **Perfect Forwarding:** Preserving value categories in template functions.

## In this chapter

- **[Lvalues and Rvalues](01_lvalues_and_rvalues.md)**
- **[Rvalue References](02_rvalue_references.md)**
- **[Move Constructors and Move Assignment](03_move_constructors_and_move_assignment.md)**
- **[Perfect Forwarding](04_perfect_forwarding.md)**
- **[Practice Problems](practice_problems.md)**
