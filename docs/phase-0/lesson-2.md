---
title: "0-2 Core Concepts"
---

# Lesson 0-2: Core Concepts

**Learning Objectives**  
By the end of this lesson, you will be able:

- Define the **Planner → Executor → Retriever** loop and explain each component’s role.
- Describe how **memory**, **tools**, and **policies** fit into an agent’s architecture.
- Distinguish between **reactive**, **goal-driven**, and **multi-agent** systems.
- Map a real-world task to an agent’s internal components for end-to-end automation.

---

## 1. Agent Lifecycle & Taxonomy

### 1.1 Lifecycle Stages

1. **Initialization**
   - Load tool registry, memory backend, and policy constraints.
2. **Planning**
   - Decompose the user goal into ordered subtasks via the Planner.
3. **Execution**
   - The Executor invokes tools or LLM calls for each subtask.
4. **Retrieval**
   - The Retriever fetches external knowledge (vector DB, APIs) as needed.
5. **Observation**
   - Capture tool outputs and update memory or state.
6. **Termination**
   - End when the goal is achieved or a timeout/failure occurs.

### 1.2 Agent Types

- **Reactive Agents**
  - Single-step: direct input → single tool call → output.
  - Example: language translation.
- **Goal-Driven Agents**
  - Multi-step: use a Planner–Executor–Retriever loop to fulfill complex goals.
  - Example: “Summarize this report and draft an email.”
- **Multi-Agent Systems**
  - Ensembles of specialized agents (e.g., planner, workers, evaluator) coordinated by an orchestrator.

---

## 2. Core Components

### 2.1 Tool Registry

Developers register available tools with names and descriptions. The Planner sees these registrations and chooses among them.

```

# Example tool registration

tools = {
"fetch_pricing_docs": PricingScraperTool(),
"summarize_text": SummarizerTool(),
"draft_email": EmailTemplateTool()
}

```

### 2.2 Planner

- **Role:** Chooses which tool to run next and with what parameters.
- **Mechanism:** Provides the LLM a prompt containing the goal and tool descriptions; the LLM returns a structured action.

```

# Simplified planner prompt (pseudo-code)

prompt = f"""
Goal: {user_goal}
Tools:

- fetch_pricing_docs: Scrapes competitor pricing.
- summarize_text: Summarizes text.
- draft_email: Generates a professional email.
  Decide next action in JSON: {{ "action": tool_name, "params": {{...}} }}
  """
  planner_output = llm.generate(prompt)

```

### 2.3 Executor

- **Role:** Maps planner’s chosen action to actual function calls.
- **Mechanism:** A dispatch that looks up the tool in `tools` and invokes its `run(**params)` method.

```

action = planner_output["action"]
params = planner_output["params"]
result = tools[action].run(\*\*params)

```

### 2.4 Retriever

- **Role:** Provides up-to-date or domain-specific context by querying a vector store or API.
- **Mechanism:** Converts a retrieval request into a similarity search or API query.

```

# Example retrieval

docs = vector_store.search(query="CompanyX pricing model")
context = docs[:3] # top 3 results

```

### 2.5 Memory

- **Short-Term Memory:** In-process list capturing recent observations.
- **Long-Term Memory:** Persistent vector database storing embeddings of past interactions.

### 2.6 Policy & Safety

- **Guardrails:** Hard constraints (tool whitelist) and soft constraints (style guidelines).
- **Human-in-Loop:** Fallback when high-risk actions are proposed.

---

## 3. Augmented Example Walkthrough

**Task:** “Research competitor X’s pricing model, summarize key points, and draft a comparison email.”

1. **Initialization**

   - Tools registered (see above).
   - Memory and policy modules initialized.

2. **Planning (Step 1)**
   - Planner receives goal and tool list.
   - LLM returns:

```

{ "action": "fetch_pricing_docs", "params": { "competitor": "CompanyX" } }

```

3. **Execution (Step 1)**

- Executor dispatches to `tools["fetch_pricing_docs"].run(competitor="CompanyX")`.
- Result: raw pricing data.

4. **Observation & Memory**

- Store `{ "step": 1, "tool": "fetch_pricing_docs", "output": "" }` in short-term memory.

5. **Retrieval (Implicit)**

- Retriever may run a similarity search if the raw data is too large for direct processing.

6. **Planning (Step 2)**

- Planner now sees updated memory and returns:

```

{ "action": "summarize_text", "params": { "text": "" } }

```

7. **Execution (Step 2)**

- Executor calls `tools["summarize_text"].run(text="")`.
- Output: concise summary.

8. **Observation & Memory**

- Append summary to memory.

9. **Planning (Step 3)**

- Planner issues:

```

{ "action": "draft_email", "params": { "summary": "", "tone": "professional" } }

```

10. **Execution (Step 3)**

- Executor calls `tools["draft_email"].run(summary="", tone="professional")`.
- Final email draft produced.

11. **Termination**

- Planner recognizes “done” or memory exceeds thresholds and stops loop.

> 🔧 **Note:** We will implement the `ToolRegistry`, detailed planner prompts, executor dispatch logic, and Retriever module in Phases 1–3.

---

## 4. Mini-Project Prompt

**Skeleton Agent:**

1. Define a `tools` dict mapping names to stub functions.
2. Write `plan(goal)` that returns `{"action": "fetch_pricing_docs", "params": {"competitor": "X"}}`.
3. Write `execute(action_dict)` that prints `"Executed {action}"`.
4. Append each step to a `memory` list.
5. Loop until action `"done"` is returned.

---

## 5. Self-Check Questions

1. How does the Planner use tool descriptions to choose actions?
2. What distinguishes the Executor from the Retriever?
3. Why is it important to record observations in memory at each step?
4. Give an example of a hard guardrail and a soft constraint in agent design.

---

## Navigation

!!! success "Phase Complete!"
**[Phase 1: Tools, Prompts & Memory →](../phase-1/)**

    Ready to start building? Learn Python integrations, prompt engineering, and memory systems.
