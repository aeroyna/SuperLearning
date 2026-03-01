# GC Basics: The Cycle of Life

## 1. Memory Pools (Generational)

### 1.1 Eden Space (Young)
*   New objects (`new Object()`) are allocated here.
*   When Eden fills up -> **Minor GC** triggers.
*   Live objects are moved to Survivor spaces. Dead objects are wiped instantly.

### 1.2 Survivor Spaces (S0 / S1)
*   Two small spaces acting as a buffer between Eden and Old.
*   Objects ping-pong between S0 and S1 during Minor GCs, incrementing their "Age".

### 1.3 Old Gen (Tenured)
*   Objects that survive typically 15 cycles (MaxTenuringThreshold) are moved here.
*   Large objects (arrays) might skip Young Gen and go straight here.
*   When Old Gen fills up -> **Major GC** (or Full GC) triggers. This often pauses the application ("Stop-The-World").

## 2. "Stop-The-World" (STW)
During a GC, the application threads must be paused so the memory map doesn't change while the collector is working.
*   **Minor GC:** Very short STW (negligible).
*   **Full GC:** Can be long (seconds). This causes lag spikes in apps. Minimizing Full GC is the primary goal of tuning.

## 3. Allocation Mechanics
*   **Bump-the-pointer:** Efficient allocation. Just increment a pointer in Eden.
*   **TLAB (Thread Local Allocation Buffer):** To avoid locking on the shared Eden space, each thread gets a tiny private slice of Eden to allocate objects lock-free.