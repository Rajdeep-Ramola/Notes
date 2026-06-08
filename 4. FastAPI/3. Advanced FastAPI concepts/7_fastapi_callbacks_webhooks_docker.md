# OpenAPI Callbacks & Webhooks — FastAPI Notes

---

## OpenAPI Callbacks

A **callback** is a process that happens when our API app calls an external API. After an operation completes, FastAPI documents that the server will send a POST request back to a client-provided URL.

**Example:**

```python
from fastapi import APIRouter, FastAPI
from pydantic import BaseModel, HttpUrl

app = FastAPI()

# Request model for creating an invoice
class Invoice(BaseModel):
    id: str
    title: str | None = None
    customer: str
    total: float

# Model representing the callback event payload
# that will be sent to the external API
class InvoiceEvent(BaseModel):
    description: str
    paid: bool

# Expected response model for the callback endpoint
class InvoiceEventReceived(BaseModel):
    ok: bool

# Router used specifically for defining callback operations
invoices_callback_router = APIRouter()

@invoices_callback_router.post(
    # Callback URL template:
    # FastAPI documents that the API provider will send a POST request
    # to the client-provided callback URL after invoice processing.
    "{$callback_url}/invoices/{$request.body.id}",
    response_model=InvoiceEventReceived,
)
def invoice_notification(body: InvoiceEvent):
    # This endpoint describes the structure of the callback request.
    # It is mainly used for OpenAPI documentation purposes.
    pass

@app.post(
    "/invoices/",
    # Register callback routes with this endpoint.
    # This tells FastAPI that this API operation may trigger
    # asynchronous callback requests to an external service.
    callbacks=invoices_callback_router.routes,
)
def create_invoice(invoice: Invoice, callback_url: HttpUrl | None = None):
    """
    Create an invoice.

    This will (let's imagine) let the API user (some external developer) create an
    invoice.

    And this path operation will:
    * Send the invoice to the client.
    * Collect the money from the client.
    * Send a notification back to the API user (the external developer), as a callback.
        * At this point is that the API will somehow send a POST request to the
            external API with the notification of the invoice event
            (e.g. "payment successful").
    """
    # Business logic would normally:
    # 1. Store the invoice
    # 2. Process payment
    # 3. Trigger the callback request to the provided callback_url
    return {"msg": "Invoice received"}
```

---

## OpenAPI Webhooks

Instead of the normal process where users send requests to our API, our API can send requests to their API or system. This is called a **webhook**.

**Example:**

```python
from datetime import datetime
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

# Data model for the webhook payload
# sent when a user creates a new subscription
class Subscription(BaseModel):
    username: str
    monthly_fee: float
    start_date: datetime

# Define a webhook event named "new-subscription"
# This documents that external systems can receive
# webhook notifications for this event.
@app.webhooks.post("new-subscription")
def new_subscription(body: Subscription):
    """
    When a new user subscribes to your service we'll send you a POST request with this
    data to the URL that you register for the event `new-subscription` in the dashboard.
    """
    # The `body` contains the webhook payload structure
    # that will be sent to subscriber applications.

# Regular API endpoint (not a webhook)
@app.get("/users/")
def read_users():
    return ["Rick", "Morty"]
```

---

## Advanced Python Types

If for some reason we can't use `|`, we can use `Union` from `typing`:

```python
from typing import Union

def say_hi(name: Union[str, None]):
    print(f"Hi {name}!")
```

---

## FastAPI CLI

Run the application using:

```bash
fastapi dev
```

Alternate commands:

```bash
$ fastapi dev main.py
$ fastapi dev --entrypoint main:app
```

---

## FastAPI in Containers (Docker)

We can deploy a FastAPI application by building a Linux container image using Docker.

- **Container** — a lightweight way to package applications including all their dependencies and necessary files while keeping them isolated from other containers in the same system.
- **Container image** — a static version of all the files, environment variables, and the default command/program that should be present in a container. When a container is started from an image, any changes (files, env vars, etc.) exist only in that running container and do not persist in the underlying image.

### Building a Docker Image for FastAPI

After creating the app, create a `Dockerfile` in the same project directory:

```dockerfile
# Use the official Python 3.14 base image
# This image includes Python and common system dependencies
FROM python:3.14

# Set the working directory inside the container
# All subsequent commands will run from /code
WORKDIR /code

# Copy the dependency file into the container
# This allows Docker to cache dependency installation layers
COPY ./requirements.txt /code/requirements.txt

# Install Python dependencies from requirements.txt
# --no-cache-dir reduces image size by avoiding pip cache storage
# --upgrade ensures latest compatible package versions are installed
RUN pip install --no-cache-dir --upgrade -r /code/requirements.txt

# Copy the application source code into the container
COPY ./app /code/app

# Start the FastAPI application when the container launches
# The app will run on port 80 inside the container
CMD ["fastapi", "run", "app/main.py", "--port", "80"]
```

### Use CMD in Exec Form

```dockerfile
# ✅ Do this
CMD ["fastapi", "run", "app/main.py", "--port", "80"]
```

If using a proxy:

```dockerfile
CMD ["fastapi", "run", "app/main.py", "--proxy-headers", "--port", "80"]
```

### Build and Run

```bash
# Build the Docker image
docker build -t image_name .

# Run a container
docker run -d container_name -p 80:80 image_name
```

### Single-File App Dockerfile

```dockerfile
FROM python:3.14

WORKDIR /code

COPY ./requirements.txt /code/requirements.txt

RUN pip install --no-cache-dir --upgrade -r /code/requirements.txt

COPY ./main.py /code/

CMD ["fastapi", "run", "main.py", "--port", "80"]
```
