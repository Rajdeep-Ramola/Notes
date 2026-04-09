# 🤖 LLMOps (Large Language Model Operations)

> A reference guide covering LLMOps — a new set of tools and best practices to manage the lifecycle of LLM-powered applications, including development, deployment, and maintenance.

---

## 📑 Table of Contents

1. [Steps in LLMOps](#1-steps-in-llmops)
   - [1.1 Selection of a Foundational Model](#11-selection-of-a-foundational-model)
   - [1.2 Adaptation to Downstream Tasks](#12-adaptation-to-downstream-tasks)
   - [1.3 Evaluation](#13-evaluation)
   - [1.4 Deployment and Monitoring](#14-deployment-and-monitoring)
2. [How LLMOps is Different than MLOps](#2-how-llmops-is-different-than-mlops)
   - [2.1 Data Management](#21-data-management)
   - [2.2 Experimentation](#22-experimentation)
   - [2.3 Evaluation](#23-evaluation)
   - [2.4 Cost](#24-cost)
   - [2.5 Latency](#25-latency)
3. [Tools Used for LLMOps](#3-tools-used-for-llmops)

---

## 1. Steps in LLMOps

LLMOps is MLOps for LLMs — MLOps (machine learning operations) is a set of tools and best practices to manage the lifecycle of ML-powered applications.

### 1.1 Selection of a Foundational Model

Foundation models are LLMs pre-trained on large amounts of data that can be used for a wide range of downstream tasks.

Currently, developers have to choose between two types of foundation models:

| Type | Description |
|------|-------------|
| **Proprietary Models** | Closed-source models offered via APIs (e.g., GPT-4, Claude) |
| **Open-Source Models** | Freely available models that can be self-hosted (e.g., LLaMA, Mistral) |

---

### 1.2 Adaptation to Downstream Tasks

Getting the output from the LLM API in your desired format might take some iterations. LLMs can also hallucinate if they don't have the required specific knowledge.

To combat these concerns, you can adapt the foundation models to downstream tasks in the following ways:

**Prompt Engineering**

Craft and iterate on prompts to guide LLM behavior without modifying model weights.

**Fine-tuning Pre-trained Models**

Update model weights on domain-specific data for more targeted performance.

---

### 1.3 Evaluation

To help evaluate LLMs, dedicated tools have emerged.

> Tools like **HoneyHive** or **HumanLoop** are used to evaluate LLM outputs at scale.

---

### 1.4 Deployment and Monitoring

LLM-powered applications require monitoring of the changes in the underlying API model.

> Tracking API model versions, latency, cost, and output quality is essential for production LLM applications.

---

## 2. How LLMOps is Different than MLOps

### 2.1 Data Management

| | MLOps | LLMOps |
|---|-------|--------|
| **Data Volume** | Requires a huge amount of data | Requires few but hand-picked samples |
| **Data Quality** | Data usually has imperfections | Curated data for desired results |
| **Example** | Thousands of labeled examples needed | Prompt engineering requires fewer attempts to get desired results |

---

### 2.2 Experimentation

**MLOps** tracks inputs such as:
- Model architecture
- Hyperparameters
- Data augmentation

**LLMOps** fine-tuning is similar to MLOps, but prompt engineering requires a different experimentation setup including **management of prompts**.

---

### 2.3 Evaluation

| | MLOps | LLMOps |
|---|-------|--------|
| **Metrics** | Fixed evaluation metrics | Generally relies on A/B testing |

---

### 2.4 Cost

| | MLOps | LLMOps |
|---|-------|--------|
| **Primary Cost Driver** | Data collection and model training | Inference |

---

### 2.5 Latency

> Latency is much more evident and critical in LLMOps compared to traditional MLOps pipelines.

---

## 3. Tools Used for LLMOps

### Langfuse

A comprehensive LLMOps platform offering:

**Observability**
- Tracing
- Logs
- Latency tracking
- Cost monitoring

**Prompt Management**
- Versioning
- Prompt registry

**Evaluation**
- LLM-as-a-judge
- Human feedback
- Custom metrics

**Experimentation & Datasets**
- Dataset management
- Experiment tracking

**Playground**
- Interactive environment for testing and iterating on prompts

---

> **Alternative Tool:** [Arize AI](https://arize.com/) — another LLMOps observability and evaluation platform.
