# Running Tests

**[⬅️ Back to Building Overview](00_building_and_testing.md)**

---

## 🧪 Types of Tests

Chromium has a massive test suite. Knowing which one to run is half the battle.

### 1. Unit Tests (`*_unittests`)
*   **Scope**: Small, isolated tests for specific classes or functions.
*   **Speed**: Fast (seconds).
*   **Examples**:
    *   `base_unittests`: Tests for strings, threads, files.
    *   `url_unittests`: Tests for URL parsing.
    *   `views_unittests`: Tests for UI views (without a full browser).

### 2. Browser Tests (`*_browsertests`)
*   **Scope**: Integration tests that spin up the full browser and renderer processes.
*   **Speed**: Slow (minutes).
*   **Examples**:
    *   `browser_tests`: The main suite for Chrome features.
    *   `content_browsertests`: Tests for the Content API logic.

### 3. Web Platform Tests (WPT)
*   **Scope**: Cross-browser tests maintained by the W3C. Checks compliance with web standards.
*   **Location**: `third_party/blink/web_tests/external/wpt`.

---

## 🏃 Execution Guide

### 1. Building the Test
You must build the test binary before running it.

```bash
# Build base_unittests
autoninja -C out/Default base_unittests

# Build browser_tests
autoninja -C out/Default browser_tests
```

### 2. Running the Test
Run the binary directly from the output directory.

```bash
./out/Default/base_unittests
```

### 3. Filtering Tests (GTest)
Don't run the whole suite! Use `--gtest_filter` to target your changes.

```bash
# Run all tests in the FileUtilTest suite
./out/Default/base_unittests --gtest_filter="FileUtilTest.*"

# Run a specific test case
./out/Default/base_unittests --gtest_filter="FileUtilTest.ReadFileToString"
```

### 4. Running Web Tests (Blink)
These use a special Python script wrapper.

```bash
# Run all web tests (don't do this)
third_party/blink/tools/run_web_tests.py -t Default

# Run a specific directory
third_party/blink/tools/run_web_tests.py -t Default fast/dom/

# Run a specific WPT test
third_party/blink/tools/run_web_tests.py -t Default external/wpt/html/semantics/
```

---

## 🐞 Debugging Tests

### 1. Parallelism
By default, tests run in parallel. This can make debugging hard or cause flakiness.
*   **Run serially**: `--test-launcher-jobs=1`

### 2. UI Output
Browser tests run headless (invisible) by default on some platforms or setups.
*   **Show me the window**: `--enable-pixel-output-in-tests` (depends on platform) or simply run in a local desktop environment.

### 3. Logs
*   **Verbose Logging**: `--v=1` (or higher) to see `VLOG(1)` output.
