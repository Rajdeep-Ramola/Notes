# 🔒 Guardrails Types

> NeMo Guardrails are organized into five types based on when they trigger during the guardrails process.

---

## 📑 Table of Contents

1. [Input Rails](#1-input-rails)
2. [Retrieval Rails](#2-retrieval-rails)
3. [Dialog Rails](#3-dialog-rails)
4. [Execution Rails](#4-execution-rails)
5. [Output Rails](#5-output-rails)
6. [Flow Diagram](#6-flow-diagram)

---

## 1. Input Rails

Apply guardrails before the LLM is called by validating and sanitizing user inputs.

---

## 2. Retrieval Rails

Filter and validate retrieved knowledge (documents and chunks) to ensure only trusted context is provided to the LLM.

---

## 3. Dialog Rails

Steer and constrain the multi-turn conversation, enforcing flow logic and policies across turns.

---

## 4. Execution Rails

Control and validate tool/function calls, their arguments, and results to safely interact with external systems.

---

## 5. Output Rails

Evaluate and post-process model responses, filtering, editing, or blocking unsafe or off-policy content before it reaches users.

---

## 6. Flow Diagram

![Programmable Guardrails Flow](https://docs.nvidia.com/nemo/guardrails/latest/_images/programmable_guardrails_flow.png)

> **Note:** NVIDIA NemoGuard collection of models includes [content-safety](https://build.nvidia.com/nvidia/llama-3_1-nemotron-safety-guard-8b-v3), [topic-control](https://build.nvidia.com/nvidia/llama-3_1-nemoguard-8b-topic-control), and [jailbreak-detect](https://build.nvidia.com/nvidia/nemoguard-jailbreak-detect) models. These models can be accessed on [build.nvidia.com](https://build.nvidia.com/) for rapid prototyping.
