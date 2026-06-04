# File Structure, APIRouter & Stream JSON Lines

## File Structure

Example file structure for a FastAPI project:

```
.
├── app                  # "app" is a Python package
│   ├── __init__.py      # this file makes "app" a "Python package"
│   ├── main.py          # "main" module, e.g. import app.main
│   ├── dependencies.py  # "dependencies" module, e.g. import app.dependencies
│   └── routers          # "routers" is a "Python subpackage"
│   │   ├── __init__.py  # makes "routers" a "Python subpackage"
│   │   ├── items.py     # "items" submodule, e.g. import app.routers.items
│   │   └── users.py     # "users" submodule, e.g. import app.routers.users
│   └── internal         # "internal" is a "Python subpackage"
│       ├── __init__.py  # makes "internal" a "Python subpackage"
│       └── admin.py     # "admin" submodule, e.g. import app.internal.admin
```

- The `app` directory contains everything. It has an empty `app/__init__.py`, so it is a "Python package": `app`.
- `app/main.py` is a "module" of that package: `app.main`.
- `app/dependencies.py` is a "module": `app.dependencies`.
- `app/routers/` is a "Python subpackage": `app.routers`.
- `app/routers/items.py` is a submodule: `app.routers.items`.
- `app/routers/users.py` is a submodule: `app.routers.users`.
- `app/internal/` is another "Python subpackage": `app.internal`.
- `app/internal/admin.py` is another submodule: `app.internal.admin`.

---

## APIRouter

Used to create path operations that are separated from the rest of the code. Think of `APIRouter` as a mini FastAPI class.

```python
from fastapi import APIRouter

router = APIRouter()

@router.get("/users/", tags=["users"])
async def read_users():
    return [{"username": "Rick"}, {"username": "Morty"}]

@router.get("/users/me", tags=["users"])
async def read_user_me():
    return {"username": "fakecurrentuser"}

@router.get("/users/{username}", tags=["users"])
async def read_user(username: str):
    return {"username": username}
```

For the same dependencies, prefix, tags, etc. across different endpoints, we can declare them in `APIRouter` instead of including them in every path operation:

```python
from fastapi import APIRouter, Depends, HTTPException
from ..dependencies import get_token_header

router = APIRouter(
    prefix="/items",
    tags=["items"],
    dependencies=[Depends(get_token_header)],
    responses={404: {"description": "Not found"}},
)

fake_items_db = {"plumbus": {"name": "Plumbus"}, "gun": {"name": "Portal Gun"}}

@router.get("/")
async def read_items():
    return fake_items_db

@router.get("/{item_id}")
async def read_item(item_id: str):
    if item_id not in fake_items_db:
        raise HTTPException(status_code=404, detail="Item not found")
    return {"name": fake_items_db[item_id]["name"], "item_id": item_id}

@router.put(
    "/{item_id}",
    tags=["custom"],
    responses={403: {"description": "Operation forbidden"}},
)
async def update_item(item_id: str):
    if item_id != "plumbus":
        raise HTTPException(
            status_code=403, detail="You can only update the item: plumbus"
        )
    return {"item_id": item_id, "name": "The great Plumbus"}
```

---

## Stream JSON Lines

If we have a sequence of data to send in a stream, we can do it with **JSON Lines** — a format where we send one JSON object per line.

To stream JSON Lines with FastAPI, instead of using `return` in a path operation function, use `yield` to produce each item in turn.

If each JSON item is of type `Item` (a Pydantic model) and it's an async function, we can declare the return type as `AsyncIterable[Item]`.

```python
from collections.abc import AsyncIterable, Iterable
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str | None

items = [
    Item(name="Plumbus", description="A multi-purpose household device."),
    Item(name="Portal Gun", description="A portal opening device."),
    Item(name="Meeseeks Box", description="A box that summons a Meeseeks."),
]

@app.get("/items/stream")
async def stream_items() -> AsyncIterable[Item]:
    for item in items:
        yield item

@app.get("/items/stream-no-async")
def stream_items_no_async() -> Iterable[Item]:
    for item in items:
        yield item

@app.get("/items/stream-no-annotation")
async def stream_items_no_annotation():
    for item in items:
        yield item

@app.get("/items/stream-no-async-no-annotation")
def stream_items_no_async_no_annotation():
    for item in items:
        yield item
```
