# 📘 Tutorials

> Step-by-step tutorials for configuring and using NeMo Guardrails for content safety.

---

## 📑 Table of Contents

1. [Check Harmful Content](#1-check-harmful-content)
2. [Content Safety Using Nemotron-Content-Safety-Reasoning-4B](#2-content-safety-using-nemotron-content-safety-reasoning-4b)
3. [Modify Prompts for a Specific Use Case](#3-modify-prompts-for-a-specific-use-case)

---

## 1. Check Harmful Content

### Make a config.yml in the config directory

The `config.yml` file contains the models used by Guardrails in the `models` section and rails controlling when to use these models. The `models` section configures the type and name of each model, along with the engine used to perform LLM inference. The model with type `main` is used to generate responses to user queries. The `rails` section configures input and output rails. The input rails operate on the user input only, while output rails operate on the combined user input and LLM response.

```yaml
models:
  - type: main
    engine: nim
    model: meta/llama-3.3-70b-instruct

  - type: content_safety
    engine: nim
    model: nvidia/llama-3.1-nemotron-safety-guard-8b-v3

rails:
  input:
    flows:
      - content safety check input $model=content_safety

  output:
    flows:
      - content safety check output $model=content_safety
```

### Make a prompts.yml in the config folder

The `prompts.yml` file contains prompt templates, the parser used to interpret a guardrail model response, and the maximum tokens to generate. The `content_safety_check_input` task prompt template replaces `{{ user_input }}` with the user input. The `content_safety_check_output` task prompt template replaces `{{ user_input }}` and `{{ bot_response }}` with the user input and LLM response respectively.

```yaml
prompts:
  - task: content_safety_check_input $model=content_safety
    content: |
      Task: Check if there is unsafe content in the user or agent messages in the below conversation in accordance with our safety policy with the below categories. Provide a 'safe' or 'unsafe' rating for both the user message and the agent response. If there is unsafe content, please also provide a list of violated categories according to our safety policy below.

      <BEGIN UNSAFE CONTENT CATEGORIES>
      S1: Violence.
      S2: Sexual.
      S3: Criminal Planning/Confessions.
      <END UNSAFE CONTENT CATEGORIES>

      <BEGIN CONVERSATION>
      user: {{ user_input }}
      <END CONVERSATION>

      Provide your safety assessment for the above conversation in the below output JSON format:
      {"User Safety": safe or unsafe rating for the user message, "Response Safety": safe or unsafe rating for the agent response. Omit if no agent response present. "Safety Categories": a comma-separated list of applicable safety categories from the provided taxonomy. Omit if all safe.}

      Do not include anything other than the output JSON in your response.

      Output JSON:

    output_parser: nemoguard_parse_prompt_safety
    max_tokens: 50

  - task: content_safety_check_output $model=content_safety
    content: |
      Task: Check if there is unsafe content in the user or agent messages in the below conversation in accordance with our safety policy with the below categories. Provide a 'safe' or 'unsafe' rating for both the user message and the agent response. If there is unsafe content, please also provide a list of violated categories according to our safety policy below.

      <BEGIN UNSAFE CONTENT CATEGORIES>
      S1: Violence.
      S2: Sexual.
      S3: Criminal Planning/Confessions.
      S4: Guns and Illegal Weapons.
      <END UNSAFE CONTENT CATEGORIES>

      <BEGIN CONVERSATION>
      user: {{ user_input }}
      response: agent: {{ bot_response }}
      <END CONVERSATION>

      Provide your safety assessment for the above conversation in the below output JSON format:
      {"User Safety": safe or unsafe rating for the user message, "Response Safety": safe or unsafe rating for the agent response. Omit if no agent response present. "Safety Categories": a comma-separated list of applicable safety categories from the provided taxonomy. Omit if all safe.}

      Do not include anything other than the output JSON in your response.

      Output JSON:

    output_parser: nemoguard_parse_response_safety
    max_tokens: 50
```

---

## 2. Content Safety Using Nemotron-Content-Safety-Reasoning-4B

### Config.yml

```yaml
models:
  # Configure your main LLM (OpenAI, NIM, vLLM, etc.)
  - type: main
    engine: openai
    model: gpt-4o-mini

  # Content Safety Model served via vLLM (OpenAI-compatible API)
  - type: content_safety_reasoning
    engine: openai
    model: nvidia/Nemotron-Content-Safety-Reasoning-4B
    parameters:
      openai_api_base: http://localhost:8001/v1
      temperature: 0.6
      top_p: 0.95

rails:
  config:
    content_safety:
      reasoning:
        # Set to true for reasoning mode (with <think> traces)
        # Set to false for low-latency mode
        enabled: false

  input:
    flows:
      - content safety check input $model=content_safety_reasoning

  output:
    flows:
      - content safety check output $model=content_safety_reasoning
```

### Prompts.yml

```yaml
prompts:
  - task: content_safety_check_input $model=content_safety_reasoning
    content: |
      You are a classifier for analyzing interactions between humans and LLMs. Given a request from a human user and a response from an LLM assistant, determine 1) whether the human user's request is harmful, 2) whether the AI assistant's response is a refusal, and 3) whether the AI assistant's response is harmful.

      Taxonomy: The following are the list of harmful categories that the prompt and response may fall into.
      S1: Violence.
      S2: Sexual.
      S3: Criminal Planning/Confessions.
      S4: Guns and Illegal Weapons.
      S5: Controlled/Regulated Substances.
      S6: Suicide and Self Harm.

      Human user:
      {{ user_input }}

      AI assistant:
      None

      Use the taxonomy to determine if the prompt and response fall into any of the harmful categories. Output your answer in the following format:

      "Prompt harm: <harmful/unharmful>
      Response harm: <harmful/unharmful>"

      Only output the answer in the format specified above after thinking about the request and
      response. If there is no response, output "Response harm: <harmful/unharmful>" as None.

      {% if reasoning_enabled %}/think{% else %}/no_think{% endif %}

    output_parser: nemotron_reasoning_parse_prompt_safety
    max_tokens: 400

  - task: content_safety_check_output $model=content_safety_reasoning
    content: |
      You are a classifier for analyzing interactions between humans and LLMs. Given a request from a human user and a response from an LLM assistant, determine 1) whether the human user's request is harmful, 2) whether the AI assistant's response is a refusal, and 3) whether the AI assistant's response is harmful.

      Taxonomy: The following are the list of harmful categories that the prompt and response may fall into.
      S1: Violence.
      S2: Sexual.
      S3: Criminal Planning/Confessions.
      S4: Guns and Illegal Weapons.
      S5: Controlled/Regulated Substances.
      S6: Suicide and Self Harm.

      Human user:
      {{ user_input }}

      AI assistant:
      {{ bot_response }}

      Use the taxonomy to determine if the prompt and response fall into any of the harmful categories. Output your answer in the following format:

      "Prompt harm: <harmful/unharmful>
      Response harm: <harmful/unharmful>"

      Only output the answer in the format specified above after thinking about the request and
      response. If there is no response, output "Response harm: <harmful/unharmful>" as None.

      {% if reasoning_enabled %}/think{% else %}/no_think{% endif %}

    output_parser: nemotron_reasoning_parse_response_safety
    max_tokens: 400
```

---

## 3. Modify Prompts for a Specific Use Case

```yaml
content: |
  You are a classifier for a customer service chatbot. Determine if the user's request
  is on-topic for our electronics store.

  Allowed topics:
  - Product inquiries (features, specifications, availability)

  Disallowed topics:
  - Competitor products or pricing

  Human user:
  {{ user_input }}

  Output format:
  "Prompt harm: <harmful/unharmful>"

  Use "harmful" for off-topic requests, "unharmful" for on-topic requests.

  {% if reasoning_enabled %}/think{% else %}/no_think{% endif %}
```
