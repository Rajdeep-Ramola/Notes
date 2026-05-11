# 📋 Logging

> Inspect and debug NeMo Guardrails generation using verbose mode, explain methods, generation options, and tracing.

---

## 📑 Table of Contents

1. [Logging Methods](#1-logging-methods)
2. [Verbose Mode](#2-verbose-mode)
3. [Explain Method](#3-explain-method)
4. [Generation Options: Log](#4-generation-options-log)
5. [Tracing Guardrails](#5-tracing-guardrails)
6. [Adapter Configurations](#6-adapter-configurations)

---

## 1. Logging Methods

| **Method** | **Use Case** |
| --- | --- |
| **Verbose Mode** | Real-time console logging during development |
| **Explain Method** | Quick summary of the last generation |
| **Generation Options (log)** | Detailed structured logs returned with responses |
| **Output Variables** | Return specific context variables |

---

## 2. Verbose Mode

```python
from nemoguardrails import LLMRails, RailsConfig

config = RailsConfig.from_path("path/to/config")

rails = LLMRails(config, verbose=True)
```

Output gives detailed information about: LLM calls, rail activations, action executions, and flow transitions.

---

## 3. Explain Method

```python
response = rails.generate(messages=[
    {"role": "user", "content": "Hello!"}
])

info = rails.explain()

info.print_llm_calls_summary()
```

The `ExplainInfo` object provides methods to inspect LLM calls summary, Colang history, and generated events.

---

## 4. Generation Options: Log

```python
response = rails.generate(
    messages=[{"role": "user", "content": "Hello!"}],
    options={
        "log": {
            "activated_rails": True,
            "llm_calls": True,
            "internal_events": True,
            "colang_history": True
        }
    }
)
```

| **Option** | **Description** |
| --- | --- |
| activated_rails | Detailed information about rails activated during generation |
| llm_calls | Information about all LLM calls (prompt, completion, tokens, timing) |
| internal_events | Array of internal generated events |
| colang_history | Conversation history in Colang format |

---

## 5. Tracing Guardrails

```python
# trace_example.py

from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor, ConsoleSpanExporter
from opentelemetry.sdk.resources import Resource
from nemoguardrails import LLMRails, RailsConfig

# Configure OpenTelemetry
resource = Resource.create({"service.name": "guardrails-quickstart"})
tracer_provider = TracerProvider(resource=resource)
trace.set_tracer_provider(tracer_provider)
tracer_provider.add_span_processor(BatchSpanProcessor(ConsoleSpanExporter()))

# Configure guardrails with tracing
config_yaml = """
models:
  - type: main
    engine: openai
    model: gpt-4o-mini

rails:
  config:
    streaming: true

tracing:
  enabled: true
  adapters:
    - name: OpenTelemetry

"""

config = RailsConfig.from_content(yaml_content=config_yaml)
rails = LLMRails(config)

response = rails.generate(messages=[{"role": "user", "content": "Hello!"}])

print(f"Response: {response}")
```

---

## 6. Adapter Configurations

| **Adapter Type** | **Use Case** | **Configuration** |
| --- | --- | --- |
| [FileSystem](https://docs.nvidia.com/nemo/guardrails/latest/observability/tracing/adapter-configurations.html) | Development, debugging, local logging | filepath: "./logs/traces.jsonl" |
| [OpenTelemetry](https://docs.nvidia.com/nemo/guardrails/latest/observability/tracing/adapter-configurations.html) | Production, monitoring platforms, distributed systems | Requires SDK configuration |
| [Custom](https://docs.nvidia.com/nemo/guardrails/latest/observability/tracing/adapter-configurations.html) | Specialized backends or formats | Implement InteractionLogAdapter |

### FileSystem Adapter

```yaml
tracing:
  enabled: true
  adapters:
    - name: FileSystem
      filepath: "./logs/traces.jsonl"
```

### OpenTelemetry Adapter

```yaml
tracing:
  enabled: true
  adapters:
    - name: OpenTelemetry
```

### Custom Adapter

```python
from nemoguardrails.tracing.adapters.base import InteractionLogAdapter

class MyCustomAdapter(InteractionLogAdapter):
    name = "MyCustomAdapter"

    def __init__(self, custom_option: str):
        self.custom_option = custom_option

    def transform(self, interaction_log):
        # Transform logic for your backend
        pass
```

Register adapter in `config.py`:

```python
from nemoguardrails.tracing.adapters.registry import register_log_adapter

register_log_adapter(MyCustomAdapter, "MyCustomAdapter")
```

Use adapter in `config.yml`:

```yaml
tracing:
  enabled: true
  adapters:
    - name: MyCustomAdapter
      custom_option: "value"
```
