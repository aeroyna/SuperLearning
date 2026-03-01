# The CPython Object Model

In CPython, every object is a C struct called `PyObject`.

## `PyObject`
Defined in `Include/object.h`, it contains two key fields:

1.  **`ob_refcnt`**: Reference count (for memory management).
2.  **`ob_type`**: Pointer to the type object (class).

```c
typedef struct _object {
    _PyObject_HEAD_EXTRA
    Py_ssize_t ob_refcnt;
    PyTypeObject *ob_type;
} PyObject;
```

## `PyVarObject`
For variable-length objects (like list, str), there is `PyVarObject`, which adds:
3.  **`ob_size`**: Number of items in the container.

## Type Objects
The type of an object (e.g., `inttype`) is itself a `PyVarObject`. It contains function pointers for standard operations:
*   `tp_name`: String name ("int")
*   `tp_basicsize`: Size of instance in bytes
*   `tp_dealloc`: Destructor function
*   `tp_repr`: `__repr__` function
*   `tp_call`: `__call__` function

When you call `len(obj)`, CPython calls `obj->ob_type->tp_as_sequence->sq_length(obj)`.
