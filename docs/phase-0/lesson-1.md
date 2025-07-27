---
title: "0-1 Why AI Agents"
---

# Lesson 0-1: Why AI Agents?

**Learning Objectives**  
By the end of this lesson, you will be able to:

- Trace the evolution from simple rule-based automation to modern agentic AI.
- Differentiate between one-off automation and true agentic autonomy.
- Explain how AI agents augment human engineers rather than replace them.
- Articulate why “agentic” skills are future-proof in a fast-changing AI landscape.

---

## 1. Evolution of Automation to AI Agents

### 1.1 Rule-Based Automation

!!! note "Early Automation"
Early automation systems executed fixed, pre-defined scripts.

- **Cron Jobs & Macros** – schedule tasks or record/replay keystrokes.
- **RPA (Robotic Process Automation)** – imitates user actions but cannot adapt if UI changes.  
  **Limitation:** brittle; any unexpected input or change breaks the flow.

> **Callout [Did you know?]:** The first commercial RPA tools appeared in the early 2000s, yet still rely on hard-coded rules that need manual upkeep.

### 1.2 Expert Systems & Heuristic Tools

!!! note "1980s–90s Era"
Expert systems encoded domain knowledge into if-then rules.

- **Chess Engines** – use heuristic evaluation functions to guide searches.
- **Medical Diagnosis Systems** – apply symptom‐to‐disease mappings.  
  **Limitation:** knowledge bases must be manually curated; lack learning.

### 1.3 Large Language Models & Reactive “Smart” Tools

!!! success "2020+ Breakthrough"
With the advent of LLMs (e.g., GPT-3 in 2020), systems could generalize language tasks.

- **Zero-Shot Prompting** – ask the model directly without examples.
- **Few-Shot Prompting** – provide a handful of input-output pairs in context.
- **Retrieval-Augmented Generation (RAG)** – combine LLM with a search index for up-to-date info.  
  **Reactive Nature:** still “you ask, it answers”—no planning or multi-step reasoning.

### 1.4 Agentic Autonomy

**Definition:** AI agents plan, execute, observe, and adapt over multiple steps using tools and memory.  
**Core Loop (ReAct + Planner–Executor–Retriever):**
[Insert Diagram: “Agent Loop” showing Planner → Executor → Retriever → Observation → Planner]

1. **Planner** decides sub-tasks toward a goal.
2. **Executor** invokes tools or LLM calls.
3. **Retriever** fetches external knowledge (vector DB, APIs).
4. **Observation** records results; loop repeats until goal met.

**Key Characteristics:**

- Dynamic task decomposition
- Multi-tool orchestration
- Contextual memory & retrieval
- Conditional branching via chain-of-thought

## 2. Automation vs. Autonomy

| Aspect           | Simple Automation          | AI Agent Autonomy                               |
| ---------------- | -------------------------- | ----------------------------------------------- |
| Task Definition  | Pre-scripted steps         | Goal-driven planning & task decomposition       |
| Adaptability     | Fails on unexpected inputs | Uses retrieval & reasoning to adapt             |
| Decision Process | No reasoning; fixed rules  | Chain-of-Thought + conditional branching        |
| Tool Integration | Single, one-off API calls  | Multi-tool pipelines with stateful interactions |
| Human Role       | Operator or watcher        | Supervisor of goals; handles exceptions only    |

## 3. Career Resilience: Agents Augment, Not Replace

1. **Augmentation Over Replacement**  
   Agents automate repetitive, multi-step tasks. Engineers shift to high-value work: strategy, exception handling, system design.

2. **Evolving Skillsets**  
   From writing scripts to designing agent loops, memory systems, RAG pipelines, evaluation dashboards, and guardrails.

3. **High-Value Roles**

   - **Agent Engineer** builds and debugs agents
   - **Orchestrator/Architect** designs large-scale multi-agent systems
   - **AI Ops Lead** sets up observability, cost controls, and incident processes

4. **Future Trends**  
   As LLMs become more capable, value moves to:
   - Robustness (hallucination prevention)
   - Safety & compliance (guardrails)
   - Adaptation (continuous learning loops)

## 4. Mini-Project: Hand-Coded ReAct Loop

**Objective:** Write a Python CLI agent that solves arithmetic queries via a public calculator API.

1. **Setup:**

```bash
pip install requests
touch react_agent.py
```

2. **Agent Structure:**

```python
import requests

def calculate(expr):
    resp = requests.get("http://api.mathjs.org/v4/", params={"expr": expr})
    return resp.text

def main():
    question = input("Enter arithmetic question: ")
    # Simple parse: assume format “A op B”
    parts = question.split()
    result = calculate(parts[0] + parts[1] + parts[2])
    print("Answer:", result)

if __name__ == "__main__":
    main()
```

3. **Enhancements (Optional):**

- Add **timeout** and **retry** logic around `requests.get`.
- Log planner decisions and executor observations to console.
- Extend to handle multi-operation expressions by chaining calls.

## 5. Self-Check Questions

1. What is the fundamental difference between a rule-based RPA bot and an AI agent?
2. Why do LLM-only systems still fail at multi-step workflows?
3. In the agent loop diagram, what role does the Retriever play?

---

## Navigation

!!! tip "Next Lesson"
**[0-2 Core Concepts →](lesson-2.md)**

    Deep dive into agent architecture patterns, autonomy vs automation, and environment interaction models.
