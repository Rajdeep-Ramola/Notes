# 🔌 Integrating NeMo Guardrails in Our Application

> NeMo Guardrails can be integrated directly into your Python application using the Python SDK or via a standalone API server.

---

## 📑 Table of Contents

1. [Python SDK](#1-python-sdk)
2. [API Server](#2-api-server)

---

## 1. Python SDK

Add guardrails directly in your Python application.

```python
from nemoguardrails import LLMRails, RailsConfig

config = RailsConfig.from_path("./config")

rails = LLMRails(config)

response = rails.generate(
    messages=[{"role": "user", "content": "Hello!"}]
)
```

---

## 2. API Server

Set up a guardrails server after programming guardrails using the SDK.

```bash
nemoguardrails server --config ./config --port 8000
```
