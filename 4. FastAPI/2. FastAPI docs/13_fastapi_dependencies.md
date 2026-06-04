# Dependencies

## Dependency Injection

Dependency Injection is a way of giving an object what it needs **from the outside**, instead of letting it create those things itself.

It is useful when we have shared logic and have to minimize code repetition.

```python
from typing import Annotated
from fastapi import Depends, FastAPI

app = FastAPI()

async def common_parameters(q: str | None = None, skip: int = 0, limit: int = 100):
    return {"q": q, "skip": skip, "limit": limit}

@app.get("/items/")
async def read_items(commons: Annotated[dict, Depends(common_parameters)]):
    return commons

@app.get("/users/")
async def read_users(commons: Annotated[dict, Depends(common_parameters)]):
    return commons
```

- We use `Depends` in the same way we use `Body`, `Query`, etc.
- `Depends` can have only a single parameter and that parameter must be a callable.

To further reduce code repetition, we can use shared annotated dependencies:

```python
async def common_parameters(q: str | None = None, skip: int = 0, limit: int = 100):
    return {"q": q, "skip": skip, "limit": limit}

CommonsDep = Annotated[dict, Depends(common_parameters)]

@app.get("/items/")
async def read_items(commons: CommonsDep):
    return commons
```

> **Note:** You can declare dependencies with `async def` inside of normal `def` path operation functions, or `def` dependencies inside of `async def` path operation functions, etc.

---

## Classes as Dependencies

Dependencies can be declared as functions, but there are other ways too. The only condition is that a dependency should be **callable** — anything that Python can call like a function.

If we pass a "callable" as a dependency in FastAPI, it will analyze the parameters for that callable, and process them in the same way as the parameters for a path operation function.

```python
class CommonQueryParams:
    def __init__(self, q: str | None = None, skip: int = 0, limit: int = 100):
        self.q = q
        self.skip = skip
        self.limit = limit

@app.get("/items/")
async def read_items(commons: Annotated[CommonQueryParams, Depends(CommonQueryParams)]):
    response = {}
    if commons.q:
        response.update({"q": commons.q})
    items = fake_items_db[commons.skip : commons.skip + commons.limit]
    response.update({"items": items})
    return response
```

Instead of using `CommonQueryParams` twice, we can write:

```python
commons: Annotated[CommonQueryParams, Depends()]
```

---

## Sub-Dependencies

We can create dependencies that have sub-dependencies. FastAPI will handle them automatically.

```python
def query_extractor(q: str | None = None):
    return q

def query_or_cookie_extractor(
    q: Annotated[str, Depends(query_extractor)],
    last_query: Annotated[str | None, Cookie()] = None,
):
    if not q:
        return last_query
    return q

@app.get("/items/")
async def read_query(
    query_or_default: Annotated[str, Depends(query_or_cookie_extractor)],
):
    return {"q_or_cookie": query_or_default}
```

**Using the same dependency multiple times:** If multiple dependencies have a common sub-dependency, FastAPI will know to call that sub-dependency only once per request. You can change that using the parameter `use_cache=False` when using `Depends`.

---

## Dependencies in Path Operation Decorators

Sometimes we don't need the return value of a dependency inside our path operation function, or the dependency itself doesn't return a value. In those cases, instead of using `Depends`, we can add a list of dependencies to our function decorator:

```python
async def verify_token(x_token: Annotated[str, Header()]):
    if x_token != "fake-super-secret-token":
        raise HTTPException(status_code=400, detail="X-Token header invalid")

async def verify_key(x_key: Annotated[str, Header()]):
    if x_key != "fake-super-secret-key":
        raise HTTPException(status_code=400, detail="X-Key header invalid")
    return x_key

@app.get("/items/", dependencies=[Depends(verify_token), Depends(verify_key)])
async def read_items():
    return [{"item": "Foo"}, {"item": "Bar"}]
```

These dependencies will be executed/solved the same way as normal dependencies, but their value (if they return any) won't be passed to the path operation function.

---

## Global Dependencies

For dependencies that we want applied to the whole application:

```python
async def verify_token(x_token: Annotated[str, Header()]):
    if x_token != "fake-super-secret-token":
        raise HTTPException(status_code=400, detail="X-Token header invalid")

async def verify_key(x_key: Annotated[str, Header()]):
    if x_key != "fake-super-secret-key":
        raise HTTPException(status_code=400, detail="X-Key header invalid")
    return x_key

app = FastAPI(dependencies=[Depends(verify_token), Depends(verify_key)])

@app.get("/items/")
async def read_items():
    return [{"item": "Portal Gun"}, {"item": "Plumbus"}]
```

---

## Dependencies with `yield`

For implementing dependencies which do some extra steps after finishing, use `yield` instead of `return`, and write the extra steps (code) after:

```python
async def get_db():
    db = DBSession()
    try:
        yield db
    finally:
        db.close()
```

The code following `yield` is executed after the response.

---

## Dependencies with `yield` and `except`

If we catch an exception using `except` in a dependency with `yield` and we don't raise it again (or raise a new exception), FastAPI won't be able to notice there was an exception.

---

## Context Managers

Any Python object that we can use in a `with` statement. When we create a dependency with `yield`, FastAPI will internally create a context manager for it.

We can create context managers by creating a class with two methods: `__enter__()` and `__exit__()`.

We can use them inside FastAPI dependencies with `yield` by using `with` or `async with` statements inside of the dependency function:

```python
class MySuperContextManager:
    def __init__(self):
        self.db = DBSession()

    def __enter__(self):
        return self.db

    def __exit__(self, exc_type, exc_value, traceback):
        self.db.close()

async def get_db():
    with MySuperContextManager() as db:
        yield db
```
