# FastAPI — Path Parameter Validations, Query Parameter Models, and Body Parameters

## Path Parameters and Numeric Validations

We can declare validations and metadata for path parameters using `Path`.

### Declare Metadata

Declare a title for a path parameter:

```python
async def read_items(
    item_id: Annotated[int, Path(title="The ID of the item to get")],
):
```

### Number Validations

```python
async def read_items(
    item_id: Annotated[int, Path(title="The ID of the item to get", ge=1)], q: str
):
```

| Operator | Meaning                   |
|----------|---------------------------|
| `ge`     | Greater than or equal to  |
| `gt`     | Greater than              |
| `le`     | Less than or equal to     |
| `lt`     | Less than                 |

**Number validation for floats:**

```python
async def read_items(
    *,
    item_id: Annotated[int, Path(title="The ID of the item to get", ge=0, le=1000)],
    q: str,
    size: Annotated[float, Query(gt=0, lt=10.5)],
):
```

---

## Query Parameter Models

If you have a group of **query parameters** that are related, you can create a **Pydantic model** to declare them:

```python
from typing import Annotated, Literal
from fastapi import FastAPI, Query
from pydantic import BaseModel, Field

app = FastAPI()

class FilterParams(BaseModel):
    limit: int = Field(100, gt=0, le=100)
    offset: int = Field(0, ge=0)
    order_by: Literal["created_at", "updated_at"] = "created_at"
    tags: list[str] = []

@app.get("/items/")
async def read_items(filter_query: Annotated[FilterParams, Query()]):
    return filter_query
```

### Forbid Extra Query Parameters

Use Pydantic's model configuration to forbid any extra fields:

```python
class FilterParams(BaseModel):
    model_config = {"extra": "forbid"}

    limit: int = Field(100, gt=0, le=100)
    offset: int = Field(0, ge=0)
```

---

## Body — Multiple Parameters

### Multiple Body Parameters

```python
class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

class User(BaseModel):
    username: str
    full_name: str | None = None

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item, user: User):
    results = {"item_id": item_id, "item": item, "user": user}
    return results
```

> There are 2 body parameters in this function.

### Single Value in Body

To include a single value in the body (and not have it mistaken for a query parameter), use `Body`:

```python
async def update_item(
    item_id: int, item: Item, user: User, importance: Annotated[int, Body()]
):
```

### Single Body Parameter with Embedding

If you want your body to be structured like:

```json
{
    "item": {
        "name": "Foo",
        "description": "The pretender",
        "price": 42.0,
        "tax": 3.2
    }
}
```

Instead of:

```json
{
    "name": "Foo",
    "description": "The pretender",
    "price": 42.0,
    "tax": 3.2
}
```

Use the `embed` parameter of `Body`:

```python
class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Annotated[Item, Body(embed=True)]):
    results = {"item_id": item_id, "item": item}
    return results
```

---

## Validation and Metadata in Pydantic Models

Implemented through `Field`:

```python
from typing import Annotated
from fastapi import Body, FastAPI
from pydantic import BaseModel, Field

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str | None = Field(
        default=None, title="The description of the item", max_length=300
    )
    price: float = Field(gt=0, description="The price must be greater than zero")
    tax: float | None = None

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Annotated[Item, Body(embed=True)]):
    results = {"item_id": item_id, "item": item}
    return results
```

---

## Nested Models

We can define deep nested models in Pydantic.

**Basic list attribute:**
```python
class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None
    tags: list = []
```

**List with a type parameter:**
```python
class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None
    tags: list[str] = []
```

**Nested model example:**
```python
class Image(BaseModel):
    url: str
    name: str

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None
    tags: set[str] = set()
    image: Image | None = None
```

> Here `Image` is used as a type for an attribute of another Pydantic model.

- **Special types in Pydantic** — Pydantic offers more complex singular types that inherit from `str` (e.g. `HttpUrl`).

### Attributes with Lists of Submodels

We can also use Pydantic models as subtypes of `list`, `set`, etc.:

```python
class Image(BaseModel):
    url: HttpUrl
    name: str

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None
    tags: set[str] = set()
    images: list[Image] | None = None
```

### Body of `dict`

```python
@app.post("/index-weights/")
async def create_index_weights(weights: dict[int, float]):
    return weights
```
