# Server-Sent Events, Background Tasks, Metadata in Tags & Static Files

## Server-Sent Events (SSE)

Server-Sent Events is a standard for streaming data from server to client over HTTP. Each event is a small text block with fields separated by blank lines.

```
data: {"name": "Portal Gun", "price": 999.99}

data: {"name": "Plumbus", "price": 32.99}
```

### Streaming SSE with FastAPI

Use `yield` in the path operation and set `response_class=EventSourceResponse`. Also import `EventSourceResponse` from `fastapi.sse`.

If we declare the return type as `AsyncIterable[Item]`, FastAPI will use it to validate, document, and serialize the data using Pydantic.

```python
from collections.abc import AsyncIterable, Iterable
from fastapi import FastAPI
from fastapi.sse import EventSourceResponse
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

@app.get("/items/stream", response_class=EventSourceResponse)  # SSE endpoint
async def sse_items() -> AsyncIterable[Item]:
    for item in items:
        yield item  # Each yield sends an SSE event

@app.get("/items/stream-no-async", response_class=EventSourceResponse)  # SSE endpoint (sync)
def sse_items_no_async() -> Iterable[Item]:
    for item in items:
        yield item

@app.get("/items/stream-no-annotation", response_class=EventSourceResponse)
async def sse_items_no_annotation():
    for item in items:
        yield item  # Async generator inferred

@app.get("/items/stream-no-async-no-annotation", response_class=EventSourceResponse)
def sse_items_no_async_no_annotation():
    for item in items:
        yield item  # Sync generator inferred
```

### `ServerSentEvent` — Custom SSE Fields

If we want SSE fields like `event`, `id`, `retry`, or `comment`, we can yield `ServerSentEvent` objects instead of plain data:

```python
from collections.abc import AsyncIterable
from fastapi import FastAPI
from fastapi.sse import EventSourceResponse, ServerSentEvent
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    price: float

items = [
    Item(name="Plumbus", price=32.99),
    Item(name="Portal Gun", price=999.99),
    Item(name="Meeseeks Box", price=49.99),
]

@app.get("/items/stream", response_class=EventSourceResponse)
async def stream_items() -> AsyncIterable[ServerSentEvent]:
    yield ServerSentEvent(comment="stream of item updates")
    for i, item in enumerate(items):
        yield ServerSentEvent(data=item, event="item_update", id=str(i + 1), retry=5000)
```

> **Note:** To send data without JSON encoding, use `raw_data` instead of `data`:
> ```python
> yield ServerSentEvent(raw_data=log_line)
> ```

### Last-Event-ID — Resuming Streams

If the connection drops, the browser sends the last received `id` in the `Last-Event-ID` header. The application reads it as a header parameter and uses it to resume the stream from where the client left off.

```python
from collections.abc import AsyncIterable
from typing import Annotated
from fastapi import FastAPI, Header
from fastapi.sse import EventSourceResponse, ServerSentEvent
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    price: float

items = [
    Item(name="Plumbus", price=32.99),
    Item(name="Portal Gun", price=999.99),
    Item(name="Meeseeks Box", price=49.99),
]

@app.get("/items/stream", response_class=EventSourceResponse)
async def stream_items(
    # Reads the "Last-Event-ID" header sent by the client
    # Represents the last event ID the client successfully received
    last_event_id: Annotated[int | None, Header()] = None,
) -> AsyncIterable[ServerSentEvent]:
    # Determine where to resume streaming:
    # - If last_event_id is provided → start from the next event (last_event_id + 1)
    # - If not → start from the beginning (index 0)
    start = last_event_id + 1 if last_event_id is not None else 0
    for i, item in enumerate(items):
        # Skip all events already sent to the client
        if i < start:
            continue
        # Yield events starting from the calculated index
        yield ServerSentEvent(data=item, id=str(i))
```

> **Note:** SSE works with any HTTP method, not just POST.

### SSE Best Practices

- Send a **"keep alive" ping comment** every 15 seconds when there hasn't been any message, to prevent some proxies from closing the connection.
- Set the `Cache-Control: no-cache` header to **prevent caching** of the stream.
- Set the `X-Accel-Buffering: no` header to **prevent buffering** in proxies like Nginx.

---

## Background Tasks

Tasks which run **after returning a response**. Useful for operations that need to happen after a request, but the client doesn't need to wait for completion before receiving the response.

**Example use case:** Email notification sent after performing an action.

### `BackgroundTasks`

Import `BackgroundTasks` and define a parameter in your path operation function with a type declaration of `BackgroundTasks`:

```python
from fastapi import BackgroundTasks, FastAPI

app = FastAPI()

def write_notification(email: str, message=""):
    with open("log.txt", mode="w") as email_file:
        content = f"notification for {email}: {message}"
        email_file.write(content)

@app.post("/send-notification/{email}")
async def send_notification(email: str, background_tasks: BackgroundTasks):
    background_tasks.add_task(write_notification, email, message="some notification")
    return {"message": "Notification sent in the background"}
```

`.add_task()` receives as arguments:
- A **task function** to be run in the background (`write_notification`)
- Any **positional arguments** that should be passed to the task function (`email`)
- Any **keyword arguments** that should be passed to the task function (`message="some notification"`)

### Background Tasks in Dependency Injection

`BackgroundTasks` also works with the dependency injection system. You can declare a parameter of type `BackgroundTasks` at multiple levels: in a path operation function, in a dependency, in a sub-dependency, etc.

---

## Metadata in Tags

```python
from fastapi import FastAPI

tags_metadata = [
    {
        "name": "users",
        "description": "Operations with users. The **login** logic is also here.",
    },
    {
        "name": "items",
        "description": "Manage items. So _fancy_ they have their own docs.",
        "externalDocs": {
            "description": "Items external docs",
            "url": "https://fastapi.tiangolo.com/",
        },
    },
]

app = FastAPI(openapi_tags=tags_metadata)

@app.get("/users/", tags=["users"])
async def get_users():
    return [{"name": "Harry"}, {"name": "Ron"}]

@app.get("/items/", tags=["items"])
async def get_items():
    return [{"name": "wand"}, {"name": "flying broom"}]
```

> **Note:** The order of each tag metadata dictionary also defines the order shown in the docs UI.

---

## Static Files

We can serve static files automatically from a directory using `StaticFiles`:

```python
from fastapi import FastAPI
from fastapi.staticfiles import StaticFiles

app = FastAPI()

app.mount("/static", StaticFiles(directory="static"), name="static")
```

**Mounting concept** — adding a complete, independent application at a specific path that then takes care of handling all the sub-paths.

- The first `"/static"` refers to the sub-path this sub-application will be mounted on. Any path starting with `/static` will be handled by it.
- `directory="static"` refers to the name of the directory that contains your static files.
- `name="static"` gives it a name that can be used internally by FastAPI.

All these parameters can be different from `"static"` — adjust them to your application's needs.
