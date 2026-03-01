# Using Context Managers

Context managers simplify resource management. They usually deal with operations that have a setup and teardown phase, such as opening files or database connections.

## The `with` Statement

The `with` statement ensures that clean-up code is executed even if errors occur during the block.

```python
# Without context manager
f = open('file.txt', 'w')
try:
    f.write('Hello')
finally:
    f.close()

# With context manager
with open('file.txt', 'w') as f:
    f.write('Hello')
# f.close() is called automatically here
```

## Common Uses
1.  **File I/O**: `open()`
2.  **Locks**: `threading.Lock()`
3.  **Database Transactions**: `session.begin()`
4.  **Testing**: `pytest.raises()`

## Multiple Context Managers
You can use multiple managers in one statement.

```python
with open('in.txt') as src, open('out.txt', 'w') as dst:
    dst.write(src.read())
```
