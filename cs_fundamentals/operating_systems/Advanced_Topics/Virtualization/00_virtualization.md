# Virtualization

## Overview

Virtualization allows multiple operating systems to run on a single physical machine. This is fundamental to cloud computing and modern infrastructure.

## Topics Covered

1. **[Virtual Machines](01_virtual_machines.md)**
   - What is a virtual machine?
   - Benefits of virtualization
   - Virtual machine monitor (VMM)
   - Guest OS vs host OS
   - Full virtualization vs paravirtualization

2. **[Hypervisors](02_hypervisors.md)**
   - Type 1 hypervisor (bare-metal)
   - Type 2 hypervisor (hosted)
   - Examples: VMware ESXi, KVM, Xen, VirtualBox, Hyper-V
   - CPU virtualization
   - Memory virtualization
   - I/O virtualization

3. **[Containers vs VMs](03_containers_vs_vms.md)**
   - Container concept
   - Docker architecture
   - Namespaces and cgroups
   - Container vs VM comparison
   - When to use each

4. **[Hardware Virtualization Support](04_hardware_virtualization.md)**
   - Intel VT-x
   - AMD-V
   - Hardware-assisted virtualization
   - Performance benefits

## Key Takeaways

- Virtualization enables resource sharing and isolation
- Type 1 hypervisors run directly on hardware
- Containers share the host OS kernel
- Hardware support makes virtualization efficient

## Interview Focus

- Compare Type 1 and Type 2 hypervisors
- Explain containers vs VMs
- Understand benefits of virtualization
- Know real-world virtualization use cases
