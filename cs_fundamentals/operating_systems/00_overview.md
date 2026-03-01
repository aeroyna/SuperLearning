# Operating Systems Course Overview

A comprehensive Operating Systems learning path designed for technical interview preparation and deep understanding of OS fundamentals, ideal for FAANG+ interviews and senior engineering roles.

## Learning Path

### Phase 1: Fundamentals

- **[1. Introduction to Operating Systems](Fundamentals/Introduction/00_introduction.md)**
  - [1.1 What is an Operating System?](Fundamentals/Introduction/01_what_is_an_os.md)
  - [1.2 Operating System Services](Fundamentals/Introduction/02_os_services.md)
  - [1.3 System Calls](Fundamentals/Introduction/03_system_calls.md)
  - [1.4 Operating System Types](Fundamentals/Introduction/04_os_types.md)

- **[2. Computer System Organization](Fundamentals/Computer_System_Organization/00_computer_system_organization.md)**
  - [2.1 Computer System Operation](Fundamentals/Computer_System_Organization/01_computer_system_operation.md)
  - [2.2 Storage Structure](Fundamentals/Computer_System_Organization/02_storage_structure.md)
  - [2.3 I/O Structure](Fundamentals/Computer_System_Organization/03_io_structure.md)
  - [2.4 Interrupts and Exceptions](Fundamentals/Computer_System_Organization/04_interrupts_and_exceptions.md)

- **[3. Operating System Structure](Fundamentals/Operating_System_Structure/00_operating_system_structure.md)**
  - [3.1 Monolithic Systems](Fundamentals/Operating_System_Structure/01_monolithic_systems.md)
  - [3.2 Layered Approach](Fundamentals/Operating_System_Structure/02_layered_approach.md)
  - [3.3 Microkernels](Fundamentals/Operating_System_Structure/03_microkernels.md)
  - [3.4 Modules and Hybrid Systems](Fundamentals/Operating_System_Structure/04_modules_and_hybrid.md)

### Phase 2: Core Concepts

#### Process Management

- **[4. Processes](Process_Management/Processes/00_processes.md)**
  - [4.1 Process Concept](Process_Management/Processes/01_process_concept.md)
  - [4.2 Process State and PCB](Process_Management/Processes/02_process_state_and_pcb.md)
  - [4.3 Process Creation and Termination](Process_Management/Processes/03_process_creation_termination.md)
  - [4.4 Inter-Process Communication (IPC)](Process_Management/Processes/04_ipc.md)
  - [4.5 IPC Mechanisms: Pipes, Message Queues, Shared Memory](Process_Management/Processes/05_ipc_mechanisms.md)

- **[5. Threads](Process_Management/Threads/00_threads.md)**
  - [5.1 Thread Concept](Process_Management/Threads/01_thread_concept.md)
  - [5.2 Multithreading Models](Process_Management/Threads/02_multithreading_models.md)
  - [5.3 Thread Libraries (Pthreads, Java Threads)](Process_Management/Threads/03_thread_libraries.md)
  - [5.4 Threading Issues](Process_Management/Threads/04_threading_issues.md)

- **[6. CPU Scheduling](Process_Management/CPU_Scheduling/00_cpu_scheduling.md)**
  - [6.1 Scheduling Criteria](Process_Management/CPU_Scheduling/01_scheduling_criteria.md)
  - [6.2 FCFS Scheduling](Process_Management/CPU_Scheduling/02_fcfs_scheduling.md)
  - [6.3 SJF Scheduling](Process_Management/CPU_Scheduling/03_sjf_scheduling.md)
  - [6.4 Priority Scheduling](Process_Management/CPU_Scheduling/04_priority_scheduling.md)
  - [6.5 Round Robin Scheduling](Process_Management/CPU_Scheduling/05_round_robin_scheduling.md)
  - [6.6 Multilevel Queue Scheduling](Process_Management/CPU_Scheduling/06_multilevel_queue_scheduling.md)
  - [6.7 Real-Time Scheduling](Process_Management/CPU_Scheduling/07_real_time_scheduling.md)

- **[7. Process Synchronization](Process_Management/Process_Synchronization/00_process_synchronization.md)**
  - [7.1 Critical Section Problem](Process_Management/Process_Synchronization/01_critical_section_problem.md)
  - [7.2 Peterson's Solution](Process_Management/Process_Synchronization/02_petersons_solution.md)
  - [7.3 Synchronization Hardware](Process_Management/Process_Synchronization/03_synchronization_hardware.md)
  - [7.4 Mutex Locks](Process_Management/Process_Synchronization/04_mutex_locks.md)
  - [7.5 Semaphores](Process_Management/Process_Synchronization/05_semaphores.md)
  - [7.6 Monitors](Process_Management/Process_Synchronization/06_monitors.md)
  - [7.7 Classic Synchronization Problems](Process_Management/Process_Synchronization/07_classic_problems.md)

- **[8. Deadlocks](Process_Management/Deadlocks/00_deadlocks.md)**
  - [8.1 Deadlock Characterization](Process_Management/Deadlocks/01_deadlock_characterization.md)
  - [8.2 Resource Allocation Graph](Process_Management/Deadlocks/02_resource_allocation_graph.md)
  - [8.3 Deadlock Prevention](Process_Management/Deadlocks/03_deadlock_prevention.md)
  - [8.4 Deadlock Avoidance (Banker's Algorithm)](Process_Management/Deadlocks/04_deadlock_avoidance.md)
  - [8.5 Deadlock Detection and Recovery](Process_Management/Deadlocks/05_deadlock_detection_recovery.md)

#### Memory Management

- **[9. Main Memory](Memory_Management/Main_Memory/00_main_memory.md)**
  - [9.1 Address Binding](Memory_Management/Main_Memory/01_address_binding.md)
  - [9.2 Logical vs Physical Address](Memory_Management/Main_Memory/02_logical_vs_physical_address.md)
  - [9.3 Contiguous Memory Allocation](Memory_Management/Main_Memory/03_contiguous_allocation.md)
  - [9.4 Fragmentation](Memory_Management/Main_Memory/04_fragmentation.md)
  - [9.5 Paging](Memory_Management/Main_Memory/05_paging.md)
  - [9.6 Segmentation](Memory_Management/Main_Memory/06_segmentation.md)

- **[10. Virtual Memory](Memory_Management/Virtual_Memory/00_virtual_memory.md)**
  - [10.1 Demand Paging](Memory_Management/Virtual_Memory/01_demand_paging.md)
  - [10.2 Page Replacement Algorithms](Memory_Management/Virtual_Memory/02_page_replacement_algorithms.md)
  - [10.3 Thrashing](Memory_Management/Virtual_Memory/03_thrashing.md)
  - [10.4 Working Set Model](Memory_Management/Virtual_Memory/04_working_set_model.md)
  - [10.5 Memory-Mapped Files](Memory_Management/Virtual_Memory/05_memory_mapped_files.md)

- **[11. Memory Allocation Strategies](Memory_Management/Memory_Allocation/00_memory_allocation.md)**
  - [11.1 First Fit, Best Fit, Worst Fit](Memory_Management/Memory_Allocation/01_allocation_strategies.md)
  - [11.2 Buddy System](Memory_Management/Memory_Allocation/02_buddy_system.md)
  - [11.3 Slab Allocation](Memory_Management/Memory_Allocation/03_slab_allocation.md)

#### Storage Management

- **[12. File System Concepts](Storage_Management/File_System_Concepts/00_file_system_concepts.md)**
  - [12.1 File Concept](Storage_Management/File_System_Concepts/01_file_concept.md)
  - [12.2 File Operations](Storage_Management/File_System_Concepts/02_file_operations.md)
  - [12.3 File Types and Structure](Storage_Management/File_System_Concepts/03_file_types_structure.md)
  - [12.4 Directory Structure](Storage_Management/File_System_Concepts/04_directory_structure.md)
  - [12.5 File System Mounting](Storage_Management/File_System_Concepts/05_file_system_mounting.md)

- **[13. File System Implementation](Storage_Management/File_System_Implementation/00_file_system_implementation.md)**
  - [13.1 File System Structure](Storage_Management/File_System_Implementation/01_file_system_structure.md)
  - [13.2 Allocation Methods (Contiguous, Linked, Indexed)](Storage_Management/File_System_Implementation/02_allocation_methods.md)
  - [13.3 Free Space Management](Storage_Management/File_System_Implementation/03_free_space_management.md)
  - [13.4 Directory Implementation](Storage_Management/File_System_Implementation/04_directory_implementation.md)
  - [13.5 Journaling and Log-Structured File Systems](Storage_Management/File_System_Implementation/05_journaling_log_structured.md)

- **[14. Mass Storage Structure](Storage_Management/Mass_Storage_Structure/00_mass_storage_structure.md)**
  - [14.1 Disk Structure](Storage_Management/Mass_Storage_Structure/01_disk_structure.md)
  - [14.2 Disk Scheduling Algorithms](Storage_Management/Mass_Storage_Structure/02_disk_scheduling.md)
  - [14.3 RAID Structure](Storage_Management/Mass_Storage_Structure/03_raid_structure.md)
  - [14.4 SSD and Flash Storage](Storage_Management/Mass_Storage_Structure/04_ssd_flash_storage.md)

- **[15. I/O Systems](Storage_Management/I_O_Systems/00_io_systems.md)**
  - [15.1 I/O Hardware](Storage_Management/I_O_Systems/01_io_hardware.md)
  - [15.2 Application I/O Interface](Storage_Management/I_O_Systems/02_application_io_interface.md)
  - [15.3 Kernel I/O Subsystem](Storage_Management/I_O_Systems/03_kernel_io_subsystem.md)
  - [15.4 I/O Buffering and Caching](Storage_Management/I_O_Systems/04_io_buffering_caching.md)
  - [15.5 DMA](Storage_Management/I_O_Systems/05_dma.md)

### Phase 3: Advanced Topics

- **[16. Protection and Security](Advanced_Topics/Protection_and_Security/00_protection_and_security.md)**
  - [16.1 Protection Mechanisms](Advanced_Topics/Protection_and_Security/01_protection_mechanisms.md)
  - [16.2 Access Control](Advanced_Topics/Protection_and_Security/02_access_control.md)
  - [16.3 Security Threats](Advanced_Topics/Protection_and_Security/03_security_threats.md)
  - [16.4 Authentication](Advanced_Topics/Protection_and_Security/04_authentication.md)

- **[17. Virtualization](Advanced_Topics/Virtualization/00_virtualization.md)**
  - [17.1 Virtual Machines](Advanced_Topics/Virtualization/01_virtual_machines.md)
  - [17.2 Hypervisors (Type 1 vs Type 2)](Advanced_Topics/Virtualization/02_hypervisors.md)
  - [17.3 Containers vs VMs](Advanced_Topics/Virtualization/03_containers_vs_vms.md)
  - [17.4 Hardware Virtualization Support](Advanced_Topics/Virtualization/04_hardware_virtualization.md)

- **[18. Distributed Systems](Advanced_Topics/Distributed_Systems/00_distributed_systems.md)**
  - [18.1 Distributed System Architecture](Advanced_Topics/Distributed_Systems/01_distributed_architecture.md)
  - [18.2 Distributed File Systems](Advanced_Topics/Distributed_Systems/02_distributed_file_systems.md)
  - [18.3 Distributed Coordination](Advanced_Topics/Distributed_Systems/03_distributed_coordination.md)
  - [18.4 Distributed Synchronization](Advanced_Topics/Distributed_Systems/04_distributed_synchronization.md)

- **[19. Real-Time Systems](Advanced_Topics/Real_Time_Systems/00_real_time_systems.md)**
  - [19.1 Real-Time System Characteristics](Advanced_Topics/Real_Time_Systems/01_rt_characteristics.md)
  - [19.2 Real-Time Scheduling Algorithms](Advanced_Topics/Real_Time_Systems/02_rt_scheduling.md)
  - [19.3 Priority Inversion](Advanced_Topics/Real_Time_Systems/03_priority_inversion.md)

### Phase 4: Interview Preparation

- **[20. Interview Prep](Interview_Prep/00_interview_prep.md)**
  - [20.1 Top 50 OS Interview Questions](Interview_Prep/01_top_50_questions.md)
  - [20.2 Common Mistakes to Avoid](Interview_Prep/02_common_mistakes.md)
  - [20.3 FAANG OS Questions](Interview_Prep/03_faang_questions.md)
  - [20.4 OS Coding Problems](Interview_Prep/04_coding_problems.md)

---

## Key Topics for Interviews

### Must-Know Concepts
1. **Process vs Thread** - Fundamental difference and when to use each
2. **Context Switching** - What happens, overhead, optimization
3. **Deadlocks** - Four conditions, prevention, avoidance, detection
4. **Paging vs Segmentation** - Memory management schemes
5. **Page Replacement Algorithms** - FIFO, LRU, Optimal, Clock
6. **CPU Scheduling** - FCFS, SJF, Priority, Round Robin
7. **Synchronization Primitives** - Mutex, Semaphore, Monitor, Condition Variables
8. **Virtual Memory** - Demand paging, TLB, page faults
9. **File Systems** - Inode structure, allocation methods
10. **IPC Mechanisms** - Pipes, message queues, shared memory, sockets

### Classic Problems
- Producer-Consumer Problem
- Readers-Writers Problem
- Dining Philosophers Problem
- Sleeping Barber Problem
- Banker's Algorithm (Deadlock Avoidance)

---

## Recommended Study Order

### For Interview Prep (4-6 Weeks)
**Week 1-2**: Processes, Threads, CPU Scheduling, Process Synchronization
**Week 3-4**: Memory Management (Main Memory, Virtual Memory)
**Week 5**: Deadlocks, File Systems
**Week 6**: Review + Practice Interview Questions

### For Deep Understanding (8-12 Weeks)
Follow the complete learning path from Phase 1 → Phase 4

---

## Additional Resources

- **Classic Textbook**: "Operating System Concepts" by Silberschatz, Galvin, Gagne (Dinosaur Book)
- **Modern Approach**: "Operating Systems: Three Easy Pieces" (Free online)
- **Linux Kernel**: Understanding Linux kernel source code
- **Practice**: Implement simple OS components (scheduler, memory allocator, file system)

---

## Interview Tips

1. **Always explain trade-offs** - No solution is perfect; discuss pros/cons
2. **Use diagrams** - Draw process states, page tables, resource allocation graphs
3. **Know real-world examples** - Linux scheduler, Windows memory management
4. **Understand the "why"** - Don't just memorize algorithms, understand the problems they solve
5. **Connect to system programming** - Link OS concepts to actual system calls and APIs
