# Access Specifiers in Inheritance

Access specifiers (`public`, `protected`, `private`) control how the members of a base class are inherited by a derived class.

## `public` Inheritance

When a class is derived with `public` inheritance, the public members of the base class become public members of the derived class, and the protected members of the base class become protected members of the derived class. Private members of the base class are never accessible directly from a derived class.

| Base Class Member | How it's inherited in Derived Class |
|-------------------|-------------------------------------|
| `public`          | `public`                            |
| `protected`       | `protected`                         |
| `private`         | Not accessible                      |

This is the most common type of inheritance.

### Example

```cpp
class Base {
public:
    int x;
protected:
    int y;
private:
    int z;
};

class PublicDerived : public Base {
    // x is public
    // y is protected
    // z is not accessible
};
```

## `protected` Inheritance

When a class is derived with `protected` inheritance, the public and protected members of the base class become protected members of the derived class.

| Base Class Member | How it's inherited in Derived Class |
|-------------------|-------------------------------------|
| `public`          | `protected`                         |
| `protected`       | `protected`                         |
| `private`         | Not accessible                      |

### Example

```cpp
class Base {
public:
    int x;
protected:
    int y;
private:
    int z;
};

class ProtectedDerived : protected Base {
    // x is protected
    // y is protected
    // z is not accessible
};
```

## `private` Inheritance

When a class is derived with `private` inheritance, the public and protected members of the base class become private members of the derived class.

| Base Class Member | How it's inherited in Derived Class |
|-------------------|-------------------------------------|
| `public`          | `private`                           |
| `protected`       | `private`                           |
| `private`         | Not accessible                      |

### Example

```cpp
class Base {
public:
    int x;
protected:
    int y;
private:
    int z;
};

class PrivateDerived : private Base {
    // x is private
    // y is private
    // z is not accessible
};
```
In this case, the derived class "hides" the inheritance from the outside world. This is a way to implement the "is-implemented-in-terms-of" relationship, as opposed to the "is-a" relationship of public inheritance.
