# Coding Standards & Best Practices

**[⬅️ Back to Contribution Guide](00_contribution_guide.md)**

---

## 📐 C++ Style Guide

Chromium follows the **Google C++ Style Guide** with a few specific additions.

### 1. The Basics
*   **Naming**:
    *   `ClassName` (PascalCase)
    *   `method_name` (snake_case)
    *   `variable_name_` (snake_case + trailing underscore for members)
    *   `kConstantName` (k + PascalCase)
*   **Headers**: Use `#pragma once` (not header guards).
*   **Order**: Standard system headers -> Third party -> Project headers.

### 2. Smart Pointers
Chromium loves explicit ownership.
*   **`std::unique_ptr`**: Default ownership. "I own this."
*   **`scoped_refptr`**: Shared ownership (ref-counted). Used heavily for Threading tasks.
*   **`base::WeakPtr`**: "I want to reference this, but it might be gone." Essential for callbacks posted to tasks.

### 3. Strings
*   **`std::string`**: ASCII/UTF-8 byte implementation.
*   **`std::u16string`**: UTF-16 (used by UI and Blink internal strings).
*   **Conversions**: Use `base::UTF8ToUTF16()` and `base::UTF16ToUTF8()`.

---

## 🧪 Testing Expectations

If you touch it, you test it.

*   **New Feature**: Needs `browser_tests` (end-to-end) and `unit_tests`.
*   **Bug Fix**: Add a regression test that fails without your fix and passes with it.
*   **Flakiness**: A flaky test is worse than no test. It breaks the CQ for everyone.

---

## 🛡️ Mojo & IPC

Security is paramount when crossing process boundaries.

*   **Validation**: Never trust data from another process.
*   **Receivers**: Validate arguments in your Mojo implementation methods.
*   **Fuzzing**: Mojo interfaces are heavily fuzzed.

---

## 📝 CL Description Guidelines

Your Commit Message is the documentation for history.

*   **Summary**: Imperative mood. "Fix crash in tab restore." (Not "Fixed" or "Fixes").
*   **Body**: Explain **WHY**, not just WHAT.
    *   "The pointer was null." -> Bad.
    *   "The render frame host can be destroyed before the callback runs, leading to a UAF. Using a WeakPtr prevents this." -> Good.
*   **Bug**: Link the issue (e.g., `Bug: 123456`).
*   **Test**: Describe how you tested it. `Test: Verified manually on Linux by opening...`
