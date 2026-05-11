# 🛡️ Use Cases

> NeMo Guardrails supports a wide range of use cases for making LLM applications safer and more reliable.

---

## 📑 Table of Contents

1. [Add Content Safety](#1-add-content-safety)
2. [Add Jailbreak Protection](#2-add-jailbreak-protection)
3. [Control Topic Conversation](#3-control-topic-conversation)
4. [Detect and Mask PII](#4-detect-and-mask-pii)
5. [Add Agentic Security](#5-add-agentic-security)
6. [Build Your Own Guardrails Solutions](#6-build-your-own-guardrails-solutions)

---

## 1. Add Content Safety

Both user inputs and LLM outputs are safe.

- **LLM self checking** — Use the LLM itself to check inputs and outputs for harmful content.
- **NVIDIA safety models** — Integrate with [Llama 3.1 NemoGuard 8B Content Safety](https://build.nvidia.com/nvidia/llama-3_1-nemotron-safety-guard-8b-v3) for robust content moderation.
- **Community models** — Use LlamaGuard, [Fiddler Guardrails](https://docs.nvidia.com/nemo/guardrails/latest/configure-rails/guardrail-catalog/community/fiddler.html), and other community content safety solutions.

---

## 2. Add Jailbreak Protection

Prevents bypassing safety measures and manipulating the LLM into generating harmful or unwanted content.

- **Self-check jailbreak detection** — Use the LLM to identify jailbreak attempts.
- **Heuristic detection** — Use pattern-based detection for common jailbreak techniques.
- **NVIDIA NemoGuard** — Integrate with NemoGuard Jailbreak Detection NIM for advanced threat detection.
- **Third-party integrations** — Use [Prompt Security](https://docs.nvidia.com/nemo/guardrails/latest/configure-rails/guardrail-catalog/community/prompt-security.html), [Pangea AI Guard](https://docs.nvidia.com/nemo/guardrails/latest/configure-rails/guardrail-catalog/community/pangea.html), and other services.

---

## 3. Control Topic Conversation

Ensures conversations stay within predefined subject boundaries.

- **Dialog rails** — Pre-define conversational flows using the Colang language.
- **Topical rails** — Control what topics the bot can and cannot discuss.
- **NVIDIA NemoGuard** — Integrate with NemoGuard Topic Control NIM for semantic topic detection.

---

## 4. Detect and Mask PII

Protect user privacy by detecting and masking sensitive data in user inputs, LLM outputs, and retrieved content.

- **Gliner** — Use [NVIDIA GLiNER-PII](https://docs.nvidia.com/nemo/guardrails/latest/configure-rails/guardrail-catalog/community/gliner.html) for detecting entities such as names, email addresses, phone numbers, and social security numbers.
- **Presidio-based detection** — Use [Microsoft Presidio](https://docs.nvidia.com/nemo/guardrails/latest/configure-rails/guardrail-catalog/community/presidio.html) for detecting entities such as names and email addresses.
- **Private AI** — Integrate with [Private AI](https://docs.nvidia.com/nemo/guardrails/latest/configure-rails/guardrail-catalog/community/privateai.html) for advanced PII detection and masking.
- **AutoAlign** — Use AutoAlign PII detection with customizable entity types.
- **GuardrailsAI** — Access GuardrailsAI PII validators from the Guardrails Hub.

---

## 5. Add Agentic Security

Guardrails for agents that interact with external systems through tools.

- **Tool call validation** — Execute rails that validate tool inputs and outputs before and after invocation.
- **Secure tool integration** — Review guidelines for safely connecting LLMs to external resources.

---

## 6. Build Your Own Guardrails Solutions

- **Python actions** — Create custom actions in Python for complex logic and external integrations.
