# 🚀 Running Guardrailed Inference

> Run guardrailed LLM inference using the Python API or the Guardrails API server.

---

## 📑 Table of Contents

1. [Python API](#1-python-api)
2. [Guardrails API Server](#2-guardrails-api-server)
3. [Actions Server](#3-actions-server)

---

## 1. Python API

Best for edge and embedded applications. Build guardrails directly into your Python application with no network overhead.

The Python API provides two core classes for running guardrails:

- `RailsConfig` — Loads and manages guardrails configuration from files or content.
- `LLMRails` — Main interface for generating responses with guardrails applied.

```python
from nemoguardrails import LLMRails, RailsConfig

# Load configuration from the config directory
config = RailsConfig.from_path("examples/configs")

# Create the LLMRails instance
rails = LLMRails(config)

# Generate a response
response = rails.generate(messages=[
    {
        "role": "user",
        "content": "What is the capital of France?",
        "config_id": "content_safety"
    }
])

print(response["content"])
```

### Different Ways to Generate Responses

| **API** | **Use Case** |
| --- | --- |
| generate() / generate_async() | Standard chat interactions with messages |
| stream_async() | Real-time token streaming |
| generate_events() / generate_events_async() | Low-level event control for custom integrations |

### LLMRails Parameters

| **Parameter** | **Type** | **Description** |
| --- | --- | --- |
| config | RailsConfig | The rails configuration |
| llm | BaseLLM │ BaseChatModel | Optional pre-configured LLM (overrides config) |
| verbose | bool | Enable verbose logging |

---

## 2. Guardrails API Server

Best for networked and multi-client applications. Instead of calling Python functions directly, you make HTTP requests to the server endpoints, making integration straightforward for any language or platform.

### Start the Server

Go to the parent directory of guardrails and run:

```bash
nemoguardrails server --config examples/configs
```

### View Chat UI

Open `http://localhost:8000` in your browser to access the built-in Chat UI for testing.

### Multi Config Mode

When the `--config` path points to a folder containing multiple sub-folders, each sub-folder with a `config.yml` file becomes an available configuration. The sub-folder name becomes the `config_id`.

For development:

```bash
nemoguardrails server --config ./configs --auto-reload
```

### CORS Configuration

To enable your guardrails server to receive requests from browser-based applications:

| **Environment Variable** | **Description** |
| --- | --- |
| NEMO_GUARDRAILS_SERVER_ENABLE_CORS | Set to `true` to enable CORS. Default: false. |
| NEMO_GUARDRAILS_SERVER_ALLOWED_ORIGINS | Comma-separated list of allowed origins. Default: *. |

---

## 3. Actions Server

The Actions Server enables you to run actions invoked from guardrails in a secure, isolated environment separate from the main guardrails server.

### Start the Actions Server

```bash
nemoguardrails actions-server [--port PORT]
```

### Action Server Endpoints

| **Method** | **Endpoint** | **Description** |
| --- | --- | --- |
| GET | /v1/actions/list | List all available actions |
| POST | /v1/actions/run | Execute an action |
