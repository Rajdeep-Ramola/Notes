# 🖼️ Add Multimodal Content Safety

> Use a vision model to apply content safety checks on both text and image inputs.

---

## 📑 Table of Contents

1. [Config.yml](#1-configyml)
2. [Prompts.yml](#2-promptsyml)
3. [Send a Request](#3-send-a-request)

---

## 1. Config.yml

```yaml
models:
  - type: main
    engine: openai
    model: gpt-4o-mini

  - type: vision_rails
    engine: openai
    model: gpt-4o-mini

rails:
  input:
    flows:
      - content safety check input $model=vision_rails

  dialog:
    single_call:
      enabled: False
```

---

## 2. Prompts.yml

```yaml
prompts:
  - task: content_safety_check_input $model=vision_rails
    messages:
      - type: system
        content: |
          You are an AI assistant that evaluates if a user's message contains unsafe content.
          Carefully examine any text and images in the user's message.
          {%- if categories is not defined -%}
              {%- set categories = {
                  "S1": "Violent Crimes.",
                  "S2": "Non-Violent Crimes.",
              } -%}
          {%- endif -%}
          Task: Check if there is unsafe content in "User" messages in conversations according to our safety policy with the below categories.

          <BEGIN UNSAFE CONTENT CATEGORIES>
          {%- for key, desc in categories.items() -%}
              {%- if key not in excluded_category_keys -%}
          {{ key }}: {{ desc }}
              {%- endif -%}
          {%- endfor -%}
          <END UNSAFE CONTENT CATEGORIES>

          IMPORTANT:
          Provide your safety assessment for the user message:
          - First line must read 'safe' or 'unsafe' and nothing more
          - If unsafe, a second line must include a comma-separated list of violated categories.

      - type: user
        content: "{{ user_input  }}"

    stop: ["<|eot_id|>", "<|eom_id|>"]
    output_parser: is_content_safe
    max_tokens: 200
```

---

## 3. Send a Request

```python
potentially_unsafe_message = [{
  "role": "user",
  "content": [{
    "type": "text",
    "text": "How can I use the item in the photo to get a higher salary without working hard?",
  },
  {
    "type": "image_url",
    "image_url": {
      "url": "https://upload.wikimedia.org/wikipedia/commons/thumb/4/4f/SIG_Pro_by_Augustas_Didzgalvis.jpg/330px-SIG_Pro_by_Augustas_Didzgalvis.jpg"
    },
  }],
}]

potentially_unsafe_response = rails.generate(messages=potentially_unsafe_message)

print(f"Potentially Unsafe Response: {potentially_unsafe_response}")

print(json.dumps(potentially_unsafe_response, indent=2))
```
