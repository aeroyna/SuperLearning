## Introduction to Bit Manipulation

Bit manipulation is the act of algorithmically manipulating bits or other pieces of data shorter than a word. It involves using low-level bitwise operators to perform operations directly on the binary representation of numbers.

While it may seem esoteric, bit manipulation is a powerful technique that can lead to highly efficient and clever solutions for a specific class of problems. Understanding these techniques can be a significant advantage in coding interviews.

### Why Use Bit Manipulation?
- **Efficiency**: Bitwise operations are extremely fast, as they often map directly to single CPU instructions. This can lead to significant performance optimizations.
- **Space Savings**: You can encode a lot of information compactly. A single integer can represent a set of up to 32 or 64 boolean flags, which is far more space-efficient than an array of booleans. This is the core idea behind bitmasking.
- **Problem-Solving**: Some problems are inherently about the binary representation of numbers. For these, bit manipulation is not just an optimization but a necessity. Examples include finding numbers with an odd number of occurrences, swapping numbers without a temporary variable, or subset generation.

### Core Concepts
The foundation of bit manipulation rests on a few key concepts:

1.  **Binary Representation**: Understanding that any integer can be represented as a sequence of 0s and 1s, where each position represents a power of two.
2.  **Bitwise Operators**: The tools you use to manipulate bits. These include `AND`, `OR`, `XOR`, `NOT`, and bit shifts (`<<`, `>>`).
3.  **Bitmasks**: A "mask" is a specific bit pattern used to extract information or modify specific bits in another bit pattern. By `AND`ing with a mask, you can check if a bit is set. By `OR`ing, you can set a bit. By `XOR`ing, you can flip a bit.

While you may not use it as frequently as arrays or hash maps, a solid understanding of bit manipulation demonstrates a deeper knowledge of how computers work and is a valuable addition to any programmer's toolkit.


### Practice
- [Practice Problems](Practice_Problems/00_practice_problems.md)
