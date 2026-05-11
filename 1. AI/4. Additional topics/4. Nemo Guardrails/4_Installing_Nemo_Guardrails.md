# ⚙️ Installing NeMo Guardrails

> Install NeMo Guardrails with the required dependencies for your use case.

---

## 📑 Table of Contents

1. [Basic Installation](#1-basic-installation)
2. [Extra Dependencies](#2-extra-dependencies)

---

## 1. Basic Installation

```bash
pip install "nemoguardrails[nvidia]"
```

---

## 2. Extra Dependencies

| **Extra** | **Description** |
| --- | --- |
| nvidia | NVIDIA-hosted model integration through [build.nvidia.com](https://build.nvidia.com/) |
| openai | OpenAI-hosted model integration |
| server | [Guardrails API server](https://docs.nvidia.com/nemo/guardrails/latest/run-rails/using-fastapi-server/overview.html) dependencies (aiofiles for async file handling, openai for API schemas). FastAPI is a core dependency. Required to run `nemoguardrails server`. |
| sdd | [Sensitive data detection](https://docs.nvidia.com/nemo/guardrails/latest/configure-rails/guardrail-catalog/pii-detection.html) using Presidio |
| eval | [Evaluation tools](https://docs.nvidia.com/nemo/guardrails/latest/evaluation/evaluate-guardrails.html) for testing guardrails |
| tracing | OpenTelemetry tracing support |
| gcp | Google Cloud Platform language services |
| jailbreak | YARA-based jailbreak detection heuristics |
| multilingual | Language detection for multilingual content |
| all | All optional packages |
