# 🤖 Evaluating AI Agents

> A reference guide covering how to evaluate AI agents — including traditional testing, LLM evaluation types, observability, router & skill evaluations, agent trajectory, and evaluation-driven development.

---

## 📑 Table of Contents

1. [Traditional Software Testing](#1-traditional-software-testing)
2. [Testing in the Time of LLMs](#2-testing-in-the-time-of-llms)
3. [Common Types of Evaluations for LLM Systems](#3-common-types-of-evaluations-for-llm-systems)
4. [Some Tools Used to Evaluate Agents](#4-some-tools-used-to-evaluate-agents)
5. [Observing Our Agent](#5-observing-our-agent)
   - [5.1 Tracing Our Agent](#51-tracing-our-agent)
6. [Adding Router and Skill Evaluations](#6-adding-router-and-skill-evaluations)
   - [6.1 Techniques for Running Evals](#61-techniques-for-running-evals)
   - [6.2 Evaluating a Router](#62-evaluating-a-router)
   - [6.3 Evaluating Skills](#63-evaluating-skills)
7. [Agent Trajectory](#7-agent-trajectory)
8. [Evaluation Driven Development](#8-evaluation-driven-development)
9. [Monitoring Agents](#9-monitoring-agents)

---

## 1. Traditional Software Testing

- **Unit testing** — testing individual components of the software application
- **Integration testing** — testing how different components work together

---

## 2. Testing in the Time of LLMs

- **Non-deterministic nature of LLMs** — outputs can vary even with same inputs
- Focus on testing the application's ability to respond to user's specific tasks
- Examine the output quality (relevance, coherence)

---

## 3. Common Types of Evaluations for LLM Systems

- **Hallucinations** — Measures whether the model generates factually incorrect or fabricated information not grounded in reality or provided context.
- **Retrieval relevance** — Evaluates how well the system retrieves useful and contextually relevant documents for a given query (important in RAG systems).
- **Q&A on retrieved data** — Checks whether the model can accurately answer questions using only the retrieved context, without adding unsupported information.
- **Toxicity** — Assesses whether the model output contains harmful, offensive, biased, or unsafe language.
- **Summarization performance** — Measures how well the model produces concise, accurate, and complete summaries of input content.
- **Code writing correctness and readability** — Evaluates whether generated code is functionally correct, efficient, and easy for humans to understand and maintain.

---

## 4. Some Tools Used to Evaluate Agents

- **Trace instrumentation** — Captures step-by-step logs of an agent's actions to show how it reasons, calls tools, and produces outputs internally.
- **Eval runner (LLM-as-judge)** — Automatically evaluates outputs by using another LLM to score quality, correctness, or alignment against defined criteria.
- **Datasets** — Structured collections of inputs and expected outputs used to consistently test, benchmark, and compare agent performance over time.
- **Human feedback** — Incorporates real user or annotator input to validate quality, correct errors, and improve the system based on human judgment.
- **Prompt playground** — An interactive environment to experiment with prompts, tweak inputs, and instantly observe model responses for iteration.

---

## 5. Observing Our Agent

- **Observability** — complete visibility into every layer of an LLM based software system (the application, the prompt and the response)
- **Traces** — A trace records the paths taken by requests as they propagate through multiple steps
- **Spans** — Data captured on individual steps in an LLM app or pipeline

**Why Observability is important:**
- Simplifies debugging while building applications
- Provides a detailed log for each call made in our application
- Helps understand and control unpredictable behavior of LLMs

---

### 5.1 Tracing Our Agent

**Setup Phoenix + Instrumentation:**

```python
import phoenix as px
from phoenix.otel import register
from openinference.instrumentation.llama_index import LlamaIndexInstrumentor

# Start Phoenix (local)
session = px.launch_app()

# Register tracer
tracer_provider = register()

# Instrument LlamaIndex (AUTO tracing)
LlamaIndexInstrumentor().instrument(tracer_provider=tracer_provider)
```

**Setup LLM:**

```python
from llama_index.llms.openai import OpenAI

llm = OpenAI(model="gpt-4o-mini")
```

**Define tools:**

```python
import pandas as pd
import duckdb
from llama_index.core.tools import FunctionTool

DATA_PATH = "data/Store_Sales_Price_Elasticity_Promotions_Data.parquet"

def lookup_sales_data(query: str) -> str:
    df = pd.read_parquet(DATA_PATH)
    duckdb.sql("CREATE TABLE sales AS SELECT * FROM df")
    # simple SQL via LLM
    sql = f"SELECT * FROM sales LIMIT 5"  # keep simple
    result = duckdb.sql(sql).df()
    return result.to_string()

lookup_tool = FunctionTool.from_defaults(fn=lookup_sales_data)

def analyze_sales_data(data: str, question: str) -> str:
    prompt = f"Data:\n{data}\n\nQuestion: {question}"
    return llm.complete(prompt).text

analyze_tool = FunctionTool.from_defaults(fn=analyze_sales_data)
```

**Create Agent:**

```python
from llama_index.core.agent import ReActAgent

agent = ReActAgent.from_tools(
    tools=[lookup_tool, analyze_tool],
    llm=llm,
    verbose=True
)
```

**Run the agent:**

```python
response = agent.chat("Which stores did the best in 2021?")
print(response)
```

**View in Phoenix:**

```python
print(session.url)  # open the link to see traces, tools calls, etc.
```

---

## 6. Adding Router and Skill Evaluations

### 6.1 Techniques for Running Evals

- **Code-based evals** — Running code to compare outputs to expected outputs
  - Matches regex
  - JSON parseable
  - Contains keywords

- **LLM as a Judge** — using a separate LLM to judge the outputs of our application
  - Always use discrete classification labels, never an undefined score

- **Human Annotations** — Use human labelers or user feedback to evaluate your application outputs
  - Using responses from users

---

### 6.2 Evaluating a Router

Routers are usually evaluated in 2 ways:

- **Function calling choice** — did the router choose the right function to call?
- **Parameter extraction** — did the router extract the right function parameters from the question?

**Example of router and skill evaluations:**

```python
# =========================
# 1. IMPORTS
# =========================
import phoenix as px
import pandas as pd
import duckdb
from phoenix.otel import register
from phoenix.trace.dsl import SpanQuery
from phoenix.trace import SpanEvaluations
from phoenix.evals import llm_classify, OpenAIModel
from openinference.instrumentation.llama_index import LlamaIndexInstrumentor
from openinference.instrumentation import suppress_tracing
from llama_index.llms.openai import OpenAI
from llama_index.core.tools import FunctionTool
from llama_index.core.agent import ReActAgent

# =========================
# 2. START PHOENIX + TRACING
# =========================
# Launch Phoenix UI
session = px.launch_app()

# Register tracer
tracer_provider = register(project_name="sql-agent-eval")

# Auto-instrument LlamaIndex (ALL traces go to Phoenix automatically)
LlamaIndexInstrumentor().instrument(tracer_provider=tracer_provider)

# =========================
# 3. LOAD DATASET (DUCKDB)
# =========================
DATA_PATH = "your data directory location"

df_global = pd.read_parquet(DATA_PATH)

# Create DuckDB table
duckdb.sql("CREATE TABLE sales AS SELECT * FROM df_global")

# =========================
# 4. LLM SETUP
# =========================
llm = OpenAI(model="gpt-4o-mini")

# =========================
# 5. TRUE SQL GENERATION TOOL
# =========================
def generate_sql(question: str) -> str:
    """
    Uses LLM to generate SQL dynamically from question + schema
    """
    schema = ", ".join(df_global.columns)
    prompt = f"""
You are a SQL expert.
Generate a DuckDB SQL query.
Rules:
- Only return SQL (no explanation)
- Table name: sales
- Columns: {schema}
Question:
{question}
"""
    sql = llm.complete(prompt).text
    # clean markdown formatting if any
    sql = sql.replace("```sql", "").replace("```", "").strip()
    return sql

# =========================
# 6. SQL EXECUTION TOOL
# =========================
def run_sql(sql: str) -> str:
    """
    Executes generated SQL safely using DuckDB
    """
    try:
        result = duckdb.sql(sql).df()
        return result.to_string()
    except Exception as e:
        return f"ERROR: {str(e)}"

# =========================
# 7. COMBINED TOOL (SQL PIPELINE)
# =========================
def sql_tool(question: str) -> str:
    """
    Full pipeline:
    1. Generate SQL
    2. Execute SQL
    3. Return both SQL + result
    """
    sql = generate_sql(question)
    result = run_sql(sql)
    return f"""
SQL GENERATED:
{sql}

RESULT:
{result}
"""

sql_function_tool = FunctionTool.from_defaults(fn=sql_tool)

# =========================
# 8. CREATE LLM AGENT
# =========================
agent = ReActAgent.from_tools(
    tools=[sql_function_tool],
    llm=llm,
    verbose=True
)

# =========================
# 9. RUN TEST QUESTIONS (GENERATE TRACES)
# =========================
questions = [
    "What was the total revenue across all stores?",
    "Which store had highest sales volume?",
    "What was the most popular product SKU?"
]

for q in questions:
    print("\nQUESTION:", q)
    agent.chat(q)

# =========================
# 10. QUERY PHOENIX TRACES (SQL + LLM OUTPUT)
# =========================
client = px.Client()

sql_df = client.query_spans(
    SpanQuery()
    .where("span_kind == 'LLM'")
    .select(
        question="input.value",
        sql="output.value"
    ),
    project_name="sql-agent-eval"
)

# keep only SQL generation prompts
sql_df = sql_df[sql_df["question"].str.contains("SQL expert", na=False)]

# =========================
# 11. SQL EVALUATION (LLM-AS-A-JUDGE)
# =========================
SQL_EVAL_PROMPT = """
You are evaluating SQL correctness.

Question:
{question}

Generated SQL:
{sql}

Rules:
- Assume table and columns exist
- Evaluate if SQL answers the question correctly

Answer ONLY:
correct or incorrect
"""

with suppress_tracing():
    sql_eval = llm_classify(
        dataframe=sql_df,
        template=SQL_EVAL_PROMPT,
        rails=["correct", "incorrect"],
        model=OpenAIModel(model="gpt-4o"),
        provide_explanation=True
    )

# convert labels to numeric score
sql_eval["score"] = sql_eval["label"].map({
    "correct": 1,
    "incorrect": 0
})

# =========================
# 12. LOG EVALUATION TO PHOENIX
# =========================
client.log_evaluations(
    SpanEvaluations(
        eval_name="SQL Correctness Eval",
        dataframe=sql_eval
    )
)

# =========================
# 13. OPEN PHOENIX DASHBOARD
# =========================
print("Phoenix UI:", session.url)
```

---

### 6.3 Evaluating Skills

Skills can be evaluated using standard LLM to test:

- Relevance
- Hallucination
- Question and answer correctness
- Generated code readability
- Summarization

---

## 7. Agent Trajectory

- **Trajectory** — the path through router steps, tool calls and logic steps that the agent took for a given input.
- To measure how closely our agent follows an optimal path for a given query, we can use a tool called **convergence**.

**How we test for convergence:**

1. Run our agent on a set of similar queries
2. Record the number of steps taken for each query
3. Find the length of the optimal path
4. Calculate the convergence score — what percentage of the time is our agent taking the optimal path, for a given set of inputs?

> A convergence score of 1 means the agent is taking the optimal path 100% of the time.

**Example — Adding trajectory evaluation:**

```python
# =============================
# Production-Ready (Correct Tracing):
# LlamaIndex + Phoenix Convergence Evaluation
# =============================
import warnings
warnings.filterwarnings('ignore')

import uuid
from datetime import datetime
import pandas as pd
import phoenix as px
from phoenix.experiments import run_experiment, evaluate_experiment
from phoenix.experiments.types import Example
from phoenix.experiments.evaluators import create_evaluator
from phoenix.otel import register

# ---------------------------
# STEP 0: Enable Phoenix Tracing
# ---------------------------
tracer_provider = register(project_name="llamaindex-convergence-prod")

# ---------------------------
# STEP 1: Dataset
# ---------------------------
questions = [
    "What was the average quantity sold per transaction?",
    "What is the mean number of items per sale?",
    "Calculate the typical quantity per transaction",
    "What's the mean transaction size in terms of quantity?",
    "On average, how many items were purchased per transaction?",
]

df = pd.DataFrame({"question": questions})

px_client = px.Client()

dataset = px_client.upload_dataset(
    dataframe=df,
    dataset_name=f"convergence-prod-{datetime.now()}",
    input_keys=["question"],
)

# ---------------------------
# STEP 2: LlamaIndex Setup
# ---------------------------
from llama_index.core import VectorStoreIndex, SimpleDirectoryReader
from llama_index.core.agent import ReActAgent
from llama_index.core.tools import QueryEngineTool

# Load data (replace with real data)
documents = SimpleDirectoryReader("./data").load_data()
index = VectorStoreIndex.from_documents(documents)
query_engine = index.as_query_engine()

tool = QueryEngineTool.from_defaults(
    query_engine=query_engine,
    name="data_query_tool",
    description="Answers questions over business data",
)

agent = ReActAgent.from_tools(
    tools=[tool],
    verbose=True,
)

# ---------------------------
# STEP 3: Correct Trace-Based Path Length
# ---------------------------
def get_path_length_for_trace(trace_id: str) -> int:
    """
    Fetch spans for a specific trace_id and count steps.
    """
    spans_df = px.Client().get_spans_dataframe()
    if spans_df.empty:
        return 1

    # Filter only this trace
    trace_spans = spans_df[spans_df["trace_id"] == trace_id]
    if trace_spans.empty:
        return 1

    # Count meaningful steps
    step_mask = trace_spans["span_kind"].isin(["LLM", "TOOL"])
    return max(1, step_mask.sum())

def run_agent_and_track_path(example: Example):
    question = example.input.get("question")

    # Unique trace ID per example
    trace_id = str(uuid.uuid4())

    # Run inside trace
    with tracer_provider.start_as_current_span(
        "agent-run",
        attributes={"example_id": trace_id}
    ) as span:
        # Attach trace_id explicitly
        span.set_attribute("trace_id", trace_id)
        response = agent.chat(question)

    # Compute path length ONLY for this trace
    path_length = get_path_length_for_trace(trace_id)

    return {
        "trace_id": trace_id,
        "path_length": int(path_length),
        "answer": str(response),
    }

# ---------------------------
# STEP 4: Run Experiment
# ---------------------------
experiment = run_experiment(
    dataset,
    run_agent_and_track_path,
    experiment_name="Prod Convergence Eval (Traced)",
    experiment_description="Trace-scoped convergence using Phoenix",
)

results_df = experiment.as_dataframe()
print(results_df[["input", "output"]])

# ---------------------------
# STEP 5: Compute Optimal Path
# ---------------------------
outputs = results_df["output"].tolist()
optimal_path_length = min(
    o["path_length"] for o in outputs if o and "path_length" in o
)
print("Optimal path length:", optimal_path_length)

# ---------------------------
# STEP 6: Evaluator
# ---------------------------
@create_evaluator(name="convergence_score", kind="CODE")
def evaluate_path_length(output):
    if output and output.get("path_length"):
        return optimal_path_length / float(output["path_length"])
    return 0.0

# ---------------------------
# STEP 7: Evaluate
# ---------------------------
experiment = evaluate_experiment(
    experiment,
    evaluators=[evaluate_path_length],
)

final_df = experiment.as_dataframe()
print("\nFinal evaluated results:")
print(final_df[["input", "output", "evaluations"]])
```

---

## 8. Evaluation Driven Development

> Curate dataset → Track Changes as an experiment → Evaluate the experiment → Score

- **Curate dataset** — construct a dataset of test cases
  - 25+ examples of inputs to our agent
  - Attach expected outputs to these test cases

- **Track agent changes** — test proposed changes to our agent
  - Prompt iterations
  - Tool definitions
  - Router logic
  - Skill structure
  - Model changes

- **Evaluate experiment**
  - Code based evals
  - LLM-as-a-Judge evals

**Example of evaluation driven development:**

```python
# =========================
# 1. Imports
# =========================
import pandas as pd
from datetime import datetime
import phoenix as px
from phoenix.evals import OpenAIModel, llm_classify
from phoenix.experiments import run_experiment
from llama_index.core.agent import ReActAgent
from llama_index.core.tools import FunctionTool
from llama_index.llms.openai import OpenAI

# =========================
# 2. Define Tools
# =========================
def lookup_sales_data(query: str) -> str:
    """
    Mock database lookup (replace with real DB logic)
    """
    if "SUM" in query:
        return "13272640"
    elif "AVG" in query:
        return "19.018132"
    return "0"

def generate_visualization(code: str) -> str:
    """
    Mock visualization tool
    """
    return code

tools = [
    FunctionTool.from_defaults(fn=lookup_sales_data),
    FunctionTool.from_defaults(fn=generate_visualization),
]

# =========================
# 3. Build LlamaIndex Agent
# =========================
llm = OpenAI(model="gpt-4o")

agent = ReActAgent.from_tools(
    tools=tools,
    llm=llm,
    verbose=True
)

# =========================
# 4. Define Agent Task
# =========================
def run_agent_task(example):
    question = example.input["question"]
    response = agent.chat(question)
    return {
        "final_output": str(response)
    }

# =========================
# 5. Create Dataset
# =========================
data = [
    {
        "question": "What was the total revenue across all stores?",
        "expected_answer": "13272640"
    },
    {
        "question": "What was the average transaction value?",
        "expected_answer": "19.018132"
    },
]

df = pd.DataFrame(data)
px_client = px.Client()

dataset = px_client.upload_dataset(
    dataframe=df,
    dataset_name=f"llamaindex-eval-{datetime.now()}",
    input_keys=["question"],
    output_keys=["expected_answer"]
)

# =========================
# 6. Setup Evaluator Model
# =========================
eval_model = OpenAIModel(model="gpt-4o")

# =========================
# 7. Evaluators
# =========================
# --- Correctness ---
def correctness_eval(output, expected):
    df = pd.DataFrame({
        "query": [expected["expected_answer"]],
        "response": [output["final_output"]]
    })
    result = llm_classify(
        data=df,
        template="""
Does the response contain the correct answer?
Expected: {query}
Response: {response}
Answer "correct" or "incorrect".
""",
        rails=["correct", "incorrect"],
        model=eval_model
    )
    return result["label"] == "correct"

# --- Clarity ---
def clarity_eval(output, input):
    df = pd.DataFrame({
        "query": [input["question"]],
        "response": [output["final_output"]]
    })
    result = llm_classify(
        data=df,
        template="""
Is the response clear and easy to understand?
Query: {query}
Response: {response}
Answer "clear" or "unclear".
""",
        rails=["clear", "unclear"],
        model=eval_model
    )
    return result["label"] == "clear"

# =========================
# 8. Run Experiment
# =========================
experiment = run_experiment(
    dataset=dataset,
    task=run_agent_task,
    evaluators=[correctness_eval, clarity_eval],
    experiment_name="llamaindex-agent-eval",
)

print("Experiment complete. Check Phoenix UI for results.")
```

---

## 9. Monitoring Agents

- **Efficiency of our system** — ensuring our system performance doesn't degrade over time
- **Dependencies on external services** — ensure that you are monitoring external LLM calls
- **Continuous improvement** — collect user feedback to understand end to end agent performance
- Curate datasets for continuous experiments
