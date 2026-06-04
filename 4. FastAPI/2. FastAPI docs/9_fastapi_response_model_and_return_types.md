# Response Model in FastAPI

## Return Type

You can declare the type used for the response of a function — e.g. Pydantic models, lists, dicts, scalar values, etc.

FastAPI will use this return type to:
- Validate the returned data
- Add a JSON schema for the response (used by automatic docs)
- Serialize the returned data to JSON (written in Rust, so it's much faster)

```python
class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None
    tags: list[str] = []

@app.post("/items/")
async def create_item(item: Item) -> Item:
    return item

@app.get("/items/")
async def read_items() -> list[Item]:
    return [
        Item(name="Portal Gun", price=42.0),
        Item(name="Plumbus", price=32.0),
    ]
```

## `response_model` Parameter

Use `response_model` when you want to return data that is not exactly what the declared type is.

> **Note:** `response_model` is a parameter of the decorator method (`get`, `post`, etc.).  
> If you declare both a return type and a `response_model`, the `response_model` will take priority.

```python
@app.post("/items/", response_model=Item)
async def create_item(item: Item) -> Any:
    return item

@app.get("/items/", response_model=list[Item])
async def read_items() -> Any:
    return [
        {"name": "Portal Gun", "price": 42.0},
        {"name": "Plumbus", "price": 32.0},
    ]
```

## Return a Response Directly

```python
from fastapi.responses import JSONResponse, RedirectResponse

app = FastAPI()

@app.get("/portal")
async def get_portal(teleport: bool = False) -> Response:
    if teleport:
        return RedirectResponse(url="https://www.youtube.com/watch?v=dQw4w9WgXcQ")
    return JSONResponse(content={"message": "Here's your interdimensional portal."})
```

This is handled automatically by FastAPI when the return type annotation is a `Response` class or subclass.

## Annotate a Response Subclass

```python
async def get_teleport() -> RedirectResponse:
    return RedirectResponse(url="https://www.youtube.com/watch?v=dQw4w9WgXcQ")
```

## Invalid Return Type Annotations

This fails because the type annotation is a union between `Response` and `dict`, not a single Pydantic type or Response subclass:

```python
async def get_portal(teleport: bool = False) -> Response | dict:
    if teleport:
        return RedirectResponse(url="https://www.youtube.com/watch?v=dQw4w9WgXcQ")
    return {"message": "Here's your interdimensional portal."}
```

## Disable Response Model

To allow custom union return type annotations, set `response_model=None`:

```python
@app.get("/portal", response_model=None)
async def get_portal(teleport: bool = False) -> Response | dict:
    if teleport:
        return RedirectResponse(url="https://www.youtube.com/watch?v=dQw4w9WgXcQ")
    return {"message": "Here's your interdimensional portal."}
```

## `model_dump()` and Union Responses

- **`pydantic_model_name.model_dump()`** — returns a dict with the model's data.
- Adding extra keyword arguments into that dict:

```python
UserInDB(**user_in.model_dump(), hashed_password=hashed_password)
```

### Union Response Model

Declare a response as the Union of two or more types (the response can be any of them):

```python
@app.get("/items/{item_id}", response_model=Union[PlaneItem, CarItem])
```

### Response with an Ordinary Dict

For declaring a response using just the type of the dict's keys, without a Pydantic model:

```python
@app.get("/keyword-weights/", response_model=dict[str, float])
```
