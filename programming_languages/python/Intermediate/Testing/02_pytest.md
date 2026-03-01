# pytest

`pytest` is the de-facto standard for testing in Python due to its simplicity and powerful fixture system.

## Comparison with unittest

| Feature | unittest | pytest |
|---------|----------|--------|
| **Boilerplate** | High (Classes) | Low (Functions) |
| **Assertions** | `self.assertEqual(a, b)` | `assert a == b` |
| **Setup/Teardown** | `setUp`/`tearDown` | Fixtures (`@pytest.fixture`) |

## Basics

```python
# test_math.py

def add(x, y):
    return x + y

def test_add():
    assert add(1, 2) == 3
    assert add(0, 0) == 0
```
Run with `pytest`.

## Fixtures
Dependency injection for tests.

```python
import pytest

@pytest.fixture
def sample_data():
    return {"key": "value"}

def test_data(sample_data):
    assert sample_data["key"] == "value"
```

## Parametrization
Run same test with different inputs.

```python
@pytest.mark.parametrize("a,b,expected", [
    (1, 1, 2),
    (10, 20, 30),
    (0, 0, 0)
])
def test_add_many(a, b, expected):
    assert add(a, b) == expected
```
