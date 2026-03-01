# RAII (Resource Acquisition Is Initialization)

RAII is a programming idiom used in C++ where the holding of a resource is a class invariant, and is tied to object lifetime. Resource allocation (or acquisition) is done during object creation (specifically initialization), by the constructor, while resource deallocation (release) is done during object destruction (specifically finalization), by the destructor. This chapter explains why RAII is the single most important pattern for exception safety and resource management in C++.

You will learn about:
- **Resource Management:** Handling file handles, network sockets, and memory.
- **Scope-Bound Resource Management:** How scopes determine resource lifetime.
- **Exception Safety:** How RAII guarantees cleanup even when exceptions are thrown.

## In this chapter

- **[RAII](01_raii.md)**
