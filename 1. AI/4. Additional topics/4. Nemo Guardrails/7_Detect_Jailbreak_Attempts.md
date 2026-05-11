# 🔐 Detect Jailbreak Attempts

> Configure NeMo Guardrails to detect and block adversarial jailbreak attempts targeting your LLM.

---

## 📑 Table of Contents

1. [Config.yml](#1-configyml)

---

## 1. Config.yml

```yaml
models:
  - type: main
    engine: nim
    model: meta/llama-3.3-70b-instruct

rails:
  input:
    flows:
      - jailbreak detection model

  config:
    jailbreak_detection:
      nim_base_url: "https://ai.api.nvidia.com"
      nim_server_endpoint: "/v1/security/nvidia/nemoguard-jailbreak-detect"
      api_key_env_var: NVIDIA_API_KEY
```

> **Note:** The NemoGuard Jailbreak Detect model does not use any prompts.
