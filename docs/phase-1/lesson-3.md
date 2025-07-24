```markdown
---
title: "0-3 Thinking Like an Agent"
---

# Lesson 0-3: Thinking Like an Agent

**Learning Objectives**  
By the end of this lesson, you will be able to:
- Decompose complex goals into agent-compatible subtasks using agentic reasoning frameworks.
- Apply behavior trees and chain-of-thought methods to structure multi-step processes.
- Recognize common failure points and cost issues in autonomous workflows.
- Sketch reflective loops and feedback cycles to increase agent reliability and robustness.

---

## 1. Introduction: Why Agentic Thinking Matters

Successful agents don’t just execute pre-defined scripts; they plan ahead, adapt, and reflect at every step. This mindset goes beyond classic procedural code, asking:  
- What is the end goal?  
- What are the minimal, modular steps needed?  
- What tools or reasoning methods apply for each step?  
- How and when should the agent check its own progress or results?

**Key Principle:** Agents break large goals into “bite-sized” subtasks, select the right tool for each, act, observe, and adjust based on results.

---

## 2. Problem Decomposition: Goal → Subtasks

To act autonomously, agents must transform broad goals into a sequence (or tree) of subtasks.  
**Example:**  
*Goal:* “Book a meeting with Alice and Bob next week.”

**Decomposition:**
- Search both users’ calendars for availability  
- Propose potential times  
- Draft invite and confirmation message  
- Send invite and await acceptance

**Diagram:**  
```
[image: goal-subtask-tree]
A tree with root "Book meeting" branching to "Search calendars", "Propose times", "Draft invite", "Send/await acceptance"
```
*(Insert a simple tree diagram here for visual context.)*

> **Did you know?** The process of breaking goals into ordered subtasks is called “task decomposition” and is a core building block of planning agents.

---

## 3. Reasoning Frameworks: Behavior Trees & Chain-of-Thought

### 3.1 Behavior Trees

Used widely in robotics and games, behavior trees structure agent logic into nodes:
- **Selector Nodes:** Choose which branch to attempt, based on conditions.
- **Sequence Nodes:** Execute children in order; fail if any step fails.
- **Action Leaves:** Invoke a tool or LLM call.

**Mini Diagram:**  
```
Book Meeting (Selector)
 ├─ Check calendars (Action)
 ├─ Find slots (Action)
 ├─ Propose time (Action)
 ├─ Draft invite (Action)
 └─ Send invite (Action)
```

### 3.2 Chain-of-Thought Prompts

Modern LLM-powered agents benefit from *thinking aloud* before acting:
- “First, I’ll check Alice’s schedule. Next, I’ll check Bob’s. Then, I’ll look for overlapping free slots. Finally, I’ll send an invite.”
- This step-wise reasoning helps the agent avoid skipping or muddling steps, providing transparency and traceability.

> **Callout:** Chain-of-thought (CoT) reduces hallucinations and improves reliability in complex, multi-step tasks.

---

## 4. Reflective Loops and Self-Correction

Agents inevitably encounter uncertainty—missing info, unexpected outputs, or tool errors. Reflection is the art of checking results, diagnosing issues, and deciding what to do next.

**Signs your agent needs reflection:**  
- Stuck in infinite loop (keeps retrying same step)
- Repeats failed actions (“Cannot find calendar. Cannot find calendar…”)
- Outputs unclear or partial results

**Mitigation:**
- **Observation logging**: Write each action/observation to memory for inspection.
- **Conditional checks**: If output is blank, error, or duplicate, trigger different plan or escalate.
- **Feedback cycle**: Use results of one step to alter subsequent steps (“if time slot not found, search next week”).

---

## 5. Worked Example: Agent for Data Pipeline

**Goal:** “Ingest a CSV file, clean the data, compute summary statistics, and email results.”

**Agentic Decomposition:**
- **Subtasks:**
    1. Load CSV from given path.
    2. Clean data (remove nulls/outliers).
    3. Compute summary statistics (mean, median).
    4. Generate summary report.
    5. Draft and send email with results attached.

**Behavior Tree:**  
```
Ingest & Report Pipeline (Sequence)
 ├─ Load CSV (Action)
 ├─ Clean Data (Action)
 ├─ Compute Stats (Action)
 ├─ Generate Report (Action)
 └─ Send Email (Action)
```
**Chain-of-Thought Example:**
- “I will start by loading the CSV file… The file loaded. I see there are missing values. I’ll remove rows with missing values… Now compute stats—mean, median, std. Compose a brief summary. Attach to an email and send to the recipient.”

> **Case Study:** In production, such an agent failed when the CSV was missing columns; self-check logic added: “If expected columns not found, send error notification.”

---

## 6. Mini-Project Prompt

**Task:** Design a high-level agent flow for “Onboard a new employee” (inputs: name, start date, department).

- **Step 1:** Sketch possible subtasks (e.g., create IT account, send onboarding document, schedule intro meeting, send welcome email).
- **Step 2:** Draw a behavior tree or write a chain-of-thought outline explaining the agent’s reasoning.

*Optional: Write a Python pseudocode list representing these subtasks and how the agent would iterate through them.*

---

## 7. Self-Check Questions

1. Why should agents “think aloud” using chain-of-thought before acting?
2. Name one benefit and one risk of automatically decomposing high-level goals.
3. How does a behavior tree help prevent infinite loops or critical step omissions?
4. What’s the first thing your onboarding agent should check before proceeding?

---

**Coming Up:**  
In the next phase you’ll learn to instantiate planners, implement behavior trees and chain-of-thought prompting, and embed basic logging for robust agent development.
