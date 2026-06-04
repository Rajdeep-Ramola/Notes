# FastAPI — Path Parameters and Query Parameters

## Basic FastAPI App

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def root():
    return {"message": "Hello World"}
```

---

## Path Parameters

**Example:**
```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(item_id):
    return {"item_id": item_id}
```

> The value of the path parameter `item_id` will be passed to your function as the argument `item_id`.

You can declare the **type** of a path parameter in the function using standard Python type annotations:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id}
```

### Path Converter

We can declare a path parameter containing a path using a URL like `/files/{file_path:path}`:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/files/{file_path:path}")
async def read_file(file_path: str):
    return {"file_path": file_path}
```

---

## Query Parameters

Any function parameters that are **not** part of the path parameters are automatically interpreted as **query parameters**.

**Example:**
```python
from fastapi import FastAPI

app = FastAPI()

fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]

@app.get("/items/")
async def read_item(skip: int = 0, limit: int = 10):
    return fake_items_db[skip : skip + limit]
```

> Query is the set of key-value pairs that go after the `?` in a URL, separated by `&` characters.
> Example: `http://127.0.0.1:8000/items/?skip=0&limit=10`

Query parameters are not a fixed part of a path — they can be optional and can have default values.

### Optional Parameters

Declare optional query parameters by setting their default to `None`:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(item_id: str, q: str | None = None):
    if q:
        return {"item_id": item_id, "q": q}
    return {"item_id": item_id}
```

### Multiple Path and Query Parameters

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/users/{user_id}/items/{item_id}")
async def read_user_item(
    user_id: int, item_id: str, q: str | None = None, short: bool = False
):
    item = {"item_id": item_id, "owner_id": user_id}
    if q:
        item.update({"q": q})
    if not short:
        item.update(
            {"description": "This is an amazing item that has a long description"}
        )
    return item
```

### Required Query Parameters

To make a query parameter required, do not declare any default value:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
async def read_user_item(item_id: str, needy: str):
    item = {"item_id": item_id, "needy": needy}
    return item
```
