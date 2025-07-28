---
title: "1-2 Prompt Engineering"
---

# 🎯 Lesson 1-2: Prompt Engineering for LLM Agents

!!! note "Learning Objectives"
    By the end of this lesson, you'll be able to:

- Explain key prompt types (zero-shot, few-shot, chain-of-thought) and their impact on agent outputs
- Design and reuse prompt templates for agent tasks
- Apply prompt chaining, tool descriptions, and advanced prompt structures
- Recognize prompt failure modes and iterate toward robust, reliable prompts

---

## 🧠 1. Introduction: Why Prompt Engineering?

_Prompts_ are the primary means by which we program LLM-based agents—defining their instructions, personality, and tool use without altering model weights.  

_Effective prompt engineering_ transforms the same underlying model into everything from a helpful Q&A bot to a multi-step agent that reasons, decomposes, and safely executes tool calls.

!!! tip "Key Insight"
    Prompt engineering is not just "hackery"—it's a systematic process of designing, evaluating, and iterating language instructions.

---

## 📝 2. Core Prompt Types

### a. Zero-Shot Prompts

- The simplest prompt: only instructions and/or context; no examples.

!!! example "Zero-Shot Example"
  ```
  Summarize the following passage in one sentence:
  {passage}
  ```

### b. Few-Shot Prompts

- Provide 1–3 input/output pairs so the model sees the expected format.
- Helps LLMs learn the desired pattern, style, or reasoning process.

!!! example "Few-Shot Example"
  ```
  Q: What is the capital of France?
  A: Paris

  Q: Who wrote '1984'?
  A: George Orwell

  Q: {user_question}
  A:
  ```

### c. Chain-of-Thought (CoT) Prompts

- Model "thinks aloud," explaining steps before the final answer—key for multi-step or complex reasoning.

!!! example "Chain-of-Thought Example"
  ```
  Question: David has 5 apples. He gives two to Sarah. How many apples remain?
  Let's think step by step:
  1. David has 5 apples.
  2. He gives away 2.
  3. 5 - 2 = 3 apples remain.
  Answer: 3
  ```

!!! info "Prompt Type Comparison"
| Prompt Type | Example Question | Prompt Structure | Characteristic Output |
    |-------------|------------------|------------------|----------------------|
    | Zero-Shot | "Summarize the paragraph in one sentence: {text}" | Instruction only, no examples | Direct, concise summary without model "thinking aloud." |
    | Few-Shot | "Q: What is the capital of France? A: Paris\nQ: Who wrote '1984'? A: George Orwell\nQ: {question}\nA:" | Includes 1–3 example Q&A pairs before the user's question | More accurate format and style consistency, fewer errors. |
    | Chain-of-Thought (CoT) | "Solve: 23 × 47 + 19. Let's think step by step:" | Instruction plus guidance to articulate intermediate reasoning steps | Detailed, stepwise reasoning leading to the correct final answer. |

---

## 🔧 3. Prompt Templates and Reuse

Good agents use **prompt templates**—reusable parameterized strings—to reliably generate effective instructions for each task or tool use.

- Templates use placeholders (e.g., `{context}`, `{question}`, `{tool_list}`), filled programmatically.
- Can be defined in Python (using f-strings/Jinja) or in config files.

!!! example "Template Example"
    ```python
qa_template = (
  "Context: {context}\n"
  "Question: {question}\n"
  "Answer:"
)
def make_qa_prompt(context, question):
    return qa_template.format(context=context, question=question)
```

!!! tip "Template Best Practices"
- **Version control:** Store templates in code or as separate files for audit, testing, and updates.
- **Flexibility:** Use the same template across tasks, providing reliability and simplification as agent complexity grows.

---

## 🚀 4. Advanced Prompting Patterns for Agents

### a. Embedding Tool Descriptions

- List available tools and their natural-language descriptions in the prompt so LLM/agent can select and explain tool use.

!!! example "Tool Description Example"
  ```
  Tools:
  - Search: Search the web for information.
  - Calculator: Do arithmetic calculations.

  Task: What's the square root of the population of Paris?
  ```

  The agent can now reason about which tool to use given the descriptions.

### b. Decision Branching and Conditional Prompts

- Craft prompts that present choices:

!!! example "Conditional Prompt Example"
  ```
  If the question is about math, use the Calculator tool.
  Otherwise, use Search.
  ```

!!! tip "Use Case"
    Useful for complex, multi-tool agents.

### c. Prompt Chaining

- The output of one prompt flows into the next step (e.g., extract entities → summarize → follow up)
- Enables agents to create pipelines: e.g., _retrieve context → answer question → produce actionable summary_

!!! tip "Best Practice"
    Use modular, standalone prompt templates for each agent step, and chain results with clear input/output contracts.

---

## 🔍 5. Evaluating and Iterating on Prompts

!!! warning "Signs of poor prompts"
    - Hallucinations (invention of details)
    - Vague or off-topic answers
    - Inconsistent output format

!!! success "Improvement cycle"
    Try → test output → tweak instructions/examples/placeholders → rerun

!!! info "Metrics to track"
    Accuracy, factuality, relevance, conciseness; for agents, also reliability and tool selection correctness

!!! tip "Documentation Tip"
    Document effective/ineffective prompt variants. Keep a prompt repository to avoid repeating past mistakes.

!!! info "Prompt Debugging Table"
- Try zero-shot → if output is unreliable, add few-shot examples.
- Still unreliable? Add explicit stepwise (CoT) instructions, clarify output format in the prompt.

---

## 💻 6. Mini-Project: Multi-Style Q&A Prompt

!!! success "Multi-Style Q&A Challenge"
**Task:**

- Write a prompt template to answer user questions about a document (simulate RAG).
- Implement three versions:
  1. Zero-shot (just the question)
  2. Few-shot (add at least two Q&A pairs as demonstration)
  3. Chain-of-thought (force the agent to explain, step by step, before the answer)
- Experiment using a small LLM endpoint (e.g., OpenAI, Google Gemini, or a local model).
- Document which prompt gave the most accurate, on-topic answer and why.

**Bonus:**
- Build a `make_prompt` Python function to generate each prompt given context and question.

---

## ❓ 7. Self-Check Questions

!!! question "Knowledge Check"
1. When should you prefer few-shot over zero-shot prompts for an agent task?
2. How does chain-of-thought prompting reduce hallucination and improve reliability?
3. How would you organize and version-control multiple prompt templates for a production agent?
4. Give a practical example where chaining prompts is required to reliably solve an agent use-case.

---

## 🧭 Navigation

!!! success "Next Up"
    **[Lesson 1-3: Tool Wrappers & LangChain →](lesson-3.md)**

In the next lesson, you'll integrate prompt engineering with Python code—wrapping prompts as callable tools and constructing agents able to reason, select, and chain tools using frameworks like LangChain.
