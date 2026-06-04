# Cookie and Header Parameters in FastAPI

## Cookie Parameters

To read data sent by client in HTTP cookies, import and use `Cookie`.

> If you don't use `Cookie`, the parameter will be interpreted as query parameters.

```python
from fastapi import Cookie, FastAPI

@app.get("/items/")
async def read_items(ads_id: Annotated[str | None, Cookie()] = None):
    return {"ads_id": ads_id}
```

## Header Parameters

Implemented by importing `Header`.

> **Note:** `Header` will convert the parameter names from underscore (`_`) to hyphen (`-`) to extract and document the headers.

```python
from fastapi import FastAPI, Header

@app.get("/items/")
async def read_items(user_agent: Annotated[str | None, Header()] = None):
    return {"User-Agent": user_agent}
```

### Duplicate Headers

The same header can have multiple values. Declare those using `list` in the type declaration.

```python
async def read_items(x_token: Annotated[list[str] | None, Header()] = None):
```

## Cookie Parameter Models

If you have a group of related cookies, you can create a Pydantic model to declare them. Just declare cookie parameters in the model, then declare that class as `Cookie`.

```python
class Cookies(BaseModel):
    session_id: str
    fatebook_tracker: str | None = None
    googall_tracker: str | None = None

@app.get("/items/")
async def read_items(cookies: Annotated[Cookies, Cookie()]):
    return cookies
```

> **Note:** To forbid extra cookies, use Pydantic's model configuration:
>
> ```python
> class Cookies(BaseModel):
>     model_config = {"extra": "forbid"}
> ```

## Header Parameter Models

For a group of related header parameters, use a Pydantic model similarly.

```python
class CommonHeaders(BaseModel):
    host: str
    save_data: bool
    if_modified_since: str | None = None
    traceparent: str | None = None
    x_tag: list[str] = []

@app.get("/items/")
async def read_items(headers: Annotated[CommonHeaders, Header()]):
    return headers
```
