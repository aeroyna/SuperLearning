# Testing in Python

Testing is essential for maintaining code quality and enabling confident refactoring.

---

## Overview

| Topic | Description |
|-------|-------------|
| [**1. unittest**](01_unittest.md) | Built-in testing framework |
| [**2. pytest**](02_pytest.md) | Popular third-party testing framework |
| [**3. Mocking**](03_mocking.md) | Simulating dependencies |
| [**4. Test-Driven Development**](04_tdd.md) | TDD principles and practice |

---

## Quick Reference

### unittest
```python
import unittest

class TestMath(unittest.TestCase):
    def setUp(self):
        self.value = 10

    def test_add(self):
        self.assertEqual(self.value + 5, 15)

    def test_subtract(self):
        self.assertEqual(self.value - 3, 7)

    def tearDown(self):
        pass

if __name__ == '__main__':
    unittest.main()
```

### pytest
```python
# test_math.py
def test_add():
    assert 1 + 1 == 2

def test_subtract():
    assert 5 - 3 == 2

# Run with: pytest test_math.py
```

### pytest Fixtures
```python
import pytest

@pytest.fixture
def sample_data():
    return [1, 2, 3, 4, 5]

def test_sum(sample_data):
    assert sum(sample_data) == 15

def test_length(sample_data):
    assert len(sample_data) == 5
```

---

## Assertions

### unittest
```python
self.assertEqual(a, b)
self.assertNotEqual(a, b)
self.assertTrue(x)
self.assertFalse(x)
self.assertIs(a, b)
self.assertIsNone(x)
self.assertIn(a, b)
self.assertRaises(Error, func, args)
```

### pytest
```python
assert x == y
assert x != y
assert x is None
assert x in collection

with pytest.raises(ValueError):
    function_that_raises()
```

---

## Mocking

```python
from unittest.mock import Mock, patch

# Basic mock
mock = Mock()
mock.method.return_value = 42
assert mock.method() == 42

# Patching
@patch('module.function')
def test_with_mock(mock_func):
    mock_func.return_value = "mocked"
    result = module.function()
    assert result == "mocked"
```

---

## Test Organization

```
project/
├── src/
│   └── mypackage/
│       └── module.py
└── tests/
    ├── __init__.py
    ├── test_module.py
    └── conftest.py  # pytest fixtures
```

---

## Next Steps
Start with [unittest](01_unittest.md).
