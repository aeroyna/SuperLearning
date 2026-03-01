# Pythonic Idioms

Writing "Pythonic" code means exploiting the features of the language to write concise, readable, and performant code.

## Looping

### Index and Value
**Bad**:
```python
for i in range(len(items)):
    print(i, items[i])
```
**Good**:
```python
for i, item in enumerate(items):
    print(i, item)
```

### Two Lists at Once
**Bad**:
```python
for i in range(len(names)):
    print(names[i], ages[i])
```
**Good**:
```python
for name, age in zip(names, ages):
    print(name, age)
```

## Dictionary Lookups

**Bad**:
```python
if 'key' in d:
    value = d['key']
else:
    value = 'default'
```
**Good**:
```python
value = d.get('key', 'default')
```

## Context Managers

**Bad**:
```python
f = open('file.txt')
data = f.read()
f.close()
```
**Good**:
```python
with open('file.txt') as f:
    data = f.read()
```

## Swapping Variables

**Bad**:
```python
temp = a
a = b
b = temp
```
**Good**:
```python
a, b = b, a
```
