# FastAPI — Custom Responses, Cookies, Headers & Advanced Dependencies

---

## Custom Responses

### HTML Response

```python
from fastapi import FastAPI
from fastapi.responses import HTMLResponse

app = FastAPI()

@app.get("/items/")
async def read_items():
    html_content = """
    <html>
        <head>
            <title>Some HTML in here</title>
        </head>
        <body>
            <h1>Look ma! HTML!</h1>
        </body>
    </html>
    """
    return HTMLResponse(content=html_content, status_code=200)
```

### Response Class Parameters

The `Response` class accepts the following parameters:

- `content` — A `str` or `bytes`.
- `status_code` — An `int` HTTP status code.
- `headers` — A `dict` of strings.
- `media_type` — A `str` giving the media type. E.g. `"text/html"`.

### Redirect Response

Returns an HTTP redirect:

```python
from fastapi import FastAPI
from fastapi.responses import RedirectResponse

app = FastAPI()

@app.get("/typer")
async def redirect_typer():
    return RedirectResponse("https://typer.tiangolo.com")

# OR

@app.get("/fastapi", response_class=RedirectResponse)
async def redirect_fastapi():
    return "https://fastapi.tiangolo.com"
```

### Streaming Response

Takes an async generator or a normal generator/iterator (a function with `yield`) and streams the response body:

```python
import anyio
from fastapi import FastAPI
from fastapi.responses import StreamingResponse

app = FastAPI()

async def fake_video_streamer():
    for i in range(10):
        yield b"some fake video bytes"
        await anyio.sleep(0)

@app.get("/")
async def main():
    return StreamingResponse(fake_video_streamer())
```

### File Response

Asynchronously streams a file as the response:

```python
from fastapi import FastAPI
from fastapi.responses import FileResponse

some_file_path = "large-video-file.mp4"
app = FastAPI()

@app.get("/")
async def main():
    return FileResponse(some_file_path)
```

---

## Response Cookies

Declare a parameter of type `Response` in the path operation function and then set cookies on the response object:

```python
from fastapi import FastAPI, Response

app = FastAPI()

@app.post("/cookie-and-object/")
def create_cookie(response: Response):
    response.set_cookie(key="fakesession", value="fake-cookie-session-value")
    return {"message": "Come to the dark side, we have cookies"}
```

---

## Response Headers

Declare a parameter of type `Response` in the path operation function and then set headers on that response object:

```python
from fastapi import FastAPI, Response

app = FastAPI()

@app.get("/headers-and-object/")
def get_headers(response: Response):
    response.headers["X-Cat-Dog"] = "alone in the world"
    return {"message": "Hello World"}
```

---

## Advanced Dependencies

### Parameterized Dependencies with `__call__`

Sometimes we want to set parameters to a dependency but also reuse that dependency with different parameters without writing multiple functions or classes.

**Example:** A dependency that checks if a query parameter contains some fixed content, where that fixed content is configurable.

To achieve this, use the `__call__` method — it lets us store an instance of a class in a variable and then use that variable as a function:

```python
from fastapi import Depends, FastAPI

app = FastAPI()

class FixedContentQueryChecker:
    def __init__(self, fixed_content: str):
        self.fixed_content = fixed_content

    def __call__(self, q: str = ""):
        if q:
            return self.fixed_content in q
        return False

checker = FixedContentQueryChecker("bar")

@app.get("/query-checker/")
async def read_query_check(fixed_content_included: Annotated[bool, Depends(checker)]):
    return {"fixed_content_in_query": fixed_content_included}
```

### Dependencies with `yield` and Scope

- Using `Depends(scope="function")`: the exit code after `yield` is executed right after the path operation function finishes, **before** the response is sent back to the client.
- Using `Depends(scope="request")` (the default): the exit code after `yield` is executed **after** the response is sent.
