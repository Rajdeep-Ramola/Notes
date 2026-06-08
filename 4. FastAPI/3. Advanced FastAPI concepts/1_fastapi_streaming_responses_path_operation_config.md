# FastAPI — Stream Data, Path Operation Config, Additional Status Codes & Custom Responses

---

## Stream Data

To stream data structured as JSON, use **Stream JSON lines**. To stream pure binary data or strings, use `response_class=StreamingResponse` in the path operation function and use `yield` to send each chunk of data in turn.

### Streaming Bytes

If we want to stream bytes instead of strings:

```python

@app.get("/story/stream-bytes", response_class=StreamingResponse)
async def stream_story_bytes() -> AsyncIterable[bytes]:
    for line in message.splitlines():
        yield line.encode("utf-8")
```

### Custom Content-Type via Subclass

For our client (e.g. frontend) to know what type of data it is receiving, we can create a subclass of `StreamingResponse` that sets the `Content-Type` header to the type of data you're streaming. Then we use this new class in `response_class=PNGStreamingResponse` in our path operation function:

```python

from fastapi.responses import StreamingResponse

class PNGStreamingResponse(StreamingResponse):
    media_type = "image/png"
```

### `with` Block

The `with` block makes sure that the file-like object is closed after the generator function is done.

> **Note:** To avoid blocking the event loop, we can declare the path operation function with a regular `def`. That way FastAPI will run it on a threadpool worker to avoid blocking the main loop.

```python

@app.get("/image/stream-no-async", response_class=PNGStreamingResponse)
def stream_image_no_async() -> Iterable[bytes]:
    with read_image() as image_file:
        for chunk in image_file:
            yield chunk
```

### `yield from`

When iterating over something like a file-like object and yielding each item, you can also use `yield from` to yield each item directly and skip the `for` loop.

### Full Streaming Example

```python
import base64
from collections.abc import AsyncIterable, Iterable
from io import BytesIO
from fastapi import FastAPI
from fastapi.responses import StreamingResponse

# Base64-encoded PNG image (just some sample binary data)
image_base64 = "iVBORw0KGgoAAAANSUhEUgAA..."

# Decode base64 string into raw binary bytes
binary_image = base64.b64decode(image_base64)

def read_image() -> BytesIO:
    # BytesIO wraps raw bytes in a "file-like" object
    # so we can iterate over it as if it's a file on disk
    return BytesIO(binary_image)

app = FastAPI()

class PNGStreamingResponse(StreamingResponse):
    # This ensures the client (browser, app, etc.)
    # understands that the streamed bytes are a PNG image
    media_type = "image/png"

@app.get("/image/stream", response_class=PNGStreamingResponse)
async def stream_image() -> AsyncIterable[bytes]:
    # ✅ ASYNC generator function (because of async def + yield)
    # "with" ensures the file is CLOSED automatically after streaming finishes
    with read_image() as image_file:
        # ✅ STREAMING: sending data in chunks instead of all at once
        for chunk in image_file:
            # ✅ "yield" sends one chunk at a time to the client
            yield chunk

@app.get("/image/stream-no-async", response_class=PNGStreamingResponse)
def stream_image_no_async() -> Iterable[bytes]:
    # ✅ SYNC generator (uses normal def, not async def)
    with read_image() as image_file:
        for chunk in image_file:
            yield chunk

@app.get("/image/stream-no-async-yield-from", response_class=PNGStreamingResponse)
def stream_image_no_async_yield_from() -> Iterable[bytes]:
    with read_image() as image_file:
        # ✅ "yield from" = delegate iteration to another iterable
        # Instead of writing a loop:
        #   for chunk in image_file:
        #       yield chunk
        # We directly forward all chunks from image_file
        # ✅ Use "yield from" when you just want to pass through data
        # ✅ Use "for + yield" when you want to MODIFY chunks
        yield from image_file

@app.get("/image/stream-no-annotation", response_class=PNGStreamingResponse)
async def stream_image_no_annotation():
    # ✅ Type hints (like AsyncIterable[bytes]) are OPTIONAL
    with read_image() as image_file:
        for chunk in image_file:
            yield chunk

@app.get("/image/stream-no-async-no-annotation", response_class=PNGStreamingResponse)
def stream_image_no_async_no_annotation():
    # ✅ Simplest version: sync generator, no type hints
    with read_image() as image_file:
        for chunk in image_file:
            yield chunk
```

---

## Path Operation Advanced Configuration

### `operationId`

A unique identifier assigned to each endpoint (path operation), used by the docs schema:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/", operation_id="some_specific_id_you_define")
async def read_items():
    return [{"item_id": "Foo"}]
```

---

## Additional Status Codes

If we want to return additional status codes beyond the main one, we can do that by returning a `Response` directly (like a `JSONResponse`) and setting the additional status code directly:

```python
from typing import Annotated
from fastapi import Body, FastAPI, status
from fastapi.responses import JSONResponse

app = FastAPI()

items = {"foo": {"name": "Fighters", "size": 6}, "bar": {"name": "Tenders", "size": 3}}

@app.put("/items/{item_id}")
async def upsert_item(
    item_id: str,
    name: Annotated[str | None, Body()] = None,
    size: Annotated[int | None, Body()] = None,
):
    if item_id in items:
        item = items[item_id]
        item["name"] = name
        item["size"] = size
        return item
    else:
        item = {"name": name, "size": size}
        items[item_id] = item
        return JSONResponse(status_code=status.HTTP_201_CREATED, content=item)
```

---

## Return a Response Directly

You cannot put a Pydantic model in a `JSONResponse` without first converting it to a dict with all the data types (like `datetime`, `UUID`, etc.) converted to JSON-compatible types.

For those cases, use `jsonable_encoder` to convert your data before passing it to a response:

```python
from datetime import datetime
from fastapi import FastAPI
from fastapi.encoders import jsonable_encoder
from fastapi.responses import JSONResponse
from pydantic import BaseModel

class Item(BaseModel):
    title: str
    timestamp: datetime
    description: str | None = None

app = FastAPI()

@app.put("/items/{id}")
def update_item(id: str, item: Item):
    json_compatible_item_data = jsonable_encoder(item)
    return JSONResponse(content=json_compatible_item_data)
```

### Return a Custom Response (e.g. XML)

To return the item directly as a custom media type:

```python
from fastapi import FastAPI, Response

app = FastAPI()

@app.get("/legacy/")
def get_legacy_data():
    data = """<?xml version="1.0"?>
    <shampoo>
    <Header>
        Apply shampoo here.
    </Header>
    <Body>
        You'll have to use soap here.
    </Body>
    </shampoo>
    """
    return Response(content=data, media_type="application/xml")
```
