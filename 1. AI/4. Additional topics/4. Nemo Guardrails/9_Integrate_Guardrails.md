# 🔗 Integrate Guardrails in Our Application

> Full integration guide using both the Python SDK and the NeMo Guardrails server.

---

## 📑 Table of Contents

1. [Configuring Guardrails - Library Configuration Overview](#1-configuring-guardrails---library-configuration-overview)
2. [Configuring the YAML File](#2-configuring-the-yaml-file)
3. [Model Configuration](#3-model-configuration)
4. [Guardrails Configuration](#4-guardrails-configuration)
5. [Parallel Execution of Input and Output Rails](#5-parallel-execution-of-input-and-output-rails)
6. [YAML Schema Reference](#6-yaml-schema-reference)

---

## 1. Configuring Guardrails - Library Configuration Overview

A guardrails configuration includes the following components:

| **Component** | **Required/Optional** | **Description** | **Location** |
| --- | --- | --- | --- |
| **Core Configuration** | Required | A `config.yml` file that contains the core configuration options such as which LLM(s) to use, general instructions (similar to system prompts), sample conversation, which rails are active, and specific rails configuration options. | config.yml |
| **Colang Flows** | Optional | A collection of Colang files (.co files) implementing the rails. Files are loaded recursively from anywhere in the config directory. | Config root, rails/ folder, or any subfolder |
| **Custom Prompts** | Optional | YAML file with custom prompts for guardrails tasks. Prompts can also be defined directly in config.yml. | prompts.yml |
| **Custom Actions** | Optional | Python functions decorated with `@action()` that can be called from Colang flows during request processing (for example, external API calls, validation logic). | actions.py or actions/ folder |
| **Custom Initialization** | Optional | Python code that runs once at startup to register custom LLM providers, embedding providers, or shared resources (for example, database connections). | config.py |
| **Knowledge Base Documents** | Optional | Documents (.md files) that can be used in a RAG (Retrieval-Augmented Generation) scenario (i.e. Retrieval rail) using the built-in Knowledge Base support. | kb folder |

---

## 2. Configuring the YAML File

The `config.yml` file is the primary configuration file for defining LLM models, guardrails behavior, prompts, knowledge base settings, and tracing options.

### Schema for config.yml

```yaml
# LLM model configuration
models:
  - type: main
    engine: openai
    model: gpt-4o

# Instructions for the LLM (similar to system prompts)
instructions:
  - type: general
    content: |
      You are a helpful AI assistant.

# Guardrails configuration
rails:
  input:
    flows:
      - self check input

  output:
    flows:
      - self check output

  ... # Other rail configurations

# Prompt customization
prompts:
  - task: self_check_input
    content: |
      Your task is to check if the user message complies with policy.

# Knowledge base settings
knowledge_base:
  embedding_search_provider:
    name: default

# Tracing and monitoring
tracing:
  enabled: true
  adapters:
    - name: FileSystem
      filepath: "./logs/traces.jsonl"
```

---

## 3. Model Configuration

### NIM Configuration

```yaml
models:
  - type: main
    engine: nim # nvidia's cloud infrastructure on which model is hosted
    model: meta/llama-3.1-8b-instruct
```

### Task-Specific Models

```yaml
models:
  - type: main
    engine: nim
    model: meta/llama-3.1-8b-instruct

  - type: self_check_input
    engine: nim
    model: meta/llama3-8b-instruct
```

### Model Parameters

```yaml
models:
  - type: main
    engine: openai
    model: gpt-4
    parameters:
      temperature: 0.7
      max_tokens: 1000
      top_p: 0.9
```

### Complete Example

```yaml
models:
  # Main application LLM
  - type: main
    engine: nim
    model: meta/llama-3.1-70b-instruct
    parameters:
      temperature: 0.7
      max_tokens: 2000

  # Embeddings for knowledge base
  - type: embeddings
    engine: FastEmbed
    model: all-MiniLM-L6-v2

  # Dedicated model for input checking
  - type: self_check_input
    engine: nim
    model: nvidia/llama-3.1-nemoguard-8b-content-safety

  # Dedicated model for output checking
  - type: self_check_output
    engine: nim
    model: nvidia/llama-3.1-nemoguard-8b-content-safety
```

---

## 4. Guardrails Configuration

The `rails` key defines which guardrails are active and their configuration options. Rails are organized into five categories based on when they trigger during the guardrails process.

### Basic Configuration

```yaml
rails:
  input:
    flows:
      - self check input
      - jailbreak detection heuristics
      - mask sensitive data on input

  output:
    flows:
      - self check output
      - self check facts
      - check output sensitive data

  retrieval:
    flows:
      - check retrieval sensitive data
```

### Input Rails — Process user messages before they reach the LLM

```yaml
rails:
  input:
    flows:
      - self check input                # LLM-based input validation
      - jailbreak detection heuristics  # Jailbreak detection
      - mask sensitive data on input    # PII masking
```

### Output Rails — Process LLM responses before returning to users

```yaml
rails:
  output:
    flows:
      - self check output          # LLM-based output validation
      - self check facts           # Fact verification
      - self check hallucination   # Hallucination detection
      - mask sensitive data on output  # PII masking
```

### Retrieval Rails — Process chunks retrieved from the knowledge base

```yaml
rails:
  retrieval:
    flows:
      - check retrieval sensitive data
```

### Dialog Rails — Control conversation flow after user intent is determined

```yaml
rails:
  dialog:
    single_call:
      enabled: false
      fallback_to_multiple_calls: true

    user_messages:
      embeddings_only: false
```

### Execution Rails — Control custom action and tool invocations

```yaml
rails:
  execution:
    flows:
      - check tool input
      - check tool output
```

### Rail-Specific Configuration

Configure options for specific rails using the `config` key:

```yaml
rails:
  config:
    # Sensitive data detection settings
    sensitive_data_detection:
      input:
        entities:
          - PERSON
          - EMAIL_ADDRESS
          - PHONE_NUMBER
      output:
        entities:
          - PERSON
          - EMAIL_ADDRESS

    # Jailbreak detection settings
    jailbreak_detection:
      length_per_perplexity_threshold: 89.79
      prefix_suffix_perplexity_threshold: 1845.65

    # Fact-checking settings
    fact_checking:
      parameters:
        endpoint: http://localhost:5000
```

---

## 5. Parallel Execution of Input and Output Rails

`LLMRails` flow can run in parallel using the `parallel: True` option in `config.yml`.

> **Note:** Avoid parallel execution for CPU-bound rails.

```yaml
models:
  - type: main
    engine: nim
    model: meta/llama-3.1-70b-instruct

  - type: content_safety
    engine: nim
    model: nvidia/llama-3.1-nemoguard-8b-content-safety

  - type: topic_control
    engine: nim
    model: nvidia/llama-3.1-nemoguard-8b-topic-control

rails:
  input:
    parallel: True
    flows:
      - content safety check input $model=content_safety
      - topic safety check input $model=topic_control

  output:
    parallel: True
    flows:
      - content safety check output $model=content_safety
      - self check output
```

---

## 6. YAML Schema Reference

### Model Configurations

| **Attribute** | **Type** | **Required** | **Description** |
| --- | --- | --- | --- |
| models.type | string | ✓ | Model identifier |
| models.engine | string | ✓ | LLM provider |
| models.model | string | ✓ | Model name (can also be in parameters.model_name) |
| models.mode | string |  | Completion mode: chat or text (default: chat) |
| models.api_key_env_var | string |  | Environment variable containing API key |
| models.parameters | object |  | Provider-specific parameters passed to LangChain |
| models.cache | object |  | Cache configuration for this model |

### Engines

| **Engine** | **Description** |
| --- | --- |
| openai | OpenAI models |
| nim | NVIDIA NIM microservices |
| nvidia_ai_endpoints | Alias for nim |
| azure | Azure OpenAI models |
| anthropic | Anthropic Claude models |
| cohere | Cohere models |
| vertexai | Google Vertex AI |

### Embedding Engines

| **Engine** | **Description** |
| --- | --- |
| FastEmbed | FastEmbed (default) |
| openai | OpenAI embeddings |
| nim | NVIDIA NIM embeddings |

### Model Cache Configuration

```yaml
models:
  - type: content_safety
    engine: nim
    model: nvidia/llama-3.1-nemotron-safety-guard-8b-v3
    cache:
      enabled: true
      maxsize: 50000
      stats:
        enabled: false
        log_interval: null
```

### Built-in Input Flows

| **Flow** | **Description** |
| --- | --- |
| self check input | LLM-based policy compliance check |
| jailbreak detection heuristics | Jailbreak detection heuristics |
| jailbreak detection model | NIM-based jailbreak detection |
| mask sensitive data on input | Mask PII in user input |
| detect sensitive data on input | Detect and block PII |
| llama guard check input | LlamaGuard content moderation |
| content safety check input | NVIDIA content safety model |
| topic safety check input | Topic control model |

### Built-in Output Flows

| **Flow** | **Description** |
| --- | --- |
| self check output | LLM-based policy compliance check |
| self check facts | Fact verification |
| self check hallucination | Hallucination detection |
| mask sensitive data on output | Mask PII in output |
| llama guard check output | LlamaGuard content moderation |
| content safety check output | NVIDIA content safety model |
| injection detection | Injection detection (SQL, XSS, code, template) |
