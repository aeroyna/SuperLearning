# unittest

`unittest` is the built-in testing framework in the Python standard library, inspired by JUnit.

## Basic Structure

1.  Import `unittest`.
2.  Inherit from `unittest.TestCase`.
3.  Method names starting with `test_` are run automatically.

```python
import unittest

def add(x, y):
    return x + y

class TestAdd(unittest.TestCase):
    def setUp(self):
        # Runs before EACH test method
        self.x = 10

    def tearDown(self):
        # Runs after EACH test method
        pass

    def test_add_positive(self):
        self.assertEqual(add(5, 5), 10)

    def test_add_negative(self,):
        # Using instance variable from setUp
        self.assertEqual(add(self.x, -5), 5)

if __name__ == '__main__':
    unittest.main()
```

## Assertions
*   `assertEqual(a, b)`
*   `assertTrue(x)`
*   `assertIn(item, list)`
*   `assertRaises(Error)`

## Pros/Cons
*   **Pros**: Built-in, no pip install needed.
*   **Cons**: Verbose (Boilerplate classes), CamelCase API (Java legacy).
