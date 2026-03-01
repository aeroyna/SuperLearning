# File System Concepts

## Overview

File systems provide a logical view of data storage and retrieval. Understanding file system concepts is crucial for system programming and interviews.

## Topics Covered

1. **[File Concept](01_file_concept.md)**
   - What is a file?
   - File attributes (name, type, location, size, protection, timestamps)
   - File operations (create, open, read, write, seek, delete, truncate)
   - Open file table

2. **[File Operations](02_file_operations.md)**
   - Opening and closing files
   - Reading and writing
   - Repositioning (seek)
   - File descriptors
   - File locking

3. **[File Types and Structure](03_file_types_structure.md)**
   - File types (regular, directory, special)
   - File structure (byte sequence, record sequence, tree)
   - File extensions
   - Magic numbers

4. **[Directory Structure](04_directory_structure.md)**
   - Single-level directory
   - Two-level directory
   - Tree-structured directory
   - Acyclic graph directory
   - General graph directory
   - Path names (absolute vs relative)

5. **[File System Mounting](05_file_system_mounting.md)**
   - Mount points
   - Mounting process
   - /etc/fstab
   - Network file systems

## Key Takeaways

- Files are named collections of related information
- Directory structures organize files hierarchically
- File operations are accessed via system calls
- Modern systems use tree-structured directories

## Interview Focus

- Explain file operations and their implementation
- Understand directory structures
- Know difference between absolute and relative paths
- Describe file system mounting
