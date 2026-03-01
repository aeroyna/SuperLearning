# The `re` Module

Python's standard library support for regex.

## Common Functions

### `re.search(pattern, string)`
Scans string for *first* location. Returns Match object or None.

```python
import re
m = re.search(r'\d+', 'foo 123 bar')
print(m.group()) # '123'
```

### `re.match(pattern, string)`
Checks for match only at the **beginning** of the string.

### `re.findall(pattern, string)`
Returns all non-overlapping matches as a list of strings.

```python
print(re.findall(r'\d+', '123 abc 456')) # ['123', '456']
```

### `re.sub(pattern, repl, string)`
Replaces occurrences.

```python
print(re.sub(r'\s+', '-', 'hello world')) # 'hello-world'
```

### `re.compile(pattern)`
Compiles pattern into a regex object. Use this if using the same pattern multiple times for performance.

```python
pattern = re.compile(r'\d+')
pattern.findall('123 abc')
```
