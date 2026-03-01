# Multithreading Models\n\n## Thread Mapping Models

### 1. Many-to-One
Multiple user threads → One kernel thread
- Pros: Fast, efficient
- Cons: No parallelism, one blocks all

### 2. One-to-One
Each user thread → One kernel thread
- Pros: True parallelism, independent blocking
- Cons: Higher overhead, limited threads
- Examples: Linux, Windows, modern Java

### 3. Many-to-Many
Multiple user threads → Multiple kernel threads
- Pros: Flexible, efficient, parallelism
- Cons: Complex implementation
- Examples: Solaris (old), older Windows

## Current Standard

**One-to-one** is most common in modern systems due to:
- Multicore processors
- Need for parallelism
- Acceptable overhead

## Key Takeaways

1. Three models: Many-to-one, one-to-one, many-to-many
2. Modern systems use one-to-one for parallelism
3. Trade-offs: efficiency vs parallelism vs complexity

## Interview Focus

1. Explain three multithreading models
2. Advantages/disadvantages of each
3. Which model does Linux/Windows use?
4. Why one-to-one is preferred?
