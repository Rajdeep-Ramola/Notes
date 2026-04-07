# 📦 Ollama

> A reference guide covering Ollama — a tool that lets us run LLMs on our local system — including model management, tool calling, Modelfiles, REST API usage, orchestration frameworks, and Ollama Cloud.

---

## 📑 Table of Contents

1. [Challenges while Running Models on Our System](#1-challenges-while-running-models-on-our-system)
2. [Ollama](#2-ollama)
   - [2.1 Benefits of Ollama](#21-benefits-of-ollama)
   - [2.2 Using Ollama through Ollama Library](#22-using-ollama-through-ollama-library)
   - [2.3 Tool Calling in Ollama](#23-tool-calling-in-ollama)
   - [2.4 Modelfile](#24-modelfile)
   - [2.5 Generate using REST API](#25-generate-using-rest-api)
   - [2.6 Using Ollama in an Orchestration Framework](#26-using-ollama-in-an-orchestration-framework)
   - [2.7 Ollama Cloud](#27-ollama-cloud)

---

## 📖 Key Definitions

- **Proprietary models** — AI models that are owned and controlled by a company
- **Open source models** — models whose core details are openly shared with the public

---

## 1. Challenges while Running Models on Our System

- How are we going to store those models on our system
- Configuring RAM/VRAM to run these models
- Compatibility issues

> ✅ All these issues are solved through Ollama

**Ollama** — a tool that lets us run LLMs on our local system

---

## 2. Ollama

### 2.1 Benefits of Ollama

- **Privacy and Data control** — input data never leaves our infrastructure
- **Low latency and offline access** — no network round trip to a cloud server
- **Cost predictability and potential savings** — only requires hardware and electricity
- **Simple installation and setup**
- **Pre-built model library** — has a collection of ready to use models
- **Customization of model** — customize models for our own use case
- **No vendor lock-in and IP protection** — not bound to cloud vendor terms
- **Run and manage models with simple commands** — models can be easily managed using simple commands

```bash
ollama pull llama2    # download model
ollama rm llama2      # remove model
ollama list           # see installed models
```

- `/show` — gives all the commands related to that model
- `/set` — gives all the available commands that we can use to fine tune and customize our model
- `/set parameter` — gives all the available commands through which we can change different parameters of the model

---

### 2.2 Using Ollama through Ollama Library

**Generate function:**

```python
response = ollama.generate(parameters)
```

**Parameters:**

- `model` — model name
- `prompt` — text for model to generate a response from
- `suffix` — used for fill-in-the-middle models
- `images` — Base64-encoded images for models that support image input
- `format` — Structured output format for the model to generate a response from
- `system` — System prompt for the model to generate a response from
- `stream` — Boolean, whether to stream output or not
- `think` — Boolean, When true, returns separate thinking output in addition to content
- `raw` — returns raw response from the model without any prompt templating
- `keep_alive` — keeps the model loaded in memory after a request
- `options` — object
  - `options.seed` — used to repeat the same output for the same prompt
  - `options.temperature` — controls randomness in generation (higher = more random)
  - `options.top_k` — limits next token selection to top K likely tokens
  - `options.top_p` — choose from smallest set of tokens whose probability >= p
  - `options.min_p` — minimum probability threshold for token selection
  - `options.stop` — stop sequences that will halt generation

---

### 2.3 Tool Calling in Ollama

**Tool calling lifecycle:**

- **Tool create** — functions
- **Tool schema** — JSON object which contains tools, description of each and every tool and parameters of those tools
- **LLM call** — prompt + tool schema is passed to the LLM and LLM returns a JSON object which contains which function is needed and what should be the values of parameters of those functions
- **Run function** — that particular tool or function has to be run manually with parameters in the JSON object
- **Final output** — All the history from starting is passed to the LLM including the tool output and then the LLM generates the output

> 📝 **Note:** for calling tools we have to use `chat` method instead of `generate` method

---

### 2.4 Modelfile

A configuration file that defines how a model should behave, respond and be used. It acts as an instruction layer on top of an existing base model.

**A modelfile usually defines:**

- Which base model to use
- What system instructions to inject
- How responses should be structured
- What constraints exist
- What defaults should be applied

```
FROM llama3.2

# sets the temperature to 1 [higher is more creative, lower is more coherent]
PARAMETER temperature 1

# sets the context window size to 4096, this controls how many tokens the LLM can use as context to generate the next token
PARAMETER num_ctx 4096

# sets a custom system message to specify the behavior of the chat assistant
SYSTEM You are Mario from super mario bros, acting as an assistant.
```

Ollama creates a new model using modelfile which we can use for our specific use case.

**Command for using a modelfile:**

```bash
ollama create new_model_name -f modelfile_name
```

A customized LLM can also be created through Python (Ollama docs).

> Internally our LLM is running on port `11434` locally with which we can interact through CLI or Python library.

---

### 2.5 Generate using REST API

```python
import requests
import json

url = "http://localhost:11434/api/generate"

payload = {
    "model": "llama3.2:1b",
    "prompt": "Explain black holes simply"
}

response = requests.post(url, json=payload)

for i in response.iter_lines():  # ollama sends multiple json lines
    print(i)
```

---

### 2.6 Using Ollama in an Orchestration Framework

**For Chat Conversation:**

```python
from langchain_ollama import ChatOllama

# Initialize the model
llm = ChatOllama(
    model="llama3.2:1b",
    temperature=0,
)

# Simple invocation
response = llm.invoke("Explain the concept of quantum entanglement in one sentence.")
print(response.content)
```

**For Non-Chat Execution:**

```python
from langchain_ollama import OllamaLLM

llm = OllamaLLM(model="llama3.2:1b")

response = llm.invoke("The capital of France is")
print(response)
```

---

### 2.7 Ollama Cloud

Models with large parameters can't be run locally due to hardware constraints and Ollama Cloud solves this problem.

Ollama Cloud is a cloud-based extension of the Ollama platform that lets us run LLMs without needing powerful local hardware.

> ⚠️ Only models which have cloud capabilities can be run in Ollama Cloud

**Steps to run a model on Ollama Cloud:**

1. In command prompt, use the command `ollama signin`. It will give a URL, copy that and paste it in the browser and connect to Ollama Cloud.
2. To run a particular model, copy its command from Ollama and use it in command prompt — it will connect you to the model.

> 📝 **Note:** Ollama Cloud does not retain any of our requests.
