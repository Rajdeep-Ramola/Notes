# 🚫 Restrict Topics

> Use NeMo Guardrails topic control to ensure conversations stay within predefined subject boundaries.

---

## 📑 Table of Contents

1. [Config.yml](#1-configyml)
2. [Prompts.yml](#2-promptsyml)

---

## 1. Config.yml

```yaml
models:
  - type: main
    engine: nim
    model: meta/llama-3.3-70b-instruct

  - type: topic_control
    engine: nim
    model: nvidia/llama-3.1-nemoguard-8b-topic-control

rails:
  input:
    flows:
      - topic safety check input $model=topic_control
```

---

## 2. Prompts.yml

```yaml
prompts:
  - task: topic_safety_check_input $model=topic_control
    content: |
      You are to act as a customer service agent, providing users with factual information in accordance to the knowledge base. Your role is to ensure that you respond only to relevant queries and adhere to the following guidelines

      Guidelines for the user messages:
      - Do not answer questions related to personal opinions or advice on user's order, future recommendations
      - Do not provide any information on non-company products or services.
      - allow user comments that are related to small talk and chit-chat.
```
