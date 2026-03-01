# Abstract vs Concrete Data Structures

Understanding the difference between abstract data types (ADT) and concrete data structures (CDT) enables programmers to select the right data structure, design efficient algorithms, and write modular, reusable code.

## Overview

- **Abstract Data Type (ADT)**: A mathematical concept that defines what operations a data structure supports, from the user's perspective
- **Concrete Data Structure (CDT)**: The actual implementation of how data is stored and manipulated in memory

## Topics

- [2.1 Abstract Data Types](01_abstract_data_types.md) - Understanding ADTs and their importance
- [2.2 Choosing the Right Structure](02_choosing_the_right_structure.md) - Decision framework for interviews

## The Vending Machine Analogy

Think of a vending machine:
- **As a user (ADT perspective)**: You insert a coin, press a button, get a drink. You don't care how it works inside.
- **As an engineer (CDT perspective)**: You need to know if drinks are stored in bottles, cans, one big row, or multiple rows.

The same applies to data structures:
- **Stack (ADT)**: Defines push, pop, peek operations
- **Stack (CDT)**: Could be implemented using an array OR a linked list

## Key Distinctions

| Aspect | Abstract Data Type (ADT) | Concrete Data Structure (CDT) |
|--------|--------------------------|-------------------------------|
| Focus | What operations are available | How operations are implemented |
| Perspective | User/Client | Implementer |
| Defines | Behavior and interface | Memory layout and algorithms |
| Example | Stack, Queue, Map | ArrayList, LinkedList, HashMap |

## Common ADT to CDT Mappings

| ADT | Possible CDT Implementations |
|-----|------------------------------|
| List | ArrayList, LinkedList, Vector |
| Stack | Array-based Stack, Linked Stack |
| Queue | Array-based Queue, Linked Queue, Circular Queue |
| Deque | ArrayDeque, LinkedList |
| Set | HashSet, TreeSet, LinkedHashSet |
| Map | HashMap, TreeMap, LinkedHashMap |
| Priority Queue | Binary Heap, Fibonacci Heap |
| Graph | Adjacency Matrix, Adjacency List |

## Why This Matters for Interviews

1. **Language Agnostic Thinking**: Interviewers want to see you understand concepts, not just library calls
2. **Trade-off Analysis**: Different implementations have different time/space characteristics
3. **Problem Solving**: Choosing the right underlying structure is often the key insight
4. **Communication**: Using precise terminology shows depth of knowledge

## Interview Example

**Question**: "Implement a min stack that supports push, pop, top, and getMin in O(1)"

**ADT Thinking**:
- Need stack operations (push, pop, top)
- Need additional operation (getMin)

**CDT Decision**:
- Could use array or linked list for stack
- Need auxiliary structure for tracking min
- Array + additional stack/space for mins
