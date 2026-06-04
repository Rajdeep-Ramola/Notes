# Python Types

- **Type hints** — special syntax that allow declaring the *type* of a variable (`str`, `float`, `int`, `bool`)

**Example:**
```python
def get_full_name(first_name: str, last_name: str):
    full_name = first_name.title() + " " + last_name.title()
    return full_name

print(get_full_name("john", "doe"))
```

- Type hints are generally declared in function parameters

- **`typing` module** — if you want to declare something has 'any type'

**Example:**
```python
from typing import Any

def some_function(data: Any):
    print(data)
```

- **Generic types** — some types can take 'type parameters' in square brackets (`list`, `tuple`, `set`, `dict`)

**Example:**
```python
def process_items(items: list[str]):
    for item in items:
        print(item)
```

**Example:**
```python
def process_items(items_t: tuple[int, int, str], items_s: set[bytes]):
    return items_t, items_s
```

**Example:**
```python
def say_hi(name: str | None = None):
    if name is not None:
        print(f"Hey {name}!")
    else:
        print("Hello World")
```

> Using `str | None` instead of just `str` will let the editor help you detect errors where you could be assuming that a value is always a `str`, when it could actually be `None` too.

- **Classes as types**

**Example:**
```python
class Person:
    def __init__(self, name: str):
        self.name = name

def get_person_name(one_person: Person):
    return one_person.name
```

> This means "one_person is an **instance** of the class `Person`"

- **Pydantic models**

**Example:**
```python
from datetime import datetime
from pydantic import BaseModel

class User(BaseModel):
    id: int
    name: str = "John Doe"
    signup_ts: datetime | None = None
    friends: list[int] = []

external_data = {
    "id": "123",
    "signup_ts": "2017-06-01 12:22",
    "friends": [1, "2", b"3"],
}

user = User(**external_data)
print(user)
# > User id=123 name='John Doe' signup_ts=datetime.datetime(2017, 6, 1, 12, 22) friends=[1, 2, 3]
print(user.id)
# > 123
```

- **Type hints with metadata annotations** — feature that allows putting **additional *metadata*** in type hints using `Annotated`

**Example:**
```python
from typing import Annotated

def say_hello(name: Annotated[str, "this is just metadata"]) -> str:
    return f"Hello {name}"
```
