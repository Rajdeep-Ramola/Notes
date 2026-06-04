# FastAPI — Request Body and Query String Validations

## Request Body

**Request Body** — when we need to send data from a client to our API, we send it as a request body. Our API almost always has to send a response body.

To declare a request body, we generally use Pydantic models:

```python
from fastapi import FastAPI
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

app = FastAPI()

@app.post("/items/")
async def create_item(item: Item):
    return item
```

> The same as when declaring query parameters, when a model attribute has a default value, it is not required. Otherwise, it is required. Use `None` to make it just optional.

### Request Body + Path Parameters

FastAPI will recognize that the function parameters that match path parameters should be taken from the path, and that function parameters declared as Pydantic models should be taken from the request body:

```python
from fastapi import FastAPI
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

app = FastAPI()

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    return {"item_id": item_id, **item.model_dump()}
```

---

## Query Parameters and String Validations

### Additional Validation

Implement additional validation through `Query` and `Annotated`:

```python
from typing import Annotated
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(q: Annotated[str | None, Query(max_length=50)] = None):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

> We use `Annotated` so that we can have additional validation for the parameter using `Query`.

### More Validations

**`min_length`:**
```python
async def read_items(
    q: Annotated[str | None, Query(min_length=3, max_length=50)] = None,
):
```

**Adding regular expressions** via `pattern`:
```python
async def read_items(
    q: Annotated[
        str | None, Query(min_length=3, max_length=50, pattern="^fixedquery$")
    ] = None,
):
```

- `^` — starts with the following characters
- `fixedquery` — the exact value that has to be matched
- `$` — end of expression

> **Note:** Having a default value of any type, including `None`, makes the parameter optional (not required). When you need to declare a value as required while using `Query`, simply do not declare a default value.

### Receiving Multiple Values in a Query Parameter

```python
async def read_items(q: Annotated[list[str] | None, Query()] = None):
```

> **Note:** To declare a query parameter with a type of `list`, you need to explicitly use `Query`, otherwise it would be interpreted as a request body.

**With default values:**
```python
async def read_items(q: Annotated[list[str], Query()] = ["foo", "bar"]):
```

### Declaring More Metadata

**Add title:**
```python
async def read_items(
    q: Annotated[str | None, Query(title="Query string", min_length=3)] = None,
):
```

**Add description:**
```python
async def read_items(
    q: Annotated[
        str | None,
        Query(
            title="Query string",
            description="Query string for the items to search in the database that have a good match",
            min_length=3,
        ),
    ] = None,
):
```

**Alias parameter:**
```python
async def read_items(q: Annotated[str | None, Query(alias="item-query")] = None):
```

**Deprecating parameters** — to deprecate a parameter we don't want to use anymore:
```python
async def read_items(
    q: Annotated[
        str | None,
        Query(
            alias="item-query",
            title="Query string",
            description="Query string for the items to search in the database that have a good match",
            min_length=3,
            max_length=50,
            pattern="^fixedquery$",
            deprecated=True,
        ),
    ] = None,
):
```

**Exclude parameters from OpenAPI documentation:**
```python
@app.get("/items/")
async def read_items(
    hidden_query: Annotated[str | None, Query(include_in_schema=False)] = None,
):
```

### Custom Validation

Implementing custom validation through Pydantic's `AfterValidator`:

```python
import random
from typing import Annotated
from fastapi import FastAPI
from pydantic import AfterValidator

app = FastAPI()

data = {
    "isbn-9781529046137": "The Hitchhiker's Guide to the Galaxy",
    "imdb-tt0371724": "The Hitchhiker's Guide to the Galaxy",
    "isbn-9781439512982": "Isaac Asimov: The Complete Stories, Vol. 2",
}

def check_valid_id(id: str):
    if not id.startswith(("isbn-", "imdb-")):
        raise ValueError('Invalid ID format, it must start with "isbn-" or "imdb-"')
    return id

@app.get("/items/")
async def read_items(
    id: Annotated[str | None, AfterValidator(check_valid_id)] = None,
):
    if id:
        item = data.get(id)
    else:
        id, item = random.choice(list(data.items()))
    return {"id": id, "name": item}
```

> **Note:** If you need to do any type of validation that requires communicating with any **external component** (like a database or another API), you should instead use **FastAPI Dependencies**. These custom validators are for things that can be checked with **only** the **same data** provided in the request.
