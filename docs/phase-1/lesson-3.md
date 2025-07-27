---
title: "1-3 Tool Wrappers & LangChain"
---

# Lesson 1-3: Tool Wrappers & LangChain

**Learning Objectives**  
By the end this lesson, you’ll be able to:

- Explain the concept of “tools” in agent architectures and why wrappers are essential
- Distinguish between **chains** (static pipelines) and **agents** (dynamic tool dispatch) in LangChain
- Define and implement custom tools by subclassing or using decorators, including metadata and argument schemas
- Compose a simple multi-tool reasoning agent using LangChain’s abstractions

---

## 1. Introduction to Tools in Agents

**What Is a Tool?**  
A _tool_ is any callable function, API, or service the agent can invoke to perform a discrete action (e.g., search the web, do arithmetic, query a database). Wrappers standardize tools with:

- A **name** for planner reference
- A **description** for prompting
- A **typed interface** (arguments schema)
- Built-in **error handling**

**Why Wrappers?**

- Uniform interface for the executor to dispatch calls
- Metadata ensures the planner (LLM) knows which tools exist and how to call them
- Simplifies logging, retries, and fallback logic

---

## 2. Chains vs. Agents in LangChain

| Concept     | Chains                                         | Agents                                                 |
| ----------- | ---------------------------------------------- | ------------------------------------------------------ |
| Definition  | Static sequence of LLM calls + transforms      | Dynamic planner-driven loops that select and run tools |
| Use Case    | Simple pipelines (e.g., summarize → translate) | Complex workflows requiring conditional tool selection |
| Key Classes | `LLMChain`, `SequentialChain`                  | `AgentExecutor`, `initialize_agent`, `AgentType`       |

**Chain Example:**
```

from langchain import LLMChain, PromptTemplate

template = PromptTemplate(
input_variables=["text"],
template="Summarize the following:\n\n{text}"
)
chain = LLMChain(llm=llm, prompt=template)
output = chain.run(text="Long article content here...")

```

---

## 3. Defining Custom Tools

### 3.1 Subclassing `BaseTool`

```

from langchain.tools import BaseTool
from pydantic import BaseModel

class SearchParams(BaseModel):
query: str

class WebSearchTool(BaseTool):
name = "web_search"
description = "Search the web for information given a query."
args_schema = SearchParams

    def _run(self, query: str) -> str:
        # Replace with real API call
        results = search_api(query)
        return results

    async def _arun(self, query: str) -> str:
        # Async variant if needed
        results = await async_search_api(query)
        return results

```

### 3.2 Using the `@tool` Decorator

```

from langchain.tools import tool

@tool(name="calculator", description="Performs arithmetic calculations.")
def calculator_fn(expression: str) -> str: # Simple eval for demonstration; use safe parser in production
try:
return str(eval(expression))
except Exception as e:
return f"Error: {e}"

```

---

## 4. Composing Chains and Agents

### 4.1 Single-Step Chain

```

# A simple chain that wraps a prompt template

from langchain import LLMChain, PromptTemplate

prompt = PromptTemplate(template="Translate to French: {sentence}", input_variables=["sentence"])
translate_chain = LLMChain(llm=llm, prompt=prompt)
print(translate_chain.run(sentence="Hello, world!"))

```

### 4.2 Multi-Tool Agent

```

from langchain.agents import initialize_agent, AgentType

tools = [WebSearchTool(), calculator_fn, SummarizerTool()] # tool instances or functions

agent = initialize_agent(
tools,
llm,
agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
verbose=True
)

# Run a compound task

task = "Find the population of Paris, calculate its square root, and summarize the result."
response = agent.run(task)
print(response)

```

- **AgentType.ZERO_SHOT_REACT_DESCRIPTION** uses tool descriptions to plan and act in a ReAct loop.
- The agent prints each action and observation when `verbose=True`.

---

## 5. Multi-Tool Reasoning Agent in Practice

1. **Planner Stage:** LLM reviews `tools` metadata and returns an action JSON:
```

{"action": "web_search", "params": {"query": "population of Paris"}}

```
2. **Executor Stage:** Dispatcher calls `tools[action].run(**params)` or `_arun` for async.
3. **Observation:** Agent logs the result, passes to planner for next step.
4. **Loop:** Continues until the planner emits a `"finish"` action or meets stopping criteria.

**Error Handling:**
- Tools should catch exceptions and return error messages.
- The planner can detect errors and choose alternative tools or abort gracefully.

---

## 6. Mini-Project: Build a Three-Tool Agent

**Goal:** Create a LangChain agent with three tools—`web_search`, `calculator`, and `summarizer`—and run a compound query.

**Steps:**

1. **Define Tools**
- Subclass `BaseTool` or use `@tool` for:
  - `web_search(query: str) → str`
  - `calculator(expression: str) → str`
  - `summarizer(text: str) → str`

2. **Initialize Agent**
```

from langchain.agents import initialize_agent, AgentType

tools = [WebSearchTool(), calculator_fn, SummarizerTool()]
agent = initialize_agent(tools, llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION)

```

3. **Run Task**
```

result = agent.run("Get the current time in New York, add 5 hours, and summarize the plan.")
print(result)

```

4. **Log Invocations**
- Enable `verbose=True` to see each tool call and output.
- Optionally write logs to a file for inspection.

---

## 7. Self-Check Questions

1. What distinguishes a **chain** from an **agent** in LangChain?
2. Why does a custom tool need a `name` and `description`?
3. How does the agent planner decide which tool to call next?
4. In your multi-tool agent, how would you implement a retry strategy for flaky API calls?

---

**Next Up:**
Lesson 1-4 will cover **Memory & Basic RAG**, where you’ll integrate vector stores for long-term knowledge retrieval and context management.
```
