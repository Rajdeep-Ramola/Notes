# FastAPI — WebSockets & Lifespan Events

---

## WebSockets

### How Different Connection Types Work

**Simple HTTP connection:**
Connection established → client sends request → server sends response → connection is closed.

**Polling:**
Connection is established periodically after being closed, and a new request is made for fresh data.

**WebSocket:**
Connection is not closed. Client sends requests and server sends responses over the same persistent connection (two-way communication). WebSockets work on the HTTP protocol.

### How a WebSocket Connection is Initialized

The client sends a request with a header indicating it wants to upgrade to a WebSocket connection. This opens (establishes) the persistent WebSocket connection.

> WebSocket connections are **stateful** and use file descriptors to maintain that connection, which means they consume some server resources.

### Basic WebSocket Endpoint

```python
from fastapi import FastAPI, WebSocket

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    while True:
        data = await websocket.receive_text()
        await websocket.send_text(f"Message text was: {data}")
```

### Sending and Receiving Messages

```python
@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    while True:
        data = await websocket.receive_text()
        await websocket.send_text(f"Message text was: {data}")
```

### WebSockets with Dependencies, Cookies, Headers, and Query Params

```python
async def get_cookie_or_token(
    websocket: WebSocket,
    session: Annotated[str | None, Cookie()] = None,
    token: Annotated[str | None, Query()] = None,
):
    if session is None and token is None:
        raise WebSocketException(code=status.WS_1008_POLICY_VIOLATION)
    return session or token

@app.websocket("/items/{item_id}/ws")
async def websocket_endpoint(
    *,
    websocket: WebSocket,
    item_id: str,
    q: int | None = None,
    cookie_or_token: Annotated[str, Depends(get_cookie_or_token)],
):
    await websocket.accept()
    while True:
        data = await websocket.receive_text()
        await websocket.send_text(
            f"Session cookie or query token value is: {cookie_or_token}"
        )
        if q is not None:
            await websocket.send_text(f"Query parameter q is: {q}")
        await websocket.send_text(f"Message text was: {data}, for item ID: {item_id}")
```

### Handling Disconnection and Multiple Clients

```python
class ConnectionManager:
    def __init__(self):
        self.active_connections: list[WebSocket] = []

    async def connect(self, websocket: WebSocket):
        await websocket.accept()
        self.active_connections.append(websocket)

    def disconnect(self, websocket: WebSocket):
        self.active_connections.remove(websocket)

    async def send_personal_message(self, message: str, websocket: WebSocket):
        await websocket.send_text(message)

    async def broadcast(self, message: str):
        for connection in self.active_connections:
            await connection.send_text(message)

manager = ConnectionManager()

@app.get("/")
async def get():
    return HTMLResponse(html)

@app.websocket("/ws/{client_id}")
async def websocket_endpoint(websocket: WebSocket, client_id: int):
    await manager.connect(websocket)
    try:
        while True:
            data = await websocket.receive_text()
            await manager.send_personal_message(f"You wrote: {data}", websocket)
            await manager.broadcast(f"Client #{client_id} says: {data}")
    except WebSocketDisconnect:
        manager.disconnect(websocket)
        await manager.broadcast(f"Client #{client_id} left the chat")
```

---

## Lifespan Events

Lifespan events let you define logic that should be executed:

- **Before** the application starts up (once, before receiving any requests)
- **After** the application is shutting down (once, after handling all requests)

This is very useful for setting up **resources** that are shared among requests and need to be **cleaned up** afterwards (e.g. database connections, ML models, thread pools).

### Example — Loading and Cleaning Up an ML Model

```python
from contextlib import asynccontextmanager
from fastapi import FastAPI

def fake_answer_to_everything_ml_model(x: float):
    return x * 42

ml_models = {}

@asynccontextmanager
async def lifespan(app: FastAPI):
    # Load the ML model (runs BEFORE the app starts receiving requests)
    ml_models["answer_to_everything"] = fake_answer_to_everything_ml_model
    yield
    # Clean up the ML models and release the resources (runs on shutdown)
    ml_models.clear()

app = FastAPI(lifespan=lifespan)

@app.get("/predict")
async def predict(x: float):
    result = ml_models["answer_to_everything"](x)
    return {"result": result}
```

### How Lifespan Works

- The code **before `yield`** is executed before the application starts receiving requests (startup).
- The code **after `yield`** is executed after the application has finished (shutdown).

The `@asynccontextmanager` decorator is used here. A context manager executes the code before `yield` when entering the `with` block, and the code after `yield` when exiting. In this case, we pass it to FastAPI via the `lifespan` parameter for it to manage automatically.
