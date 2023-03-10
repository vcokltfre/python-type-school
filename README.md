# Python Type School

Everything you need to know about typing in Python to write good, correct code.

## Index

- [What is type hinting?](#what-is-type-hinting)
- [Some basic typing](#some-basic-typing)
  - [Literals](#literals)
  - [User defined types](#user-defined-types)
- [More advanced typing](#more-advanced-typing)
  - [Unions, Optional, and Any types](#unions-optional-and-any-types)
  - [Generic types](#generic-types)
  - [Type aliases](#type-aliases)
  - [Type variables](#type-variables)
  - [Protocols](#protocols)
- [Overloading](#overloading)
- [Typing functions](#typing-functions)

Something missing or wrong? Open an issue with what's missing or incorrect and I'll try to fix it!

## What is type hinting?

Type hinting is an optional (but highly recommended) feature of Python which allows you to annotate the types of variables and functions. They're used primarily with a tool called a type checker - such as Pyright or MyPy - which read these hints and can use them to find errors in your code, for example, if you try to pass a `str` to a function expecting an `int`.

To type hint a variable, function parameter, or attribute on a class, you simply put the expected type with a colon after the variable name, for example:

```py
a: int = 5
b: str = "Hello"

def add(a: int, b: int): ...

class MyClass:
    c: bool = True
```

To type hint a function, you simply put the expected return type with an arrow after the function, for example:

```py
def add(a: int, b: int) -> int:
    return a + b
```

For the above example, if you attempt to pass a `str` to the `add` function, the type checker will throw an error. This won't stop your program from executing using the `python` command, but if your type checker is part of your IDE it will display that an invalid type has been passed. In scenarios such as CI/CD, this will usually entirely stop the code from ending up in production, reducing the risk of bugs at runtime, and all but eliminating `TypeError`s.

In addition to helping the type checker know whether your code types are correct before running it, type hinting code is also incredibly useful for being able to read and understand it at a later date. The type of a variable, as well as its name, give a lot of context as to its purpose. This is most apparent when figuring out what a function does without trying to interpret the whole function body.

Often, type checkers are able to implicitly infer the type of a variable, so here's a small guide of when it's recommended or unnecessary to type variables:

- Always type function parameters and return types. This will greatly aid in understanding what a function does, and will often provide type context that the type checker simply does not have available to infer the type.
- You should always type variables that can be any one of a [union] of types.
- You can - but don't need to - type variables set to a concrete value. For example, if you set `a = 5` its type can be trivially inferred using the value. It's also ok to do this with user defined concrete types, such as `a = MyClass()`, since - once again - the type checker can trivially infer that the value is of the type `MyClass`.
- You can - but don't need to - type variables which are the result of a typed function call. For example, if you set `a = get_int()` (where `get_int` is specified to have a return type of `int`) the type checker can infer that the type of `a` is `int`, since the function `get_int` returns an `int`. In cases where the function is not typed, you should type the variable.

## Some basic typing

### Literals

The most basic types are the literals, which are the literal data types that are built into Python. These are:

- `int`
- `float`
- `bool`
- `str`
- `bytes`
- `None`

These types are essentially as you'd expect them to be, and are very simple to type as they can simply be used as-is. For example:

```py
# Note that as mentioned above, you don't need to type variables set to a concrete value,
# and these are simply typed as an example of how to use these types as type hints.

a: int = 5
b: float = 5.0
c: bool = True
d: str = "Hello"
e: bytes = b"Hello"
e: None = None
```

### User defined types

User defined types are typed defined by you, or a library you import, and are not built into Python. These, once again, are simply typed using their name:

```py
class MyClass:
    ...

a: MyClass = MyClass()
```

## More advanced typing

Often, after a certain level of complexity, your programs will need to start using more complex and advanced types. The main concepts we'll cover here are:

- [Union, Optional, and Any types](#unions-optional-and-any-types)
- [Generics](#generic-types)
- [Type aliases](#type-aliases)
- [Type variables](#type-variables)
- [Protocols](#protocols)

### Unions, Optional, and Any types

The most common advanced type is the union type, which is a type that can be any one of a set of types. For example, if you have a function that can take either an `int` or a `str`, you can type it as `int | str`. For example:

```py
def add(a: int | str, b: int | str) -> int | str:
    if isinstance(a, int) and isinstance(b, int):
        return a + b
    elif isinstance(a, str) and isinstance(b, str):
        return a + b
    else:
        raise TypeError("a and b must be either both `int` or both `str`")
```

Note: In Python versions below 3.9, union types are represented using the `typing.Union` type from the `typing` module, for example `Union[int, str]`.

Optional types are types that can be either the type specified, or `None`. For example, if you have a function that can take either an `int` or `None`, you can type it as `Optional[int]`. For example:

```py
from typing import Optional

def add(a: int, b: Optional[int] = None) -> int:
    if b is None:
        return a
    else:
        return a + b
```

Note: In Python versions above 3.9 is is possible to type optional types using the `|` operator, for example `int | None`, since optional types are a union of the specified type and `None`. It is up to you which you prefer to use.

Any types are types that can be any type. For example, if you have a function that can take any type, you can type it as `Any`. For example:

```py
from typing import Any

def print_twice(obj: Any) -> None:
    print(obj)
    print(obj)
```

It is recommended to avoid using `Any` types as much as possible, as they essentially remove all type checking. However, there are some cases where they are useful, such as when you are using a library that is not typed, or when you are using a function that is not typed.

### Generic types

Generic types are types comprised of other types which can be, as the name suggests, generic. In Python, generics are represented using square braces (internally using the `__class_getitem__` dunder method).

Python has several built-in generic types:

- `list[T]`
- `dict[KT, KV]`
- `set[T]`
- `tuple[*T]`

Note: In Python versions below 3.9, typing these types requires importing their equivalent from the `typing` module, which is the name of the type but starting with an uppercase letter, for example `typing.List[T]`.

For lists, the type parameter `T` is the type of the items in the list. For example:

```py
a: list[int] = [1, 2, 3]
```

For dictionaries, the type parameters `KT` and `KV` are the types of the keys and values respectively. For example:

```py
a: dict[str, int] = {"a": 1, "b": 2, "c": 3}
```

For sets, the type parameter `T` is the type of the items in the set. For example:

```py
a: set[int] = {1, 2, 3}
```

For tuples, the type parameters `*T` are the types of the items in the tuple. For example:

```py
a: tuple[int, str, bool] = (1, "a", True)
```

In all of the above types, it is possible to use complex/compound types as the type parameter. For example:

```py
a: list[dict[str, int]] = [{"a": 1, "b": 2}, {"c": 3, "d": 4}]
```

Note: In the above examples, it is usually possible for the type checker to infer the type of the variable from the value, however in the name of readability and clarity, we will be explicitly typing the variables, and recommend you do the same.

### Type aliases

Type aliases are variables that are assigned to a type. These are useful if you have a type you often repeat in your code, or that you want to export from a module. For example, let's modify the above `add` function to use a type alias:

```py
IntOrStr = int | str

def add(a: IntOrStr, b: IntOrStr) -> IntOrStr:
    if isinstance(a, int) and isinstance(b, int):
        return a + b
    elif isinstance(a, str) and isinstance(b, str):
        return a + b
    else:
        raise TypeError("a and b must be either both `int` or both `str`")
```

### Type variables

Type variables are used to create user defined generic functions and classes. For example, let's create a function that simply returns exactly what it was given:

```py
from typing import TypeVar

T = TypeVar("T")

def identity(x: T) -> T:
    return x
```

Type variables can also be used to create generic classes, for example:

```py
from typing import TypeVar, Generic

T = TypeVar("T")

class Box(Generic[T]):
    def __init__(self, value: T) -> None:
        self.value = value

    def value_is(self, other: T) -> bool:
        return self.value == other

a: Box[int] = Box(1)
print(a.value_is(1))    # True
print(a.value_is("1"))  # Type error is reported
```

### Protocols

Protocols are a way of defining a set of methods that a type must implement. For example, let's create a protocol that specified which methods a class must implement to be considered "loadable", according to our toy definition:

```py
from typing import Protocol

class Loadable(Protocol):
    def load(self) -> None:
        ...

    def unload(self) -> None:
        ...

class Car:
    def load(self) -> None:
        print("Loading car")

    def unload(self) -> None:
        print("Unloading car")

class Truck:
    def load(self) -> None:
        print("Loading truck")

    def unload(self) -> None:
        print("Unloading truck")

a: Loadable = Car()    # Ok!
b: Loadable = Truck()  # Ok!
c: Loadable = 1        # Type error is reported
```

## Overloading

Python does not support true function overloading, however it allows you to type a function definition with overloads, while the function internally handles which type of input it uses and produces. For example, let's modify the `add` function to correctly report the output type when either `int` or `str` parameters are provided:

```py
from typing import overload

IntOrStr = int | str

@overload
def add(a: int, b: int) -> int:
    ...

@overload
def add(a: str, b: str) -> str:
    ...

def add(a: IntOrStr, b: IntOrStr) -> IntOrStr:
    if isinstance(a, int) and isinstance(b, int):
        return a + b
    elif isinstance(a, str) and isinstance(b, str):
        return a + b
    else:
        raise TypeError("a and b must be either both `int` or both `str`")

a: int = add(1, 2)      # Ok!
b: str = add("1", "2")  # Ok!
add(1, "2")             # Type error is reported
```

## Typing functions

Since functions in Python are first class objects, it's possible to pass them as variables, and thus it's important to be able to type them. This is done using `typing.Callable`. For example, let's create a function that takes a function as a parameter, and calls it:

```py
from typing import Any, Callable

def call(func: Callable[..., Any]) -> Any:
    return func()
```

In this case we have achieved what we wanted to, however the type information given to the type checker here is unsatisfactory, and there's more that we can do to help it - and ourselves - out. This is where both generic components `TypeVar` and `ParamSpec` come in.

```py
from typing import Callable, TypeVar, ParamSpec

P = ParamSpec("P")
R = TypeVar("R")

def call(func: Callable[P, R], *args: P.args, **kwargs: P.kwargs) -> R:
    return func(*args, **kwargs)
```

We've now removed the need for the use of `Any` entirely, which lets both us and the type checker reason better about the code. This is often used in decorators that wrap functions.
