# 📘 Agentic Design Patterns

1. [Agent Fundamentals](#1-agent-fundamentals)
   - [1.1 The Five-Step Loop](#11-the-five-step-loop)
   - [1.2 Levels of Agent Complexity](#12-levels-of-agent-complexity)
   - [1.3 Context Engineering](#13-context-engineering)
2. [Prompt Chaining Pattern](#2-prompt-chaining-pattern)
   - [2.1 Core Concept](#21-core-concept)
   - [2.2 Why Single Prompts Fail](#22-why-single-prompts-fail)
   - [2.3 Use Cases](#23-use-cases)
   - [2.4 When to Use](#24-when-to-use)
3. [Routing Pattern](#3-routing-pattern)
   - [3.1 Core Concept](#31-core-concept)
   - [3.2 Routing Methods](#32-routing-methods)
   - [3.3 When to Use](#33-when-to-use)
4. [Parallelization Pattern](#4-parallelization-pattern)
   - [4.1 Core Concept](#41-core-concept)
   - [4.2 Use Cases](#42-use-cases)
   - [4.3 Technical Note](#43-technical-note)
5. [Reflection Pattern](#5-reflection-pattern)
   - [5.1 Core Concept](#51-core-concept)
   - [5.2 Two-Role Implementation](#52-two-role-implementation)
   - [5.3 Memory & Trade-offs](#53-memory--trade-offs)
6. [Tool Use Pattern](#6-tool-use-pattern)
   - [6.1 Core Concept & Mechanism](#61-core-concept--mechanism)
   - [6.2 What Counts as a Tool](#62-what-counts-as-a-tool)
   - [6.3 Use Cases](#63-use-cases)
   - [6.4 When to Use](#64-when-to-use)
7. [Planning Pattern](#7-planning-pattern)
   - [7.1 Core Concept](#71-core-concept)
   - [7.2 Application Domains](#72-application-domains)
   - [7.3 When to Use](#73-when-to-use)
8. [Multi-Agent Systems](#8-multi-agent-systems)
   - [8.1 Core Concept](#81-core-concept)
   - [8.2 Collaboration Modes](#82-collaboration-modes)
   - [8.3 Agent Network Topologies](#83-agent-network-topologies)
   - [8.4 Task Decomposition](#84-task-decomposition)
9. [Memory Pattern](#9-memory-pattern)
   - [9.1 Short-Term Memory](#91-short-term-memory)
   - [9.2 Long-Term Memory](#92-long-term-memory)
   - [9.3 Memory Types](#93-memory-types)
   - [9.4 Learning Modes](#94-learning-modes)
   - [9.5 Reinforcement Learning Algorithms](#95-reinforcement-learning-algorithms)
   - [9.6 When to Use](#96-when-to-use)
10. [Agent-to-Agent (A2A) Protocol](#10-agent-to-agent-a2a-protocol)
    - [10.1 Core Concept](#101-core-concept)
    - [10.2 Key Participants](#102-key-participants)
    - [10.3 Agent Discovery](#103-agent-discovery)
    - [10.4 Communication Modes](#104-communication-modes)
    - [10.5 Security](#105-security)
11. [Resource Management & Compute Scaling](#11-resource-management--compute-scaling)
    - [11.1 Core Concept](#111-core-concept)
    - [11.2 Test-Time Compute Scaling](#112-test-time-compute-scaling)
    - [11.3 Reasoning Techniques](#113-reasoning-techniques)
12. [Agent Engineering Best Practices](#12-agent-engineering-best-practices)
    - [12.1 Checkpointing & Rollback](#121-checkpointing--rollback)
    - [12.2 Modularity & Security](#122-modularity--security)
    - [12.3 Monitoring & Evaluation](#123-monitoring--evaluation)
    - [12.4 Multi-Agent Evaluation](#124-multi-agent-evaluation)
13. [Contractor Agent Pattern](#13-contractor-agent-pattern)
    - [13.1 Core Concept](#131-core-concept)
    - [13.2 Four Pillars](#132-four-pillars)
14. [Task Prioritization Pattern](#14-task-prioritization-pattern)
    - [14.1 Core Concept](#141-core-concept)
    - [14.2 Prioritization Elements](#142-prioritization-elements)
    - [14.3 Use Cases](#143-use-cases)
15. [Exploration & Discovery Pattern](#15-exploration--discovery-pattern)
    - [15.1 Core Concept](#151-core-concept)
    - [15.2 Use Cases](#152-use-cases)
    - [15.3 Real-World Example: Google Co-Scientist](#153-real-world-example-google-co-scientist)
16. [Multi-Agent System Search (MASS) Framework](#16-multi-agent-system-search-mass-framework)
    - [16.1 Core Concept](#161-core-concept)
    - [16.2 Three-Stage Optimization](#162-three-stage-optimization)
    - [16.3 Key Principles](#163-key-principles)
17. [Prompting Techniques](#17-prompting-techniques)
    - [17.1 Fundamentals](#171-fundamentals)
    - [17.2 Shot-Based Prompting](#172-shot-based-prompting)
    - [17.3 Prompt Structure & Roles](#173-prompt-structure--roles)
    - [17.4 Reasoning Prompting Techniques](#174-reasoning-prompting-techniques)
    - [17.5 Automated Prompt Optimization](#175-automated-prompt-optimization)
18. [GUI/Computer Use Agents](#18-guicomputer-use-agents)

---

## 1. Agent Fundamentals

### 1.1 The Five-Step Loop

An AI agent operates on a **five-step loop** to get things done:

1. **Receive goal** — You give it a goal.
2. **Gather information** — It gathers all the necessary information (reading, etc.).
3. **Plan** — It devises a plan of action by considering the optimal approach.
4. **Execute** — It executes the plan.
5. **Observe & Adapt** — It observes successful outcomes and adapts accordingly.

---

### 1.2 Levels of Agent Complexity

There is a **range of agent complexity**:

| Level | Description |
|-------|-------------|
| **Level 0** | LLM operates without tools, memory, or environment interaction — responds solely from pre-trained knowledge. Trade-off: complete lack of current-event awareness. |
| **Level 1** | LLM becomes a functional agent by connecting to external tools. Can execute a sequence of actions to gather and process information from sources like the internet and databases. |
| **Level 2** | Agent moves beyond single-tool use to tackle complex, multi-part problems through strategic problem-solving. |
| **Level 3** | Agent achieves **self-improvement** by refining its own context engineering processes. Automatically improves how it packages information for future tasks. |

> *When it asks for feedback on how a prompt could have been improved, it is learning how to better curate its initial inputs.*

---

### 1.3 Context Engineering

**Context engineering** is the strategic process of selecting, packaging, and managing the most relevant information for each step.

The trend is moving **away** from the pursuit of a single, all-powerful super-agent and **towards** the rise of sophisticated, collaborative multi-agent systems.

---

## 2. Prompt Chaining Pattern

### 2.1 Core Concept

Rather than expecting an LLM to solve a complex problem in a single, monolithic step, **prompt chaining** advocates for a **divide-and-conquer strategy**:

- Break down the original problem into a **sequence of smaller, more manageable sub-problems**.
- Each sub-problem is addressed individually through a specifically designed prompt.
- The output from one prompt is strategically fed as input into the subsequent prompt in the chain.
- Prompt chaining also enables the **integration of external knowledge and tools** — it's not just about breaking down problems.

---

### 2.2 Why Single Prompts Fail

**Limitations of single prompts:** For multifaceted tasks, a single complex prompt causes the model to struggle with constraints and instructions, potentially leading to *instruction neglect*.

- The **reliability of a prompt chain** is highly dependent on the integrity of the data passed between steps.
- **Specifying a structured output format** (such as JSON or XML) between steps is crucial.

Prompt chaining addresses these challenges by breaking the complex task into a focused, sequential workflow, which **significantly improves reliability and control**.

---

### 2.3 Use Cases

- Processing raw information through multiple transformations
- Automated content analysis
- AI-driven research assistants
- Conversion of unstructured text into a structured format
- Composition of complex content (decomposed into distinct phases)
- Generation of functional code (multi-stage, discrete logical operations)
- Analyzing datasets with diverse modalities

---

### 2.4 When to Use

Use the Prompt Chaining pattern when:
- A task is **too complex for a single prompt**
- It involves **multiple distinct processing stages**
- It requires **interaction with external tools** between steps

The quality of a model's output is less dependent on the model's architecture and more on the **richness of the context provided** — including system prompt, retrieved documents, tool outputs, user identity, interaction history, and environmental state.

---

## 3. Routing Pattern

### 3.1 Core Concept

**Routing** introduces conditional logic into an agent's operational framework. The agent dynamically evaluates specific criteria to select from a set of possible subsequent actions — the LLM *performs the evaluation and directs the flow*.

---

### 3.2 Routing Methods

| Method | Description | Trade-off |
|--------|-------------|-----------|
| **LLM-based** | Language model itself is prompted to analyze the input and route. | Flexible, handles nuanced inputs. |
| **Semantic/Embedding** | Input query is converted into a vector embedding, compared to route embeddings; routed to the most similar. Useful for **semantic routing** (based on *meaning*, not keywords). | High accuracy for meaning-based decisions. |
| **Rule-based** | Uses predefined rules or logic based on keywords, patterns, or structured data. | **Faster**, but less flexible for nuanced or novel inputs. |
| **Classifier-based** | Employs a discriminative model (e.g., a classifier) to perform routing. LLMs are not involved in the real-time routing decision itself. | Efficient and deterministic. |

Routing is most commonly employed to **interpret user intent**.

---

### 3.3 When to Use

Use the **Routing pattern** when an agent must decide between multiple distinct workflows, tools, or sub-agents based on the user's input or the current state.

---

## 4. Parallelization Pattern

### 4.1 Core Concept

Many complex agentic tasks involve multiple sub-tasks that can be **executed simultaneously** rather than one after another.

- Parallel execution allows independent tasks to run at the same time, significantly **reducing overall execution time** for tasks that can be broken down into independent parts.
- The key is to **identify parts of the workflow that do not depend on the output of other parts** and execute them in parallel.
- Particularly effective when dealing with **external services** (APIs, databases).

---

### 4.2 Use Cases

- **Information Gathering & Research:** Processing different data segments concurrently
- Calling multiple independent APIs or tools simultaneously
- Generating different parts of a complex piece of content in parallel
- Performing multiple independent checks
- Processing different modalities of the same input concurrently
- Generating multiple variations of a response or output in parallel to select the best one

---

### 4.3 Technical Note

> `asyncio` provides **concurrency**, not parallelism.

---

## 5. Reflection Pattern

### 5.1 Core Concept

The **Reflection pattern** involves an agent evaluating its own work, output, or internal state, and then using that evaluation to improve its performance or refine its response.

- The agent doesn't just produce an output — **it then examines that output**.

---

### 5.2 Two-Role Implementation

A key and highly effective implementation separates the process into **two distinct logical roles**:

1. **Generator** — Produces the initial output.
2. **Critic** — Evaluates the output and provides feedback for refinement.

---

### 5.3 Memory & Trade-offs

- The effectiveness of the Reflection pattern is significantly enhanced when the LLM **keeps a memory of the conversation**. Without memory, each reflection is a self-contained event.
- **Trade-off:** The iterative process, though powerful, can lead to **higher costs and latency** since every refinement loop may require a new LLM call — making it suboptimal for time-sensitive applications.

---

## 6. Tool Use Pattern

### 6.1 Core Concept & Mechanism

Also known as **Function Calling** or the **Tool Use pattern**, it is implemented through this flow:

1. External functions or capabilities are **defined and described** to the LLM (name, purpose, parameters, types, descriptions).
2. The LLM receives the user's request and available tool definitions.
3. Based on its understanding, the LLM **decides if calling one or more tools is necessary**.
4. If yes, it generates a **structured output** specifying the tool name and arguments extracted from the request.
5. The agentic framework **intercepts this structured output**, identifies the tool, and **executes the actual external function**.
6. The result is returned to the agent.
7. The LLM receives the tool's output as context and uses it to **formulate a final response** or decide on the next step.

---

### 6.2 What Counts as a Tool

A "tool" can be:
- A traditional function
- A complex API endpoint
- A database request
- An instruction directed at another specialized agent

---

### 6.3 Use Cases

- **Real-time data access** not present in the LLM's training data
- **Database operations** — queries, updates, and other operations on structured data
- **Computation** — external calculators, data analysis libraries, or statistical tools
- **Communication** — sending emails, messages, or making API calls to external services
- **Code execution** in a safe environment
- **IoT interaction** — smart home devices, IoT platforms, or other connected systems

---

### 6.4 When to Use

Use the **Tool Use pattern** whenever an agent needs to break out of the LLM's internal knowledge and interact with the outside world.

---

## 7. Planning Pattern

### 7.1 Core Concept

**Planning** is the ability for an agent or system of agents to formulate a sequence of actions to move from an **initial state** towards a **goal state**.

- An initial plan is merely a **starting point, not a rigid script**.
- The agent's real power is its ability to **incorporate new information and steer the project around obstacles**.
- When a problem's solution is already well-understood and repeatable, constraining the agent to a **predetermined, fixed workflow** is more effective.

---

### 7.2 Application Domains

- **Procedural task automation** — orchestrating complex workflows
- **Robotics and autonomous navigation** — fundamental for state-space traversal
- **Structured information synthesis**

---

### 7.3 When to Use

Without a structured approach, an agentic system struggles to handle multifaceted requests that involve multiple steps and dependencies.

Use this pattern when a user's request is **too complex to be handled by a single action or tool**.

---

## 8. Multi-Agent Systems

### 8.1 Core Concept

Multi-agent design involves systems where **multiple independent or semi-independent agents work together** to achieve a common goal.

- Each agent typically has a **defined role**, specific goals aligned with the overall objective, and potentially access to different tools or knowledge bases.

---

### 8.2 Collaboration Modes

| Mode | Description |
|------|-------------|
| **Pipeline** | One agent completes a task and passes its output to another agent for the next step. |
| **Parallel** | Multiple agents work on different parts of a problem simultaneously. |
| **Collaborative Debate** | Agents with varied perspectives engage in discussions to evaluate options. |
| **Hierarchical Delegation** | A manager agent delegates tasks to worker agents dynamically based on their tool access. |
| **Specialized Domain** | Agents with specialized knowledge in different domains collaborate. |
| **Review-Revise Loop** | Agents create initial outputs; a second group critically assesses them for quality and compliance; the original creator revises based on feedback. |

---

### 8.3 Agent Network Topologies

| Topology | Description |
|----------|-------------|
| **Single Agent** | Operates autonomously without direct interaction or communication with other entities. |
| **Peer-to-Peer** | Multiple agents interact directly with each other in a decentralized fashion. |
| **Supervisor** | A dedicated "supervisor" agent oversees and coordinates the activities of subordinate agents. |
| **Mentor/Support** | The supervisor's role is less about direct command and more about providing resources, guidance, or analytical support. |
| **Custom** | Allows for unique interrelationship and communication structures tailored to specific requirements. |

---

### 8.4 Task Decomposition

- A complex problem is broken down into **smaller, more manageable sub-problems**.
- Each sub-problem is then assigned to a **specialized agent** with the precise tools and capabilities required to solve it.

---

## 9. Memory Pattern

### 9.1 Short-Term Memory

**Memory** refers to an agent's ability to retain and utilize information from past interactions, observations, and learning experiences.

**Short-term memory** primarily exists within the **context window**. It contains:
- Recent messages
- Agent replies
- Tool usage results
- Agent reflections from the current interaction

Efficient short-term memory management involves:
- Keeping the **most relevant information** within the limited context window
- **Summarizing older conversation segments**
- **Emphasizing key details**

> Note: Context is still ephemeral and is **lost once the session concludes**.

---

### 9.2 Long-Term Memory

Long-term memory holds information agents need to **retain across various interactions**. Data is typically stored in **databases**.

When an agent needs information from long-term memory, it:
1. Queries the external storage
2. Retrieves relevant data
3. Integrates it into the short-term context for immediate use

---

### 9.3 Memory Types

| Type | Description |
|------|-------------|
| **Semantic** | Retaining specific facts and concepts (user preferences, domain knowledge) |
| **Episodic** | Recalling past events or actions |
| **Procedural** | Memory of how to perform tasks |

---

### 9.4 Learning Modes

| Mode | Description |
|------|-------------|
| **Reinforcement Learning** | Agents try actions and receive rewards for positive outcomes and penalties for negative ones. |
| **Supervised Learning** | Agents learn from labeled examples, connecting inputs to desired outputs. |
| **Unsupervised Learning** | Agents discover hidden connections and patterns in unlabeled data. |
| **Few-Shot / In-Context** | Agents leveraging LLMs can quickly adapt to new tasks with minimal examples. |
| **Online Learning** | Agents continuously update knowledge with new data — essential for real-time reactions in dynamic environments. |
| **Case-Based Reasoning** | Agents recall past experiences to adjust current actions in similar situations. |

---

### 9.5 Reinforcement Learning Algorithms

**PPO (Proximal Policy Optimization)**
- Used in environments with a continuous range of actions (e.g., controlling a robot's joints).
- Makes **small, careful updates** to the agent's policy, creating a "trust region" to avoid drastic changes that could collapse performance.
- Flow: agent interacts → uses current policy → collects a batch of experiences → calculates policy update → checks it stays within the trust region → applies the update.

**RLHF (Reinforcement Learning from Human Feedback)**
- Designed for **aligning LLMs with human preferences**.
- Steps: collect human feedback (ratings/comparisons) → train a **reward model** to predict human scores → train the LLM to maximize reward model scores.

**DPO (Direct Preference Optimization)**
- **Skips the reward model entirely** — uses preference data directly to update the LLM's policy.
- Links preference data to the optimal policy, teaching the model to increase probability of preferred responses and decrease probability of disfavored ones.

---

### 9.6 When to Use

Use the Memory pattern when an agent needs to do **more than answer a single question** — particularly for multi-step tasks, personalized assistance, or continuous adaptation.

---

## 10. Agent-to-Agent (A2A) Protocol

### 10.1 Core Concept

The **(A2A) protocol** is an **open standard** designed to enable communication and collaboration between different AI agent frameworks.

---

### 10.2 Key Participants

| Role | Description |
|------|-------------|
| **User** | Initiates requests for agent assistance. |
| **A2A Client** | AI agent that acts on the user's behalf to request actions or information. |
| **A2A Server (Remote Agent)** | AI agent or system providing an HTTP endpoint to process client requests and return results. Operates as an **"opaque" system** — the client does not need to understand its internal operational details. |

An agent's **digital identity** is defined by its **Agent Card** (usually a JSON file).

---

### 10.3 Agent Discovery

| Method | Description |
|--------|-------------|
| **Standardized Path** | Agents host their Agent Card at a standardized path for clients to find. |
| **Registry** | A centralized catalog where Agent Cards are published and queried. Well-suited for enterprise environments needing centralized management. |
| **Private Sharing** | Agent Card information is embedded or privately shared. Appropriate for closely coupled or private systems. |

---

### 10.4 Communication Modes

| Mode | Description |
|------|-------------|
| **Synchronous Request/Response** | Client sends a request and actively waits for a complete response in a single exchange. |
| **Asynchronous Polling** | Client sends a request; server immediately returns a "working" status and task ID. Client polls for completion status. |
| **Streaming (Server-Sent Events)** | One-way connection from server to client, ideal for real-time, incremental results. Server pushes updates continuously without requiring multiple client requests. |
| **Webhooks** | For very long-running tasks. Client registers a webhook URL; server sends an async notification to that URL when the task status changes significantly. |

The Agent Card specifies whether an agent supports streaming.

---

### 10.5 Security

- **Mutual Transport Layer Security (mTLS):** Encrypted and authenticated connections prevent unauthorized access and data interception.
- **Comprehensive Logging:** All inter-agent communications are meticulously recorded.
- **Declared Authentication:** Requirements are explicitly declared in the Agent Card. Agents authenticate using secure credentials like **OAuth 2.0 tokens or API keys**, passed via HTTP headers to prevent credential exposure in URLs or message bodies.

---

## 11. Resource Management & Compute Scaling

### 11.1 Core Concept

Agents must make decisions regarding **action execution to achieve goals within specified resource budgets** and optimize efficiency.

Key strategy: the **fallback mechanism** — an intelligent system assesses query difficulty and routes accordingly:
- **Simple queries** → cost-effective language model
- **Complex inquiries** → more powerful (but expensive) language model

Agents can also:
- Intelligently choose from a suite of tools
- Intelligently summarize and selectively retain only the most relevant information from interaction history
- Distribute computational workloads across multiple machines or processors
- Adapt and optimize resource allocation strategies over time based on feedback and performance metrics

---

### 11.2 Test-Time Compute Scaling

**Test-Time Compute (TTC) Scaling** is the allocation of increased computational resources *during inference*:
- Granting the agent **more processing time or steps** to process a query and generate a response.
- Extended processing time during inference **often significantly enhances accuracy**.

The **Scaling Inference Law for LLMs** states that a model's performance **predictably improves** as the computational resources allocated to it increase. Specifically, this law examines the dynamic trade-offs when an LLM is actively generating an output.

> Superior results can frequently be achieved from a *comparatively smaller LLM* by **augmenting the computational investment at inference time**.

One approach: instruct the model to generate multiple potential answers (e.g., via **diverse beam search**) and then employ a **selection mechanism** to identify the most optimal output.

---

### 11.3 Reasoning Techniques

**Chain-of-Thought (CoT)**
- Instead of providing a direct answer, CoT prompts guide the model to generate a **sequence of intermediate reasoning steps**.
- *Zero-shot CoT:* Simply adding "Let's think step by step" — no examples needed.
- *Few-shot CoT:* Providing several examples of input + step-by-step reasoning + output.

**Limitation:** Standard CoT generates a **single, predetermined line of thought** without adapting to the complexity of the problem.

**Reasoning Models**
- A specialized category of models that dedicate a **variable amount of "thinking" time** before providing an answer.
- Produces a **more extensive and dynamic Chain-of-Thought** that can be thousands of tokens long.

**Tree of Thoughts (ToT)**
- Enables a language model to explore **multiple reasoning paths concurrently**.
- Supports complex problem-solving by enabling **backtracking, self-correction, and exploration of alternative solutions**.
- Evaluates various reasoning trajectories before finalizing an answer.
- Enhances the model's ability to handle challenging tasks requiring **strategic planning and decision-making**.

**ReAct (Reason + Act)**
- Integrates Chain-of-Thought prompting with an agent's ability to **interact with external environments through tools**.
- Flow: **Reason** (think about which actions to take) → **Act** (execute an action) → **Observe** (observe the outcome) → **Incorporate** (use observation in subsequent reasoning).

**Program-Aided Language (PAL) / Code Interpreter**
- Integrates LLMs with **symbolic reasoning capabilities**.
- Allows the LLM to **generate and execute code** (e.g., Python) as part of its problem-solving process.
- Offloads complex calculations and data manipulation to a **deterministic programming environment**.
- Flow: faced with symbolic challenges → model produces code → executes it → converts results into natural language.

**Self-Consistency**
- Generates **multiple diverse reasoning paths** for the same problem and selects the most consistent answer.
- Improves accuracy particularly for tasks where multiple valid reasoning paths might exist.

**Analogical Reasoning**
- Framing a task using an analogy helps the model understand the desired output by relating it to something familiar.
- Particularly useful for **creative tasks**.

**Society of Mind / Multi-Model Debate**
- AI framework where **multiple, diverse models collaborate and argue** to solve a problem, moving beyond a single AI's chain of thought.

**Argumentation Graph Networks**
- Reimagines discussion as a **dynamic, non-linear network** rather than a simple chain.
- Arguments are individual nodes connected by edges signifying relationships like 'supports' or 'refutes'.
- Conclusion is reached by identifying the **most robust and well-supported cluster** of arguments within the entire graph.

---

## 12. Agent Engineering Best Practices

### 12.1 Checkpointing & Rollback

Autonomous agents manage complex states and can head in unintended directions. **Implementing checkpoints** is akin to designing a transactional system with **commit and rollback capabilities**:

- Each checkpoint is a **validated state** (a successful "commit" of the agent's work).
- A **rollback** is the mechanism for fault tolerance.

---

### 12.2 Modularity & Security

**Modularity and Separation of Concerns:**
- Modularity in multi-agentic systems enhances performance by **enabling parallel processing**.
- Engineers need **structured logs** that capture the agent's entire "chain of thought."

**Principle of Least Privilege:**
- An agent should be granted the **absolute minimum set of permissions** required to perform its task.

---

### 12.3 Monitoring & Evaluation

Key metrics to monitor:
- **Accuracy** — Does the agent deliver pertinent, correct, logical, unbiased, and accurate information?
- **Latency** — Measuring the duration required for an agent to process requests and generate outputs.
- **Resource Consumption** — Tracking token usage (e.g., via a class like `LLMInteractionMonitor`).

Strategies:
- **A/B Testing** — Systematically comparing the performance of different agent versions or strategies in parallel to identify optimal approaches.
- **Comparison methods** — Exact match, in-order match, any-order match, precision, and recall.
- **Test files & Evalset files** — Each test file contains a single session with multiple turns (user query, expected tool use trajectory, intermediate agent responses, and final output).

---

### 12.4 Multi-Agent Evaluation

Key questions to assess multi-agent systems:
- Are the agents **cooperating effectively**?
- Did they create a good plan and **stick to it**?
- Is the **right agent being chosen** for the right task?

---

## 13. Contractor Agent Pattern

### 13.1 Core Concept

The **evolution from simple AI agents to advanced "contractors"** establishes a **rigorous, formalized relationship** between the user and the AI — built upon clearly defined and mutually agreed-upon terms, like a **legal service agreement** in the human world.

---

### 13.2 Four Pillars

**1. Formalized Contract**
- A detailed specification that serves as the **single source of truth** for a task.
- Explicitly defines required deliverables, making the outcome **objectively verifiable**.

**2. Dynamic Lifecycle of Negotiation and Feedback**
- The contract is **not a static command but the start of a dialogue**.
- The contractor agent can analyze initial terms and **negotiate**.
- The negotiation phase allows the agent to **flag ambiguities or potential risks** before execution begins — preventing costly failures.

**3. Quality-Focused Iterative Execution**
- Operates on a principle of **self-validation and correction**.
- Internal loop of generating, reviewing, and improving work until specifications are met.
- The contractor **prioritizes correctness and quality**.

**4. Hierarchical Decomposition via Subcontracts**
- For tasks of significant complexity, the primary contractor acts as a **project manager**, breaking the main goal into smaller, manageable sub-tasks.
- This allows the system to tackle **immense, multifaceted projects** in a highly organized and scalable manner.

---

## 14. Task Prioritization Pattern

### 14.1 Core Concept

Task prioritization enables agents to **assess and rank tasks** based on their significance, urgency, dependencies, and established criteria — ensuring agents concentrate efforts on the most critical tasks.

---

### 14.2 Prioritization Elements

| Element | Description |
|---------|-------------|
| **Criteria Definition** | Establishes rules/metrics for task evaluation: urgency, importance, dependencies, resource availability, cost/benefit analysis, user preferences. |
| **Task Evaluation** | Assessing each task against defined criteria using methods ranging from simple rules to complex LLM reasoning. |
| **Scheduling/Selection Logic** | Algorithm that selects the optimal next action or task sequence based on evaluations. |
| **Dynamic Re-prioritization** | Allows the agent to modify priorities as circumstances change (new critical event, approaching deadline). |

---

### 14.3 Use Cases

- **IT/Customer Support:** Prioritizing urgent requests (e.g., system outage reports) over routine matters.
- **Cloud Resource Management:** AI manages and schedules resources by prioritizing allocation to critical applications during peak demand.
- **Autonomous Vehicles:** Continuously prioritize actions to ensure safety and efficiency.
- **Trading Bots:** Prioritize trades by analyzing market conditions, risk tolerance, profit margins, and real-time news.
- **Project Management:** AI agents prioritize tasks on a project board based on deadlines, dependencies, team availability.
- **Cybersecurity:** Agents monitoring network traffic prioritize alerts by assessing threat severity, potential impact, and asset criticality.

---

## 15. Exploration & Discovery Pattern

### 15.1 Core Concept

Exploration and Discovery patterns **enable intelligent agents to actively seek out novel information**, uncover new possibilities, and identify **unknown unknowns** within their operational environment.

- Focus on agents proactively venturing into **unfamiliar territories**, experimenting with new approaches, and generating new knowledge.
- Crucial for agents operating in **open-ended, complex, or rapidly evolving domains**.

---

### 15.2 Use Cases

- **Scientific Research:** An agent designs and runs experiments, analyzes results.
- **Trend Analysis:** Agents scan unstructured data (social media, news, reports) to identify trends.
- **Security Testing:** Agents probe systems or codebases to find security flaws or attack vectors.

---

### 15.3 Real-World Example: Google Co-Scientist

A key advantage of the Co-Scientist system is its ability to **synthesize answers from information fragmented across multiple documents**.

**Phase 1 — Literature Review:** Specialized LLM-driven agents autonomously collect and critically analyze pertinent scholarly literature.

**Phase 2 — Experimentation:** Encompasses collaborative formulation of experimental designs, data preparation, and execution of experiments.

**Phase 3 — Reporting:** System automates generation of comprehensive research reports, synthesizing findings from experimentation with literature review insights and structuring the document according to academic conventions.

**AgentRxiv** is a platform enabling autonomous research agents to **share, access, and collaboratively advance** scientific discoveries.

---

## 16. Multi-Agent System Search (MASS) Framework

### 16.1 Core Concept

The MASS Framework is a **three-stage optimization process** that navigates a search space encompassing optimizable prompts and configurable agent building blocks.

The design of multi-agent systems is critically dependent on:
1. The **quality of prompts** used to program individual agents.
2. The **topology** that dictates their interactions.

---

### 16.2 Three-Stage Optimization

**Stage 1 — Block-level Prompt Optimization**
- Independently optimizes prompts for **each agent module** before integration into the larger system.

**Stage 2 — Workflow Topology Optimization**
- Optimizes the workflow topology by selecting and arranging different agent interactions from a customizable design space.
- Calculates the **"incremental influence"** of each topology (performance gain relative to a baseline agent).
- Uses these scores to guide the search toward **more promising combinations**.

**Stage 3 — Workflow-level Prompt Optimization**
- A second round of prompt optimization for the **entire multi-agent system** after the optimal workflow is identified.
- A **global optimization** that fine-tunes prompts as a single, integrated entity to ensure agent interdependencies are optimized.

> *Key finding: For coding, a structure that combines iterative self-correction with external verification is superior to simpler MAS designs.*

---

### 16.3 Key Principles

1. Optimize individual agents with **high-quality prompts before composing them**.
2. Construct MAS by composing **influential topologies** rather than exploring an unconstrained search space.
3. Model and optimize the **interdependencies between agents** through final, workflow-level joint optimization.

---

## 17. Prompting Techniques

### 17.1 Fundamentals

- Language models **interpret patterns** — multiple interpretations may lead to unintended responses.
- Define the **task, desired output format, and any limitations or requirements**.
- Instructions should be **direct**; prompts should be **simple**.
- **Action verbs** indicate the expected operation. Effective verbs include: *Act, Analyze, Categorize, Classify, Contrast, Compare, Create, Describe, Define, Evaluate, Extract, Find, Generate, Identify, List, Measure, Organize, Parse, Pick, Predict, Prioritize...*

---

### 17.2 Shot-Based Prompting

| Technique | Description |
|-----------|-------------|
| **Zero-Shot** | Language model is provided with an instruction and input data with **no examples** of the desired input-output pair. |
| **One-Shot** | Providing the language model with a **single example** of input and corresponding desired output. Useful when the desired output format or style is specific or less common. |
| **Few-Shot** | Enhances one-shot prompting by supplying **several examples** (typically 3–5) of input-output pairs. Examples should be accurate, representative, and cover potential edge cases. For classification tasks, **mix up the order** of examples from different classes to prevent overfitting. |

---

### 17.3 Prompt Structure & Roles

**Structuring:** Using different sections within the prompt to provide distinct types of information:
- **Instructions** — what to do
- **Context** — background information, user identity, interaction history, environmental state
- **Examples** — input-output demonstrations

**Delimiters:** Triple backticks (` ``` `), XML tags (`<instruction>`, `<context>`), or markers (`---`) to visually and programmatically separate sections.

**Output format:** Specify JSON, XML, CSV, or Markdown tables as needed.

**System Prompting:**
- Sets the **overall context and purpose** for the language model.
- Defines intended behavior for an interaction or session.
- Used to **establish rules, a persona, or overall behavior**.

**Role Prompting:**
- Assigns a specific character, persona, or identity to the language model.
- Instructs the model to adopt the **knowledge, tone, and communication style** associated with that role.

**Persona Pattern:**
- Describes the **user or target audience** for the model's output.
- *Example: "You are explaining quantum physics. The target audience is a high school student with no prior knowledge of the subject. Explain it simply and use analogies they might understand."*

---

### 17.4 Reasoning Prompting Techniques

*(See full breakdown in Section 11.3)*

| Technique | Core Idea |
|-----------|-----------|
| **Chain-of-Thought (CoT)** | Guide model to generate intermediate reasoning steps before the final answer. |
| **Tree of Thoughts (ToT)** | Explore multiple reasoning paths with backtracking. |
| **Self-Consistency** | Generate multiple diverse reasoning paths; select the most consistent answer. |
| **ReAct** | Combine reasoning with tool-based actions and observations. |
| **PAL/Code Interpreter** | Offload computation to deterministic code execution. |
| **Analogical Reasoning** | Frame tasks using analogies to aid understanding. |
| **Society of Mind** | Multiple diverse models collaborate and debate to solve problems. |

---

### 17.5 Automated Prompt Optimization

**Automatic Prompt Engineer (APE):**
- A "meta-model" takes a task description and **generates multiple candidate prompts**.
- Best-performing prompts are selected and potentially refined further.
- Requires: a representative set of **high-quality input-output pairs**, a function that automatically evaluates the LLM's output against the "golden" output.
- Uses an optimizer (e.g., **Bayesian optimizer**) to systematically refine the prompt.
- Strategy: start with a **simple, basic prompt** and iteratively refine based on the model's responses.

---

## 18. GUI/Computer Use Agents

GUI agents (Computer Use agents) interact with graphical interfaces autonomously:

1. The agent **captures a visual representation of the screen** (screenshot).
2. **Analyzes the image** to distinguish between various GUI elements.
3. An **ACI (Agent-Computer Interface) module** acts as a bridge between visual data and the agent's core intelligence — interpreting elements within the context of the task.
4. The agent then **programmatically controls the mouse and keyboard** to execute its plan — clicking, typing, scrolling, and dragging.

---
