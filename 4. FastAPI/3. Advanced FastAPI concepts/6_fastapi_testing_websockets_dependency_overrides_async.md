# FastAPI Testing — WebSockets, Dependency Overrides & Async Tests

---

## Testing WebSockets

We can use `TestClient` in a `with` statement to connect to a WebSocket endpoint and test it.

**Example:**

```python
from fastapi import FastAPI
from fastapi.testclient import TestClient
from fastapi.websockets import WebSocket

app = FastAPI()

@app.get("/")
async def read_main():
    return {"msg": "Hello World"}

@app.websocket("/ws")
async def websocket(websocket: WebSocket):
    # Accept the incoming WebSocket connection
    await websocket.accept()
    # Send a JSON message to the connected client
    await websocket.send_json({"msg": "Hello WebSocket"})
    # Close the WebSocket connection after sending the response
    await websocket.close()

def test_read_main():
    client = TestClient(app)
    # Test the HTTP GET endpoint
    response = client.get("/")
    # Verify the response status code
    assert response.status_code == 200
    # Verify the response payload
    assert response.json() == {"msg": "Hello World"}

def test_websocket():
    client = TestClient(app)
    # Create a test WebSocket connection to the "/ws" endpoint
    with client.websocket_connect("/ws") as websocket:
        # Receive JSON data sent from the WebSocket server
        data = websocket.receive_json()
        # Validate the received WebSocket message
        assert data == {"msg": "Hello WebSocket"}
```

---

## Testing Dependencies with Overrides

Sometimes we might want to override a dependency during testing.

**Example use case:** Suppose we have an external authentication provider that we call by sending it a token, and it returns an authenticated user. The provider might charge us per request. In this case, we can override the dependency that calls that provider with a custom dependency that returns a mock user — only for our tests.

We use `app.dependency_overrides` for this.

**Example:**

```python
from typing import Annotated
from fastapi import Depends, FastAPI
from fastapi.testclient import TestClient

app = FastAPI()

# Original dependency used by the API endpoints
async def common_parameters(q: str | None = None, skip: int = 0, limit: int = 100):
    return {"q": q, "skip": skip, "limit": limit}

@app.get("/items/")
async def read_items(commons: Annotated[dict, Depends(common_parameters)]):
    return {"message": "Hello Items!", "params": commons}

@app.get("/users/")
async def read_users(commons: Annotated[dict, Depends(common_parameters)]):
    return {"message": "Hello Users!", "params": commons}

client = TestClient(app)

# Override dependency used specifically during testing
# This replaces the original `common_parameters` dependency
# with controlled test values.
async def override_dependency(q: str | None = None):
    return {"q": q, "skip": 5, "limit": 10}

# Apply the dependency override globally for tests
# Any endpoint depending on `common_parameters`
# will now use `override_dependency` instead.
app.dependency_overrides[common_parameters] = override_dependency

def test_override_in_items():
    # Test endpoint using overridden dependency values
    response = client.get("/items/")
    # Verify successful response
    assert response.status_code == 200
    # Verify overridden dependency values are returned
    assert response.json() == {
        "message": "Hello Items!",
        "params": {"q": None, "skip": 5, "limit": 10},
    }

def test_override_in_items_with_q():
    # Test overridden dependency while passing query parameter
    response = client.get("/items/?q=foo")
    # Verify successful response
    assert response.status_code == 200
    # Ensure overridden values are still applied
    assert response.json() == {
        "message": "Hello Items!",
        "params": {"q": "foo", "skip": 5, "limit": 10},
    }

def test_override_in_items_with_params():
    # Even when skip and limit are passed in the request,
    # the dependency override forces test values instead.
    response = client.get("/items/?q=foo&skip=100&limit=200")
    # Verify successful response
    assert response.status_code == 200
    # Confirm dependency override takes precedence
    assert response.json() == {
        "message": "Hello Items!",
        "params": {"q": "foo", "skip": 5, "limit": 10},
    }
```

---

## Async Tests

If we want to call asynchronous functions in our tests, our test functions must also be asynchronous.

The marker `@pytest.mark.anyio` tells pytest that the test function should be called asynchronously.

**Example:**

```python
import pytest
from httpx import ASGITransport, AsyncClient
from .main import app

@pytest.mark.anyio
async def test_root():
    async with AsyncClient(
        transport=ASGITransport(app=app), base_url="http://test"
    ) as ac:
        response = await ac.get("/")
    assert response.status_code == 200
    assert response.json() == {"message": "Tomato"}
```
