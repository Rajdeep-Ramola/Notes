# Response Status Code, Form Data, and File Uploads in FastAPI

## Response Status Code

You can declare the HTTP status code for a response using the `status_code` parameter in the decorator method.

```python
@app.post("/items/", status_code=201)
```

## Form Data

When you need to receive form fields instead of JSON, use `Form`.

```python
from typing import Annotated
from fastapi import FastAPI, Form

app = FastAPI()

@app.post("/login/")
async def login(username: Annotated[str, Form()], password: Annotated[str, Form()]):
    return {"username": username}
```

### Form Models

You can use Pydantic models to declare form fields in FastAPI. Declare a Pydantic model with the fields, then declare the parameter as `Form`.

```python
from fastapi import FastAPI, Form

class FormData(BaseModel):
    username: str
    password: str

@app.post("/login/")
async def login(data: Annotated[FormData, Form()]):
    return data
```

## Request Files

Define files to be uploaded by the client using `File`. File parameters are created the same way as `Body` or `Form`.

If you declare the type of your path operation function parameter as `bytes`, FastAPI will read the file and you will receive the contents as `bytes` (the whole content will be stored in memory).

For larger files, use `UploadFile` — contents are stored on disk.

```python
from typing import Annotated
from fastapi import FastAPI, File, UploadFile

app = FastAPI()

@app.post("/files/")
async def create_file(file: Annotated[bytes, File()]):
    return {"file_size": len(file)}

@app.post("/uploadfile/")
async def create_upload_file(file: UploadFile):
    return {"filename": file.filename}
```

### UploadFile Attributes

| Attribute | Type | Description |
|---|---|---|
| `filename` | `str` | Original file name that was uploaded (e.g. `myimage.jpg`) |
| `content_type` | `str` | MIME type / media type (e.g. `image/jpeg`) |
| `file` | `SpooledTemporaryFile` | The actual Python file-like object, passable to other functions or libraries |

### UploadFile Async Methods

| Method | Description |
|---|---|
| `write(data)` | Writes data (`str` or `bytes`) to the file |
| `read(size)` | Reads `size` (`int`) bytes/characters of the file |
| `seek(offset)` | Goes to the byte position `offset` (`int`) in the file |
