# 🤖 Crew AI

> A reference guide covering Crew AI — a Python framework used to build multi-agent systems — including components of AI agents, CrewAI philosophy, and project setup.

---

## 📑 Table of Contents

1. [Evolution of Capabilities of LLM](#1-evolution-of-capabilities-of-llm)
2. [Components of an AI Agent](#2-components-of-an-ai-agent)
   - [2.1 Memory](#21-memory)
   - [2.2 Tool](#22-tool)
3. [Characteristics of AI Agents](#3-characteristics-of-ai-agents)
4. [Philosophy Behind CrewAI](#4-philosophy-behind-crewai)
5. [Components of CrewAI](#5-components-of-crewai)
   - [5.1 Agent](#51-agent)
   - [5.2 Tasks](#52-tasks)
   - [5.3 Crew](#53-crew)
6. [Steps to Install CrewAI](#6-steps-to-install-crewai)
7. [Creating a New Project in CrewAI](#7-creating-a-new-project-in-crewai)

---

## 1. Evolution of Capabilities of LLM

- LLMs were good at interacting in natural language
- Then LLMs became good at **structured output** which made them better at interacting with other machines
- Then LLMs also became good at **reasoning** through chain of thought process (breaking a complex prompt into sub steps for better inference)
- Then came the concept of **tools** through which LLMs can execute some tasks apart from generating natural language responses
- This further evolved into concepts of **AI agents** which can perform various tasks

---

## 2. Components of an AI Agent

| Component | Description |
|-----------|-------------|
| **Brain** | Gives cognitive abilities to our agent (LLMs) |
| **Memory** | Gives knowledge and context to our agent |
| **Tool** | Gives ability to agent to perform tasks |

**Brain** capabilities include:

- **Understand the intent** of the user
- **Generation capabilities** — enables the agent to produce relevant responses, content, or actions based on user input
- **Planning** — allows the agent to break down a goal into structured steps and decide the sequence of actions
- **Reasoning** — helps in how to execute a particular step of the goal

### 2.1 Memory

Memory consists of:

- **Internal knowledge** of agent (parametric knowledge of LLM)
- **External knowledge** which is gained from user or external documents

**3 types of memory:**

| Type | Description |
|------|-------------|
| Short term memory | To store information to perform the current task |
| Long term memory | To exchange information between sessions or tasks |
| Entity based memory | Stores user information or preferences |

### 2.2 Tool

Tools are **functions/APIs**.

Convert a function to tool using `@tool` decorator, docstring to describe the tool, and type hints to let the LLM know what we are expecting in input and output.

To use the tool, LLM uses the technique of **tool call** which uses a schema in form of a JSON:

```json
{
  "name": "get_weather",
  "description": "Get current weather for a given city",
  "parameters": {
    "type": "object",
    "properties": {
      "city": {
        "type": "string",
        "description": "Name of the city"
      },
      "unit": {
        "type": "string",
        "enum": ["celsius", "fahrenheit"],
        "description": "Temperature unit"
      }
    },
    "required": ["city"]
  }
}
```

---

## 3. Characteristics of AI Agents

| Characteristic | Description |
|----------------|-------------|
| **Goal oriented** | It doesn't just respond to prompts — it tries to achieve an outcome. Goal is the primary motivation of an agent. |
| **Autonomy** | Agents can execute tasks without any human intervention or minimal human intervention |
| **Planning** | Break a complex task into easier subtasks. This is done by the LLM. Planning involves reasoning at global level. |
| **Reasoning** | Cognitive ability of an agent to perform certain tasks. It helps in devising a way to complete a certain task. |
| **Adaptability** | Makes the agent adaptable to roadblocks in the execution |
| **Orchestration** | Agent also manages the task execution order |

---

## 4. Philosophy Behind CrewAI

- Using CrewAI as a tool which we can access through **command line interface**
- Provides a **readymade project template** which suits building multi agent systems
- **Team structure** in CrewAI mimics a human team
- Instead of a single agent system which can perform various tasks, we have a **multi agent system** in which agents are specialized in performing a specific set of tasks

---

## 5. Components of CrewAI

### 5.1 Agent

**Agent** — entity which performs actions or a given task.

It has 3 attributes:

| Attribute | Description |
|-----------|-------------|
| **Role** | Role of agent inside crew |
| **Goal** | Goal of agent |
| **Backstory** | Persona and capability of a particular agent |

### 5.2 Tasks

**Task** — entity given to agent to perform actions.

| Attribute | Description |
|-----------|-------------|
| **Description** | What to achieve and series of steps to achieve this task |
| **Expected output** | What the output of the task will look like |
| **Agent** | Which agent will perform this particular task |

### 5.3 Crew

**Crew** — Team of agents.

| Attribute | Required | Description |
|-----------|----------|-------------|
| **Tasks** | Required | A list of tasks assigned to the crew |
| **Agents** | Required | A list of agents that are part of the crew |
| **Process** | Optional | The process flow (e.g., sequential, hierarchical) the crew follows. Default is sequential. |
| **Verbose** | Optional | The verbosity level for logging during execution. Defaults to False. |
| **manager_LLM** | Optional | The language model used by the manager agent in a hierarchical process. Required when using a hierarchical process. |
| **function_calling_llm** | Optional | If passed, the crew will use this LLM to do function calling for tools for all agents in the crew. Each agent can have its own LLM, which overrides the crew's LLM for function calling. |
| **config** | Optional | Optional configuration settings for the crew, in Json or `Dict[str, Any]` format. |
| **memory** | Optional | Utilized for storing execution memories (short-term, long-term, entity memory). |

---

## 6. Steps to Install CrewAI

**Install uv:**

```bash
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

**Install crewAI:**

```bash
uv tool install crewai
```

---

## 7. Creating a New Project in CrewAI

**Create a new project:**

```bash
crewai create crew <your_project_name>
```

Your project will contain these essential files:

| File/Directory | Purpose |
|----------------|---------|
| `agents.yaml` | Define your AI agents and their roles |
| `tasks.yaml` | Set up agent tasks and workflows |
| `.env` | Store API keys and environment variables |
| `main.py` | Project entry point and execution flow |
| `crew.py` | Crew orchestration and coordination |
| `tools/` | Directory for custom agent tools |
| `knowledge/` | Directory for knowledge base |

**Create a virtual environment before running your project:**

```bash
crewai install
```

**To add additional packages to project:**

```bash
uv add <package-name>
```

---

**Defining the class for our crew:**

```python
@CrewBase
class ResearchAndBlogCrew():
    agents: list[BaseAgent]
    tasks: list[Task]

    # define the paths of config files
    agents_config = "config/agents.yaml"
    tasks_config = "config/tasks.yaml"
```

**Defining our agents:**

```python
@agent
def report_generator(self) -> Agent:
    return Agent(
        config=self.agents_config["report_generator"]
    )

@agent
def blog_writer(self) -> Agent:
    return Agent(
        config=self.agents_config["blog_writer"]
    )
```

**Defining our tasks:**

```python
# order of task definition matters
@task
def report_task(self) -> Task:
    return Task(
        config=self.tasks_config["report_task"]
    )

@task
def blog_writing_task(self) -> Task:
    return Task(
        config=self.tasks_config["blog_writing_task"],
        output_file="blogs/blog.md"
    )
```

**Defining our crew:**

```python
@crew
def crew(self) -> Crew:
    return Crew(
        agents=self.agents,
        tasks=self.tasks,
        process=Process.sequential,
        verbose=True
    )
```

**To run our crew — in `main.py`:**

```python
from reaseach_and_blog_crew.crew import ResearchAndBlogCrew

def run():
    """
    Run the crew.
    """
    inputs = {
        'topic': 'AI agents in coding'
    }
    try:
        ResearchAndBlogCrew().crew().kickoff(inputs=inputs)
    except Exception as e:
        raise Exception(f"An error occurred while running the crew: {e}")
```

**Then use the command in root directory of the project:**

```bash
crewai run
```
