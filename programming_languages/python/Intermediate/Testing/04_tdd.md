# Test Driven Development (TDD)

TDD is a software development process where you write tests *before* writing the code.

## The Cycle (Red-Green-Refactor)

1.  **Red**: Write a failing test for a new feature.
    *   Fails because code doesn't exist yet.
2.  **Green**: Write simplest code to pass the test.
    *   Do not worry about quality or performance yet.
3.  **Refactor**: Improve the code while keeping tests green.

## Example

### 1. Red
```python
def test_calculator_add():
    from calculator import add
    assert add(1, 2) == 3
```
Running this fails (ImportError).

### 2. Green
```python
# calculator.py
def add(x, y):
    return x + y
```
Test passes.

### 3. Refactor
(Not needed for this simple case, but maybe add type hints).

## Benefits
*   Living documentation.
*   Confidence to refactor (regressions are caught immediately).
*   Better design (code is designed to be testable).
