# Behavioral Patterns

Behavioral patterns deal with communication between objects.

## Observer
Defines a subscription mechanism to notify multiple objects about events.

```python
class Subject:
    def __init__(self):
        self._observers = []

    def attach(self, observer):
        self._observers.append(observer)

    def notify(self, message):
        for observer in self._observers:
            observer.update(message)

class Observer:
    def update(self, message):
        print(f"Received: {message}")

s = Subject()
o = Observer()
s.attach(o)
s.notify("Hello")
```

## Strategy
Defines a family of algorithms and makes them interchangeable.
In Python, this is often just passing a **function**.

```python
def sort_by_name(data):
    return sorted(data, key=lambda x: x['name'])

def sort_by_age(data):
    return sorted(data, key=lambda x: x['age'])

class Context:
    def __init__(self, strategy):
        self.strategy = strategy

    def do_sort(self, data):
        return self.strategy(data)

c = Context(sort_by_name)
```

## Iterator
Traverse a collection.
See [Intermediate/Generators](../../Intermediate/Generators_and_Iterators/01_iterators.md).
