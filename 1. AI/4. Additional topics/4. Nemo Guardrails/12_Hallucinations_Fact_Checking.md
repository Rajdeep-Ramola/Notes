# 🔍 Hallucinations and Fact Checking

> Protect against false or unsupported claims in LLM responses using NeMo Guardrails' built-in fact-checking and hallucination detection rails.

---

## 📑 Table of Contents

1. [Self Check Fact Checking](#1-self-check-fact-checking)
2. [Hallucination Detection](#2-hallucination-detection)
3. [Other Implementations](#3-other-implementations)

---

## 1. Self Check Fact Checking

Ensures that the answer to a RAG query is grounded in the provided evidence extracted from the knowledge base.

### Include `self check facts` flow in the output rails section

```yaml
rails:
  output:
    flows:
      - self check facts
```

### Define `self_check_facts` prompt in prompts.yml

```yaml
prompts:
  - task: self_check_facts
    content: |-
      You are given a task to identify if the hypothesis is grounded and entailed to the evidence.
      You will only use the contents of the evidence and not rely on external knowledge.
      Answer with yes/no. "evidence": {{ evidence }} "hypothesis": {{ response }} "entails":
```

The `self_check_facts` prompt has two input variables: `{{ evidence }}`, which includes the relevant chunks, and `{{ response }}`, which includes the bot response that should be fact-checked.

---

## 2. Hallucination Detection

Protects against false claims in the generated bot message.

### Include `self check hallucination` flow in the output rails section of config.yml

```yaml
rails:
  output:
    flows:
      - self check hallucination
```

### Define `self_check_hallucination` prompt

```yaml
prompts:
  - task: self_check_hallucination
    content: |-
      You are given a task to identify if the hypothesis is in agreement with the context below.
      You will only use the contents of the context and not rely on external knowledge.
      Answer with yes/no. "context": {{ paragraph }} "hypothesis": {{ statement }} "agreement":
```

The `self_check_hallucination` prompt has two input variables: `{{ paragraph }}`, which represents alternative generations for the same user query, and `{{ statement }}`, which represents the current bot response. The completion must be "yes" if the statement is not a hallucination.

### Two Modes of Hallucination Detection

**Blocking** — Block the message if a hallucination is detected. Set the `$check_hallucination` context variable to `True`:

```colang
define flow  # colang file
  user ask about people
  $check_hallucination = True
  bot respond about people
```

**Warning** — Warn the user if the response is prone to hallucinations. Set the `$hallucination_warning` context variable to `True`.

---

## 3. Other Implementations

### AlignScore-based Fact Checking

```yaml
rails:
  config:
    fact_checking:
      parameters:
        # Point to a running instance of the AlignScore server
        endpoint: "http://localhost:5000/alignscore_large"

  output:
    flows:
      - alignscore check facts
```

### Patronus Lynx-based RAG Hallucination Detection

```yaml
rails:
  output:
    flows:
      - patronus lynx check output hallucination
```
