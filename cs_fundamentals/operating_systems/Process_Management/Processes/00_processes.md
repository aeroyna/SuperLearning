# Processes

## Overview

A process is a program in execution. Understanding processes is fundamental to operating systems - they are the basic unit of work in a system.

## Topics Covered

1. **[Process Concept](01_process_concept.md)**
   - What is a process?
   - Process vs Program
   - Process components (text, data, stack, heap)
   - Process in memory layout

2. **[Process State and PCB](02_process_state_and_pcb.md)**
   - Process states (New, Ready, Running, Waiting, Terminated)
   - State transitions
   - Process Control Block (PCB)
   - Context switching

3. **[Process Creation and Termination](03_process_creation_termination.md)**
   - Process creation: fork() system call
   - Process termination: exit() system call
   - Process hierarchy (parent-child relationship)
   - Orphan and zombie processes
   - exec() family of system calls

4. **[Inter-Process Communication (IPC)](04_ipc.md)**
   - Why IPC?
   - Shared memory vs message passing
   - Direct vs indirect communication
   - Synchronous vs asynchronous communication
   - Buffering strategies

5. **[IPC Mechanisms](05_ipc_mechanisms.md)**
   - Pipes (named and unnamed)
   - Message queues
   - Shared memory
   - Sockets
   - Signals
   - POSIX vs System V IPC

## Key Takeaways

- Process is an active entity (program is passive)
- PCB contains all information about a process
- Context switching has overhead
- IPC allows processes to communicate and synchronize

## Interview Focus

- Explain process states and transitions
- Describe what happens during a context switch
- Compare IPC mechanisms and their use cases
- Understand fork(), exec(), wait() system calls
- Zombie vs orphan processes
