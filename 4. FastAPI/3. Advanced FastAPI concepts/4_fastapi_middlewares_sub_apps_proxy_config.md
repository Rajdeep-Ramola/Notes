# FastAPI — Built-in Middlewares, Sub-Application Mounts & Behind a Proxy

---

## Built-in Middlewares

### HTTPSRedirectMiddleware

Enforces that all incoming requests must either be `https` or `wss`:

```python
from fastapi import FastAPI
from fastapi.middleware.httpsredirect import HTTPSRedirectMiddleware

app = FastAPI()

app.add_middleware(HTTPSRedirectMiddleware)

@app.get("/")
async def main():
    return {"message": "Hello World"}
```

---

### TrustedHostMiddleware

Enforces that all incoming requests have a correctly set `Host` header, in order to guard against HTTP Host Header attacks:

```python
from fastapi import FastAPI
from fastapi.middleware.trustedhost import TrustedHostMiddleware

app = FastAPI()

app.add_middleware(
    TrustedHostMiddleware, allowed_hosts=["example.com", "*.example.com"]
)

@app.get("/")
async def main():
    return {"message": "Hello World"}
```

**Supported arguments:**

- `allowed_hosts` — A list of domain names that should be allowed as hostnames.
- `www_redirect` — If set to `True`, requests to non-www versions of the allowed hosts will be redirected to their www counterparts.

---

### GZipMiddleware

Handles GZip responses for any request that includes `"gzip"` in the `Accept-Encoding` header:

```python
from fastapi import FastAPI
from fastapi.middleware.gzip import GZipMiddleware

app = FastAPI()

app.add_middleware(GZipMiddleware, minimum_size=1000, compresslevel=5)

@app.get("/")
async def main():
    return "somebigcontent"
```

---

## Sub-Application Mounts

Mounting a sub-application under a path prefix:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/app")
def read_main():
    return {"message": "Hello World from main app"}

subapi = FastAPI()

@subapi.get("/sub")
def read_sub():
    return {"message": "Hello World from sub API"}

app.mount("/subapi", subapi)
```

---

## Behind a Proxy

### What is a Proxy?

A **proxy** is a middle service in front of our FastAPI app (backend) that handles requests, adds important info, and forwards them correctly. It typically:

- Handles HTTPS termination
- Acts as a gateway

### Why Proxy Headers Matter

**Scenario 1 — Without proxy headers:**

1. User visits `https://mysite.com/items`
2. FastAPI sees `http://internal-ip:port/items`
3. FastAPI redirects to `http://internal-ip:port/items/`
4. Browser tries to follow the redirect, but that IP is internal and not available on the open internet
5. Redirect fails → user sees an error

**Scenario 2 — Without proxy headers (localhost):**

1. User visits `https://mysite.com/items`
2. FastAPI receives `http://127.0.0.1:8000/items`
3. App thinks it's running over HTTP on localhost (not secure)

The proxy adds headers to prevent this:

| Header | Value |
|--------|-------|
| `X-Forwarded-For` | Original user IP |
| `X-Forwarded-Proto` | `https` |
| `X-Forwarded-Host` | `mysite.com` |

### Enable Proxy Forwarded Headers

Through `--forward-allow-ips`, we tell FastAPI to trust requests coming from specific IPs and then read the forwarded headers.

### Root Path

If our app is served under a prefix (e.g. `https://myapp.com/api/v1/items`) but our code handles `/items`, the proxy strips `/api/v1` before forwarding the request to FastAPI.

We can set the root path in our FastAPI app:

```python
from fastapi import FastAPI, Request

app = FastAPI(root_path="/api/v1")

@app.get("/app")
def read_main(request: Request):
    return {
        "message": "Hello World",
        "root_path": request.scope.get("root_path")
    }
```
