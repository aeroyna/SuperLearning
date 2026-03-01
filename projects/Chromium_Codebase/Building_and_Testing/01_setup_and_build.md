# Chromium Setup & Build Guide

**[⬅️ Back to Building Overview](00_building_and_testing.md)**

---

## 🛠️ Prerequisites (Linux)

Building Chromium is resource-intensive.
*   **RAM**: At least 16GB (32GB+ recommended).
*   **Disk**: At least 100GB of SSD space.
*   **OS**: Ubuntu 20.04+ (or compatible).

### 1. Install Depot Tools
`depot_tools` is a package of scripts (gclient, gn, ninja) used to manage the checkout and build.

```bash
# Clone the tools
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git

# Add to PATH (add this to your .bashrc/.zshrc)
export PATH="$PATH:/path/to/depot_tools"
```

### 2. Get the Code
**Warning**: This downloads ~20-30GB of history and checkout.

```bash
mkdir chromium && cd chromium
fetch --no-history chromium
cd src
```
*   `fetch chromium`: Wrapper around `gclient` to initialize the `.gclient` file and sync code.
*   `--no-history`: Skips full git history to save time/space (shallow clone).

### 3. Install Dependencies
Chromium has a script to install all system packages (apt-get).

```bash
./build/install-build-deps.sh
```

---

## 🏗️ Building Chromium

Chromium uses **GN** (Generate Ninja) to generate build files and **Ninja** to execute the build.

### 1. Generate Build Files
Create a build directory (e.g., `out/Default`) and configure it.

```bash
gn gen out/Default
```

**Configure Arguments (`args.gn`)**:
To speed up the build for development, run `gn args out/Default` and add:
```python
is_debug = true
symbol_level = 0  # Reduces binary size and link time significantly
is_component_build = true # Faster incremental builds (uses shared libs)
```

### 2. Compile
Start the build. Go grab a coffee ☕.

```bash
autoninja -C out/Default chrome
```
*   `autoninja`: Wrapper that automatically sets job count based on CPU cores.
*   `chrome`: The target to build.

---

## 🏃 Running

Once built, the binary is in your output directory.

```bash
./out/Default/chrome
```

To run with a specific URL:
```bash
./out/Default/chrome https://google.com
```
