# OOP Deep Dive

This section covers advanced Object-Oriented Programming concepts that are frequently asked in C++ interviews. Understanding these internals will help you write better code and debug complex issues.

## Topics Covered

- [[01_vtable_and_vptr|Virtual Tables (vtable) and vptr]]
- [[02_diamond_problem|The Diamond Problem]]
- [[03_object_slicing|Object Slicing]]
- [[04_virtual_inheritance|Virtual Inheritance]]
- [[05_shallow_vs_deep_copy|Shallow vs Deep Copy]]
- [[06_copy_elision_rvo_nrvo|Copy Elision (RVO/NRVO)]]
- [[practice_problems|Practice Problems]]

## Why These Topics Matter

These concepts appear frequently in:
- **Technical interviews** at top tech companies
- **Debugging** polymorphic code issues
- **Performance optimization** understanding
- **System design** decisions for large codebases

## Quick Reference

| Concept | What It Is | When It Matters |
|---------|------------|-----------------|
| vtable | Table of function pointers for virtual dispatch | Understanding polymorphism overhead |
| Diamond Problem | Ambiguous inheritance with shared base | Multiple inheritance design |
| Object Slicing | Losing derived part when copying to base | Passing objects by value |
| Virtual Inheritance | Solving diamond problem | Complex inheritance hierarchies |
| Deep vs Shallow Copy | Copying pointers vs pointed-to data | Resource management |
| Copy Elision | Compiler optimization avoiding copies | Performance, move semantics |

## Prerequisites

Before diving into these topics, ensure you understand:
- [[../Object-Oriented_Programming/Classes_and_Objects/01_classes_and_objects|Classes and Objects]]
- [[../Object-Oriented_Programming/Inheritance/01_inheritance|Inheritance]]
- [[../Object-Oriented_Programming/Polymorphism/02_virtual_functions|Virtual Functions]]
- [[../Memory_Management/Smart_Pointers/01_smart_pointers_intro|Smart Pointers]]
