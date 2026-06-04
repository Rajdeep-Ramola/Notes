# Path Operation Configuration & JSON Compatible Encoder

## Tags

Add tags to path operations (decorator method) for more understandable docs.

```python
@app.post("/items/", tags=["items"])
async def create_item(item: Item) -> Item:
    return item

@app.get("/items/", tags=["items"])
async def read_items():
    return [{"name": "Foo", "price": 42}]
```

If we are using several tags in our application, we can store our tags in an `Enum`:

```python
class Tags(Enum):
    items = "items"
    users = "users"

@app.get("/items/", tags=[Tags.items])
async def get_items():
    return ["Portal gun", "Plumbus"]

@app.get("/users/", tags=[Tags.users])
async def read_users():
    return ["Rick", "Morty"]
```

---

## Summary and Description

```python
@app.post(
    "/items/",
    summary="Create an item",
    description="Create an item with all the information, name, description, price, tax and a set of unique tags",
)
```

A better way of writing a description is using docstrings:

```python
@app.post("/items/", summary="Create an item")
async def create_item(item: Item) -> Item:
    """
    Create an item with all the information:

    - **name**: each item must have a name
    - **description**: a long description
    - **price**: required
    - **tax**: if the item doesn't have tax, you can omit this
    - **tags**: a set of unique tag strings for this item
    """
    return item
```

---

## Deprecate a Path Operation

Deprecating a path operation without removing it:

```python
@app.get("/users/", tags=["users"])
async def read_users():
    return [{"username": "johndoe"}]

@app.get("/elements/", tags=["items"], deprecated=True)
async def read_elements():
    return [{"item_id": "Foo"}]
```

---

## JSON Compatible Encoder

When we need to convert a data type to a JSON-compatible data type (dict, list, etc.), FastAPI provides the `jsonable_encoder()` function for this.

It receives an object, like a Pydantic model, and returns a JSON-compatible version. For example, it would convert the Pydantic model to a dict, and the `datetime` to a `str`.

```python
from fastapi.encoders import jsonable_encoder
from pydantic import BaseModel

fake_db = {}

class Item(BaseModel):
    title: str
    timestamp: datetime
    description: str | None = None

app = FastAPI()

@app.put("/items/{id}")
def update_item(id: str, item: Item):
    json_compatible_item_data = jsonable_encoder(item)
    fake_db[id] = json_compatible_item_data
```

---

## Pydantic's Update Parameter

We can create a copy of the existing model using `.model_copy()`, and pass the `update` parameter with a dict containing the data to update.
