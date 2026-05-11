# ⚡ Custom Actions

> Extend NeMo Guardrails with custom Python actions to implement complex logic, external API integrations, and shared resources.

---

## 📑 Table of Contents

1. [Creating Custom Actions](#1-creating-custom-actions)
2. [System Actions](#2-system-actions)
3. [Example Actions](#3-example-actions)
4. [Built-in Actions](#4-built-in-actions)
5. [Action Parameters](#5-action-parameters)
6. [Registering Actions](#6-registering-actions)
7. [Configuring Custom Initialization](#7-configuring-custom-initialization)
8. [LLM Providers](#8-llm-providers)
9. [Custom Embedding Providers](#9-custom-embedding-providers)
10. [Custom Configuration Data](#10-custom-configuration-data)

---

## 1. Creating Custom Actions

Use the `@action` decorator from `nemoguardrails.actions` to define custom actions.

### Decorator Parameters

| **Parameter** | **Type** | **Description** | **Default** |
| --- | --- | --- | --- |
| name | str | Custom name for the action | Function name |
| is_system_action | bool | Always run locally, bypassing the actions server | False |
| execute_async | bool | Don't block event processing while the action runs (Colang 2.x only) | False |
| output_mapping | Callable[[Any], bool] | Function to interpret the action result for blocking decisions | default_output_mapping |

```python
@action(name="validate_user_input")
async def check_input(text: str):
    """Validates user input."""
    return len(text) > 0
```

Call from Colang:

```colang
$is_valid = execute validate_user_input(text=$user_message)
```

---

## 2. System Actions

When `is_system_action=True`, the action always runs locally, even when an `actions_server_url` is configured. This is important for actions that need access to special parameters like `context`, `llm`, `config`, and `events`, which are only injected for locally-run actions.

---

## 3. Example Actions

### Input Validation Action

```python
from typing import Optional
from nemoguardrails.actions import action

@action(is_system_action=True)
async def check_input_length(context: Optional[dict] = None):
    """Ensure user input is not too long."""
    user_message = context.get("last_user_message", "")
    max_length = 1000

    if len(user_message) > max_length:
        return False  # Block the input

    return True  # Allow the input
```

### External API Action

```python
import aiohttp

@action(execute_async=True)
async def query_knowledge_base(query: str, top_k: int = 5):
    """Query an external knowledge base API."""
    async with aiohttp.ClientSession() as session:
        async with session.post(
            "https://api.example.com/search",
            json={"query": query, "limit": top_k}
        ) as response:
            data = await response.json()
            return data.get("results", [])
```

---

## 4. Built-in Actions

### Core Actions

| **Action** | **Description** |
| --- | --- |
| generate_user_intent | Generate the canonical form for the user utterance |
| generate_next_steps | Generate the next step in the conversation flow |
| generate_bot_message | Generate a bot message based on the desired intent |
| retrieve_relevant_chunks | Retrieve relevant chunks from the knowledge base |

### Guardrail-Specific Actions

| **Action** | **Description** |
| --- | --- |
| self_check_input | Check if user input should be accepted |
| self_check_output | Check if bot response should be allowed |
| self_check_facts | Verify factual accuracy of bot response |
| self_check_hallucination | Detect hallucinations in bot response |

### Sensitive Data Detection Actions

| **Action** | **Description** |
| --- | --- |
| detect_sensitive_data | Detect PII in text |
| mask_sensitive_data | Mask detected PII |

### Content Safety Actions

| **Action** | **Description** |
| --- | --- |
| llama_guard_check_input | LlamaGuard input moderation |
| llama_guard_check_output | LlamaGuard output moderation |
| content_safety_check_input | NVIDIA content safety model for input (requires model_name parameter) |
| content_safety_check_output | NVIDIA content safety model for output (requires model_name parameter) |

### Jailbreak Detection Actions

| **Action** | **Description** |
| --- | --- |
| jailbreak_detection_model | Detect jailbreak attempts using a trained classifier |
| jailbreak_detection_heuristics | Detect jailbreak attempts using heuristic checks |

---

## 5. Action Parameters

### Special Parameters

When you include these parameters in your action's function signature, they are automatically populated:

| **Parameter** | **Type** | **Description** |
| --- | --- | --- |
| context | dict | Context data available to the action |
| events | List[dict] | History of events in the conversation |
| llm | BaseLLM | Access to the LLM instance |
| config | RailsConfig | The full configuration instance |
| llm_task_manager | LLMTaskManager | Access to the LLM task manager for prompt rendering |
| state | State | The runtime state object (Colang 2.x only) |

### Context Parameter

```python
from typing import Optional
from nemoguardrails.actions import action

@action(is_system_action=True)
async def my_action(context: Optional[dict] = None):
    # Access context variables
    user_message = context.get("last_user_message")
    bot_message = context.get("bot_message")
    relevant_chunks = context.get("relevant_chunks")

    return True
```

### Common Context Variables

| **Variable** | **Description** | **Availability** |
| --- | --- | --- |
| last_user_message | The most recent user message | Always available after user input |
| bot_message | The current bot message (in output rails) | Available in output rails |
| last_bot_message | The previous bot message | Available after first bot response |
| relevant_chunks | Retrieved knowledge base chunks | Only after retrieve_relevant_chunks runs |
| user_intent | The canonical user intent | Only after generate_user_intent runs |
| bot_intent | The canonical bot intent | Only after generate_next_steps runs |

### Events Parameter

```python
from typing import List, Optional
from nemoguardrails.actions import action

@action()
async def analyze_conversation(events: Optional[List[dict]] = None):
    # Count user messages
    user_messages = [
        e for e in events
        if e.get("type") == "UtteranceUserActionFinished"
    ]
    return {"message_count": len(user_messages)}
```

### Event Types

| **Event Type** | **Description** |
| --- | --- |
| UtteranceUserActionFinished | User sent a message |
| StartUtteranceBotAction | Bot started responding |
| UtteranceBotActionFinished | Bot finished responding |
| StartInternalSystemAction | System action started |
| InternalSystemActionFinished | System action completed |
| UserIntent | User intent was determined |
| BotIntent | Bot intent was determined |

### LLM Parameter

```python
from typing import Optional
from langchain_core.language_models import BaseLLM
from nemoguardrails.actions import action

@action()
async def custom_llm_call(
    prompt: str,
    llm: Optional[BaseLLM] = None
):
    """Make a custom LLM call."""
    if llm is None:
        return "LLM not available"

    response = await llm.agenerate([prompt])
    return response.generations[0][0].text
```

Use cases for LLM access: custom prompt engineering, multiple LLM calls within a single action, and specialized text processing.

```python
@action()
async def summarize_and_validate(
    text: str,
    llm: Optional[BaseLLM] = None
):
    """Summarize text and validate the summary."""
    # First call: summarize
    summary_prompt = f"Summarize this text: {text}"
    summary = await llm.agenerate([summary_prompt])
    summary_text = summary.generations[0][0].text

    # Second call: validate
    validation_prompt = f"Is this summary accurate? {summary_text}"
    validation = await llm.agenerate([validation_prompt])

    return {
        "summary": summary_text,
        "validation": validation.generations[0][0].text
    }
```

### Config Parameter

```python
from typing import Optional
from nemoguardrails import RailsConfig
from nemoguardrails.actions import action

@action()
async def check_config_setting(config: Optional[RailsConfig] = None):
    """Access configuration settings."""
    # Access model configuration
    models = config.models
    main_model = next(
        (m for m in models if m.type == "main"),
        None
    )

    # Access custom config data
    custom_data = config.custom_data

    return {
        "model_engine": main_model.engine if main_model else None,
        "custom_data": custom_data
    }
```

### LLM Task Manager Parameter

```python
from typing import Optional
from nemoguardrails.actions import action
from nemoguardrails.llm.taskmanager import LLMTaskManager

@action()
async def custom_task(llm_task_manager: Optional[LLMTaskManager] = None):
    """Use the LLM task manager for prompt rendering."""
    pass
```

---

## 6. Registering Actions

### Registration Methods

| **Method** | **Description** | **Use Case** |
| --- | --- | --- |
| File-based | Actions in actions.py are auto-registered | Standard configurations |
| Programmatic | Register via LLMRails.register_action() | Dynamic registration |
| LangChain tools | Register LangChain tools as actions | Tool integration |
| Actions server | Remote action execution | Distributed systems |

### File-Based Registration

Actions defined in `actions.py` or the `actions/` package are automatically registered when the configuration is loaded.

For larger projects, organize actions into packages:

```
config/
├── config.yml
├── actions/
│   ├── __init__.py
│   ├── validation.py
│   ├── external.py
│   └── utils.py
└── rails/
    └── ...
```

### Programmatic Registration

Register actions dynamically using `LLMRails.register_action()`.

**Runtime configuration:**

```python
def setup_rails(environment: str):
    config = RailsConfig.from_path("config")
    rails = LLMRails(config)

    if environment == "production":
        rails.register_action(production_validator, "validate")
    else:
        rails.register_action(dev_validator, "validate")

    return rails
```

**Dependency injection:**

```python
class DatabaseService:
    async def query(self, sql: str):
        # Database query logic
        pass

db = DatabaseService()

async def db_query_action(query: str):
    return await db.query(query)

rails.register_action(db_query_action, name="query_database")
```

### Registration in config.py

```python
# config/config.py
from nemoguardrails import LLMRails

def init(app: LLMRails):
    """Custom initialization function."""
    # Register actions
    async def custom_action(param: str):
        return f"Custom: {param}"

    app.register_action(custom_action, name="custom_action")

    # Register action parameters
    db_connection = create_db_connection()
    app.register_action_param("db", db_connection)
```

### Providing Shared Resources to Actions

**config.py:**

```python
# config/config.py
def init(app: LLMRails):
    # Create shared resources
    http_client = aiohttp.ClientSession()
    cache = RedisCache()

    # Register as action parameters
    app.register_action_param("http_client", http_client)
    app.register_action_param("cache", cache)
```

**actions.py:**

```python
# config/actions.py
from nemoguardrails.actions import action

@action()
async def fetch_with_cache(
    url: str,
    http_client=None,  # Injected automatically
    cache=None         # Injected automatically
):
    # Check cache first
    cached = await cache.get(url)
    if cached:
        return cached

    # Fetch and cache
    response = await http_client.get(url)
    data = await response.json()
    await cache.set(url, data)
    return data
```

---

## 7. Configuring Custom Initialization

### When to Use config.py vs actions.py

| **Use Case** | **File** | **Reason** |
| --- | --- | --- |
| Register custom LLM provider | config.py | Must happen before LLMRails initialization |
| Register custom embedding provider | config.py | Must happen before LLMRails initialization |
| Initialize database connection | config.py | Shared resource, initialized once |
| Validate user input | actions.py | Called during request processing |
| Call external API | actions.py | Called during request processing |
| Custom guardrail logic | actions.py | Called from Colang flows |

### The Init Function

If `config.py` contains an `init` function, it is called during `LLMRails` initialization:

```python
from nemoguardrails import LLMRails

def init(app: LLMRails):
    # Initialize database connection
    db = DatabaseConnection()

    # Register as action parameter (available to all actions)
    app.register_action_param("db", db)
```

The `init` function can also be used for registering action parameters, accessing configurations such as database connections and API client initialization.

---

## 8. LLM Providers

NeMo supports 2 types of custom LLM providers:

| **Type** | **Base Class** | **Input** | **Output** |
| --- | --- | --- | --- |
| Text Completion | BaseLLM | String prompt | String response |
| Chat Model | BaseChatModel | List of messages | Message response |

### BaseLLM

```python
from typing import Any, List, Optional
from langchain_core.callbacks.manager import CallbackManagerForLLMRun
from langchain_core.language_models import BaseLLM
from nemoguardrails.llm.providers import register_llm_provider

class MyCustomLLM(BaseLLM):
    """Custom text completion LLM."""

    @property
    def _llm_type(self) -> str:
        return "my_custom_llm"

    def _call(
        self,
        prompt: str,
        stop: Optional[List[str]] = None,
        run_manager: Optional[CallbackManagerForLLMRun] = None,
        **kwargs: Any,
    ) -> str:
        """Synchronous text completion."""
        # Your implementation here
        return "Generated text response"

    async def _acall(
        self,
        prompt: str,
        stop: Optional[List[str]] = None,
        run_manager: Optional[CallbackManagerForLLMRun] = None,
        **kwargs: Any,
    ) -> str:
        """Asynchronous text completion (recommended)."""
        # Your async implementation here
        return "Generated text response"

# Register the provider
register_llm_provider("my_custom_llm", MyCustomLLM)
```

### BaseChatModel

```python
from typing import Any, List, Optional
from langchain_core.callbacks.manager import CallbackManagerForLLMRun
from langchain_core.language_models import BaseChatModel
from langchain_core.messages import AIMessage, BaseMessage
from langchain_core.outputs import ChatGeneration, ChatResult
from nemoguardrails.llm.providers import register_chat_provider

class MyCustomChatModel(BaseChatModel):
    """Custom chat model."""

    @property
    def _llm_type(self) -> str:
        return "my_custom_chat"

    def _generate(
        self,
        messages: List[BaseMessage],
        stop: Optional[List[str]] = None,
        run_manager: Optional[CallbackManagerForLLMRun] = None,
        **kwargs: Any,
    ) -> ChatResult:
        """Synchronous chat completion."""
        # Convert messages to your model's format
        response_text = "Generated chat response"
        message = AIMessage(content=response_text)
        generation = ChatGeneration(message=message)
        return ChatResult(generations=[generation])

    async def _agenerate(
        self,
        messages: List[BaseMessage],
        stop: Optional[List[str]] = None,
        run_manager: Optional[CallbackManagerForLLMRun] = None,
        **kwargs: Any,
    ) -> ChatResult:
        """Asynchronous chat completion (recommended)."""
        response_text = "Generated chat response"
        message = AIMessage(content=response_text)
        generation = ChatGeneration(message=message)
        return ChatResult(generations=[generation])

# Register the provider
register_chat_provider("my_custom_chat", MyCustomChatModel)
```

After registering your custom provider in `config.py`, use it in `config.yml`:

```yaml
models:
  - type: main
    engine: my_custom_llm  # or my_custom_chat
    model: optional-model-name
```

### BaseLLM Methods

| **Method** | **Required** | **Description** |
| --- | --- | --- |
| _call | Yes | Synchronous text completion |
| _llm_type | Yes | Returns the LLM type identifier |
| _acall | Yes | Asynchronous text completion |
| _stream | Optional | Streaming text completion |
| _astream | Optional | Async streaming text completion |

### BaseChatModel Methods

| **Method** | **Required** | **Description** |
| --- | --- | --- |
| _generate | Yes | Synchronous chat completion |
| _llm_type | Yes | Returns the LLM type identifier |
| _agenerate | Recommended | Asynchronous chat completion |
| _stream | Optional | Streaming chat completion |
| _astream | Optional | Async streaming chat completion |

> **Note:** Use the correct registration function: `register_llm_provider()` for BaseLLM subclasses; `register_chat_provider()` for BaseChatModel subclasses.

---

## 9. Custom Embedding Providers

### Create a custom embedding provider

Create a class that inherits from `EmbeddingModel`:

```python
from typing import List
from nemoguardrails.embeddings.providers.base import EmbeddingModel

class CustomEmbedding(EmbeddingModel):
    """Custom embedding provider."""

    engine_name = "custom_embedding"

    def __init__(self, embedding_model: str, **kwargs):
        """Initialize the embedding model.

        Args:
            embedding_model: The model name from config.yml
            **kwargs: Additional parameters from config.yml
        """
        self.model_name = embedding_model
        # Initialize your model here
        self.model = load_model(embedding_model)

    def encode(self, documents: List[str]) -> List[List[float]]:
        """Encode documents into embeddings (synchronous).

        Args:
            documents: List of text documents to encode

        Returns:
            List of embedding vectors
        """
        return [self.model.encode(doc) for doc in documents]

    async def encode_async(self, documents: List[str]) -> List[List[float]]:
        """Encode documents into embeddings (asynchronous).

        Args:
            documents: List of text documents to encode

        Returns:
            List of embedding vectors
        """
        # For simple models, can just call sync version
        return self.encode(documents)
```

### Register the provider in config.py

```python
from nemoguardrails import LLMRails

def init(app: LLMRails):
    from .embeddings import CustomEmbedding
    app.register_embedding_provider(CustomEmbedding, "custom_embedding")
```

### Configure in config.yml

```yaml
models:
  - type: embeddings
    engine: custom_embedding
    model: my-model-name
```

### Required Methods

| **Method** | **Description** |
| --- | --- |
| __init__(embedding_model: str, **kwargs) | Initialize with model name and additional parameters from config |
| encode(documents: List[str]) | Synchronous encoding |
| encode_async(documents: List[str]) | Asynchronous encoding |

---

## 10. Custom Configuration Data

Passing additional configuration to your initialization code and actions.

### Adding `custom_data` section to config.yml

```yaml
models:
  - type: main
    engine: openai
    model: gpt-4

custom_data:
  api_endpoint: "https://api.example.com"
  max_retries: 3
  timeout_seconds: 30
  feature_flags:
    enable_caching: true
    debug_mode: false
```

### Access custom data in `init` function

```python
from nemoguardrails import LLMRails

def init(app: LLMRails):
    # Access custom_data from the configuration
    custom_data = app.config.custom_data

    # Get individual values
    api_endpoint = custom_data.get("api_endpoint")
    max_retries = custom_data.get("max_retries", 3)  # with default

    # Access nested values
    feature_flags = custom_data.get("feature_flags", {})
    enable_caching = feature_flags.get("enable_caching", False)

    # Load sensitive values from environment variables
    import os
    api_key = os.environ.get("API_KEY")

    # Use to configure your providers
    client = APIClient(
        endpoint=api_endpoint,
        api_key=api_key,
        max_retries=max_retries
    )

    app.register_action_param("api_client", client)
```
