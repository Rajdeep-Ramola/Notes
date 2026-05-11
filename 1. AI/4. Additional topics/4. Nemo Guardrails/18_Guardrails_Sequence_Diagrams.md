# 📊 Guardrails Sequence Diagrams and Use Case Diagrams

> Visual reference for understanding NeMo Guardrails architecture, master flow, and individual use case summaries.

---

## 📑 Table of Contents

1. [Master Rails Flow Diagram](#1-master-rails-flow-diagram)
2. [Use Case Diagrams](#2-use-case-diagrams)

---

## 1. Master Rails Flow Diagram

![Master Rails Flow](https://docs.nvidia.com/nemo/guardrails/latest/_images/master_rails_flow.png)

---

## 2. Use Case Diagrams

- **Content safety** — Content safety guardrails check both user inputs and LLM outputs for harmful content.
![Content safety flow Diagram](assets/image1.png)

- **Jailbreak protection** — Jailbreak protection prevents adversarial attempts from bypassing safety measures.
![Jailbreak protection flow Diagram](assets/image2.png)

- **Topic control** — Topic control ensures conversations stay within predefined subject boundaries.
![Topic control flow Diagram](assets/image3.png)

- **PII detection and masking** — PII detection protects user privacy by detecting and masking sensitive data.
![PII detection flow Diagram](assets/image4.png)

- **Agentic security** — Agentic security provides guardrails for LLM agents using tools and external systems.
![Agentic security flow Diagram](assets/image5.png)

- **Custom and third-party guardrails** — Build custom guardrails using Python actions, LangChain tools, or third-party APIs.
![Custom and third-party guardrails flow Diagram](assets/image6.png)

- **Integration options** — Multiple integration options: Python SDK, HTTP Server API, or OpenAI-compatible endpoint.
![Integration options flow Diagram](assets/image7.png)

- **Combined architecture overview** — A unified view of how all guardrail components work together in a production system.
![Combined architecture overview flow Diagram](assets/image8.png)
