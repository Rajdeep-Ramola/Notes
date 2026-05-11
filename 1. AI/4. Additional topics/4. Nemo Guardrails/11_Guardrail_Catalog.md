# 📚 Guardrail Catalog

> Ready-to-use guardrail implementations for content safety, jailbreak protection, topic control, PII detection, agentic security, and more.

---

## 📑 Table of Contents

1. [Content Safety](#1-content-safety)
2. [Jailbreak Protection](#2-jailbreak-protection)
3. [Topic Control](#3-topic-control)
4. [PII Detection](#4-pii-detection)
5. [Agentic Security](#5-agentic-security)
6. [YAML Schema Reference — Rails Config Section](#6-yaml-schema-reference--rails-config-section)

---

## 1. Content Safety

To use the content safety check:

### Include desired content safety models in the models section of config.yml

```yaml
models:
  - type: main
    engine: openai
    model: gpt-3.5-turbo-instruct

  - type: "content_safety"
    engine: nim
    parameters:
      base_url: "http://localhost:8123/v1"
      model_name: "llama-3.1-nemoguard-8b-content-safety"

  - type: llama_guard_2
    engine: vllm_openai
    parameters:
      openai_api_base: "http://localhost:5005/v1"
      model_name: "meta-llama/Meta-Llama-Guard-2-8B"
```

### Include content safety check in input and output rails section of config.yml

```yaml
rails:
  input:
    flows:
      - content safety check input $model=content_safety

  output:
    flows:
      - content safety check output $model=content_safety
```

`content safety check output` and `content safety check input` return a Boolean value indicating whether the input or output text is safe.

### Specify the prompts for each content safety check flow in prompts.yml

```yaml
prompts:
  - task: content_safety_check_input $model=shieldgemma
    content: |
      You are a policy expert trying to help determine whether a user prompt is in
      violation of the defined safety policies.

      <start_of_turn>
      Human Question: {{ user_input }}
      <end_of_turn>

      Our safety principle is defined in the below:

      * "No Hate Speech": The prompt shall not contain or seek generation of content

      Does the human prompt/question violate the above principle? Your answer must start with
      'Yes' or 'No'. And then walk through step by step to be sure we answer
      correctly.

    output_parser: is_content_safe
```

You must specify the output parser. You can use your own parser and register it, or use the off-the-shelf `is_content_safe` output parser as shown above.

### Custom Refusal Messages

```yaml
rails:
  config:
    content_safety:
      multilingual:
        enabled: true
        refusal_messages:
          en: "Sorry, I cannot help with that request."
          es: "Lo siento, no puedo ayudar con esa solicitud."
          zh: "抱歉，我无法处理该请求。"
          # Add other languages as needed
```

---

## 2. Jailbreak Protection

NeMo Guardrails library supports jailbreak detection using a set of heuristics — length per perplexity and prefix and suffix perplexity.

Perplexity is a metric that measures how well a language model predicts text. Lower is better, meaning less randomness or surprise. Typically jailbreak attempts result in higher perplexity.

### Activate the jailbreak detection heuristics flow as an input rail

```yaml
rails:
  input:
    flows:
      - jailbreak detection heuristics
```

### Set the desired threshold in config.yml

```yaml
rails:
  config:
    jailbreak_detection:
      server_endpoint: "http://0.0.0.0:1337/heuristics"
      length_per_perplexity_threshold: 89.79
      prefix_suffix_perplexity_threshold: 1845.65
```

### Heuristics

**Length per perplexity** — Computes the length of the input divided by the perplexity of the input. If the value is above the specified threshold (default 89.79), then the input is considered a jailbreak attempt.

**Prefix and suffix perplexity** — Takes the input and computes the perplexity for the prefix and suffix. This heuristic examines strings of more than 20 "words" (strings separated by whitespace) to detect potential prefix/suffix attacks.

> **Note:** To compute the perplexity of a string, the current implementation uses the `gpt2-large` model.

---

## 3. Topic Control

The topic safety feature allows you to define and enforce specific conversation rules and boundaries.

### Include the topic control model in the models section of config.yml

```yaml
models:
  - type: "topic_control"
    engine: nim
    parameters:
      base_url: "http://localhost:8123/v1"
      model_name: "llama-3.1-nemoguard-8b-topic-control"
```

### Include the topic safety check in your rails configuration

```yaml
rails:
  input:
    flows:
      - topic safety check input $model=topic_control
```

### Define your topic rules in the system prompt

```yaml
prompts:
  - task: topic_safety_check_input $model=topic_control
    content: |
      You are to act as a customer service agent, providing users with factual information in accordance to the knowledge base. Your role is to ensure that you respond only to relevant queries and adhere to the following guidelines

      Guidelines for the user messages:
      - Do not answer questions related to personal opinions or advice on user's order, future recommendations
```

> **Note:** The system prompt must end with the topic safety output restriction — "If any of the above conditions are violated, please respond with 'off-topic'. Otherwise, respond with 'on-topic'. You must respond with 'on-topic' or 'off-topic'."

---

## 4. PII Detection

To activate the PII detection, you need to set up the server endpoint of the PII detection model in your `config.yml` and specify the entities that you want to detect and mask.

### GLiNER-based PII Detection

```yaml
rails:
  config:
    gliner: # model
      server_endpoint: http://localhost:1235/v1/extract
      threshold: 0.5  # Confidence threshold (0.0 to 1.0)
      input:
        entities:  # If no entity is specified, all default PII categories are detected
          - email
          - phone_number
          - ssn
          - first_name
          - last_name
      output:
        entities:
          - email
          - phone_number
          - credit_debit_card

  input:
    flows:
      - gliner detect pii on input

  output:
    flows:
      - gliner detect pii on output
```

### Presidio-based Sensitive Data Detection

```yaml
rails:
  config:
    sensitive_data_detection:
      input:
        entities:
          - PERSON
          - EMAIL_ADDRESS
          - ...
```

---

## 5. Agentic Security

Specialized guardrails for LLM-based agents that use tools and interact with external systems. Detects potential exploitation attempts using injection such as code injection, cross-site scripting, SQL injection, and template injection.

### YARA Rules

The first part of injection detection uses [YARA rules](https://yara.readthedocs.io/en/stable/index.html). A YARA rule specifies a set of strings (text or binary patterns) to match and a Boolean expression that specifies the logic of the rule.

```
rule silent_banker : banker
{
    meta:
        description = "This is just an example"
        threat_level = 3
        in_the_wild = true

    strings:
        $a = {6A 40 68 00 30 00 00 6A 14 8D 91}
        $b = {8D 4D B0 2B C1 83 C0 27 99 6A 4E 59 F7 F9}
        $c = "UVODFRYSIHLNWPEJXQZAKCBGMT"

    condition:
        $a or $b or $c
}
```

### Configuring Injection Detection

```yaml
rails:
  config:
    injection_detection:
      injections:
        - code
        - sqli
        - template
        - xss
      action:
        reject

  output:
    flows:
      - injection detection
```

### `rails.config.injection_detection` Field Syntax Reference

| **Field** | **Description** | **Default Value** |
| --- | --- | --- |
| injections | Specifies the injection detection rules to use. Available types: `code` for Python code injection, `sqli` for SQL injection, `template` for Jinja template injection, `xss` for cross-site scripting. | None (required) |
| action | Specifies the action to take when injection is detected. `reject` returns a message to the user indicating that the query could not be handled. `omit` returns the model response, removing the offending detected content. | None (required) |
| yara_path | Specifies the path to a directory that contains custom YARA rules. | library/injection_detection/yara_rules in the NeMo Guardrails package. |
| yara_rules | Specifies inline YARA rules as a dictionary mapping rule names to rule content. If specified, these inline rules override the rules found in the yara_path field. | None |

> **Note:** Install `yara-python` and `nemoguardrails[jailbreak]` before using injection detection.

---

## 6. YAML Schema Reference — Rails Config Section

The `rails.config` section contains configuration for specific built-in rails.

### Jailbreak Detection

| **Attribute** | **Type** | **Default** | **Description** |
| --- | --- | --- | --- |
| rails.config.jailbreak_detection.server_endpoint | string | null | Heuristics model endpoint |
| jailbreak_detection.length_per_perplexity_threshold | float | 89.79 | Length/perplexity threshold |
| jailbreak_detection.prefix_suffix_perplexity_threshold | float | 1845.65 | Prefix/suffix perplexity threshold |
| rails.config.jailbreak_detection.nim_base_url | string | null | NIM base URL |
| rails.config.jailbreak_detection.nim_server_endpoint | string | "classify" | NIM endpoint path |
| rails.config.jailbreak_detection.api_key_env_var | string | null | Environment variable for API key |
| rails.config.jailbreak_detection.api_key | string | null | API key (not recommended) |

### Sensitive Data Detection

| **Attribute** | **Type** | **Default** | **Description** |
| --- | --- | --- | --- |
| rails.config.sensitive_data_detection.recognizers | list | [] | Custom Presidio recognizers |
| sensitive_data_detection.input/output/retrieval.entities | list | [] | Entity types to detect |
| sensitive_data_detection.input/output/retrieval.mask_token | string | "*" | Token for masking |
| sensitive_data_detection.input/output/retrieval.score_threshold | float | 0.2 | Detection confidence threshold |

### Injection Detection

| **Attribute** | **Type** | **Default** | **Description** |
| --- | --- | --- | --- |
| rails.config.injection_detection.injections | list | [] | Injection types: sqli, template, code, xss |
| rails.config.injection_detection.action | string | "reject" | Action: reject or omit |
| rails.config.injection_detection.yara_path | string | "" | Custom YARA rules path |
| rails.config.injection_detection.yara_rules | object | {} | Inline YARA rules |

### Fact Checking

```yaml
rails:
  config:
    fact_checking:
      parameters:
        endpoint: "http://localhost:5000"
      fallback_to_self_check: false
```
