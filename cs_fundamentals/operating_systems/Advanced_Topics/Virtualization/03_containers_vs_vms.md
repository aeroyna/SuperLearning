# Containers vs VMs\n\n## Containers vs VMs

### Overview

Docker, LXC, isolation levels

### Architecture

```
Type 1 Hypervisor (Bare Metal):
┌────────────────────────────┐
│  VM 1    VM 2    VM 3      │
├────────────────────────────┤
│  Hypervisor (VMware ESXi)  │
├────────────────────────────┤
│  Hardware                  │
└────────────────────────────┘

Type 2 Hypervisor (Hosted):
┌────────────────────────────┐
│  VM 1    VM 2              │
├────────────────────────────┤
│  Hypervisor (VirtualBox)   │
├────────────────────────────┤
│  Host OS (Windows/Linux)   │
├────────────────────────────┤
│  Hardware                  │
└────────────────────────────┘
```

### Technologies

**Docker Container**:
```dockerfile
FROM ubuntu:20.04
RUN apt-get update
COPY app /app
CMD ["/app/run.sh"]
```

**Virtualization APIs**:
```python
import libvirt

conn = libvirt.open('qemu:///system')
dom = conn.createXML(xml_config)
dom.create()  # Start VM
```

### Comparison

Containers vs VMs:
- **Containers**: Lightweight, shared kernel, fast startup
- **VMs**: Full isolation, separate kernel, slower startup

## Key Takeaways

1. Virtualization enables resource consolidation
2. Hypervisors manage multiple guest OSes
3. Containers provide lightweight isolation
4. Hardware support improves performance

## Interview Focus

1. Compare Type 1 and Type 2 hypervisors
2. Containers vs VMs - when to use each?
3. How does hardware virtualization work?
4. What are benefits of virtualization?
