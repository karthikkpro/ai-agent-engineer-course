```markdown
---
title: "2-B Prompt & Tool Chaining"
---

# Lesson 2-B: Prompt & Tool Chaining

**Learning Objectives**  
By the end of this lesson you will be able to:  
- Chain multiple prompts and tools into conditional, multi-step workflows  
- Design branching logic so the agent dynamically selects different sub-chains based on intermediate results  
- Abstract provider details (LLM or vector store) to enable seamless swapping without code changes  
- Construct prompts at runtime that adapt to context, tool output, and policy constraints  
- Implement error handling, graceful degradation, and compensating actions within chained pipelines  

---

## 1. Introduction: Why Prompt & Tool Chaining?

Monolithic prompts grow brittle as task complexity increases. Chaining breaks a complex workflow into modular steps—each a prompt or tool call—improving maintainability, reusability, and clarity. Conditional branching enables the agent to adapt at runtime, executing only relevant sub-chains.

---

## 2. Branching Logic for Agents

### 2.1 Conditional Chains  
Agents inspect intermediate outputs and choose the next sub-chain:

```
if sentiment_score < 0.3:
    run_apology_chain()
elif sentiment_score < 0.7:
    run_neutral_reply_chain()
else:
    run_thank_you_chain()
```

- **Use Case:** Respond differently to positive vs. negative customer feedback.

### 2.2 Dynamic Routing  
Implement a dispatcher that reads an “intent” or numeric score and invokes the appropriate chain:

```
chain_map = {
    "negative": apology_chain,
    "neutral": neutral_chain,
    "positive": thank_you_chain
}
intent = classify_sentiment(review_text)
chain_map[intent].run(review_text)
```

- **Benefit:** New branches can be added by extending `chain_map`, without altering core loop logic.

---

## 3. Multi-Step Prompt Chaining

### 3.1 Sequential Prompts  
Feed the output of one LLM call into the next:

1. **Extract entities**  
2. **Summarize findings**  
3. **Draft action items**

```
entities = entity_chain.run(text=report)
summary = summary_chain.run(entities=entities)
actions = action_chain.run(summary=summary)
```

### 3.2 Nested Prompts  
Embed a sub-chain’s final output into a higher-level template:

```
Prompt:
  "Based on this table of entities and summary:
   {{ summary }}
   Generate three next-step recommendations."
```

- Ensures context continuity and reduces prompt length.

---

## 4. Tool Chain Composition with LangChain

### 4.1 `SequentialChain` vs. `LLMChain`  
- **LLMChain:** Single prompt → LLM call  
- **SequentialChain:** A list of `LLMChain` steps executed in sequence, passing outputs along  

```
from langchain.chains import SequentialChain

workflow = SequentialChain(
    chains=[entity_chain, summary_chain, action_chain],
    input_variables=["report_text"],
    output_variables=["actions"]
)
workflow.run(report_text=doc)
```

### 4.2 Custom Chains with `ChainOfThoughtChain`  
- Integrates reasoning steps and tool calls in one chain, preserving CoT context  

---

## 5. Provider Abstraction & Pluggability

### 5.1 Abstracting LLM Calls  
Wrap model invocation behind a uniform interface:

```
class ModelBridge:
    def __init__(self, client):
        self.client = client
    def generate(self, prompt):
        return self.client.generate(prompt)
```

Switch clients by injecting different `client` implementations (OpenAI, Anthropic, Hugging Face).

### 5.2 Abstracting Retrieval Chains  
Define a `RetrieverInterface` so you can swap FAISS, Chroma, or an API-based retriever:

```
class RetrieverInterface(ABC):
    @abstractmethod
    def retrieve(self, query):
        pass
```

Implement and inject concrete retrievers without changing downstream logic.

---

## 6. Error Handling in Chained Workflows

### 6.1 Graceful Degradation  
If a tool or chain fails, return a safe default and continue:

```
try:
    summary = summary_chain.run(text)
except Exception:
    summary = "Summary unavailable at this time."
```

### 6.2 Compensating Actions  
On critical failure, trigger a corrective sub-chain (e.g., notify human or rollback):

```
if not valid_actions(actions):
    rollback_chain.run(context)
    notify_admin_chain.run(context)
```

---

## 7. Mini-Project: Customer Feedback Workflow

**Task:** Build a chained workflow to process and respond to customer reviews:

1. **Sentiment Analysis Chain:** Classify review as “negative,” “neutral,” or “positive.”  
2. **Branching Chains:**  
   - **Negative →** apology_chain (draft apology email)  
   - **Neutral →** info_chain (provide more information)  
   - **Positive →** thank_you_chain (draft thank-you email)  
3. **Follow-Up Survey Chain:** Always send a survey prompt via a simulated API.  
4. **Logging:** Record each chain invocation, inputs, and outputs to `feedback_workflow.log`.  

Use LangChain’s `SequentialChain` and custom conditionals. Demonstrate graceful degradation and compensating actions if any chain step throws an exception.

---

## 8. Self-Check Questions

1. How do conditional chains improve agent adaptability compared to static pipelines?  
2. Why is provider abstraction critical for maintaining a multi-chain workflow?  
3. Describe a scenario requiring a compensating action in a chained agent.  
4. How would you implement a fallback for a failed chain step without stopping the entire workflow?

---

**Next Up:**  
Lesson 2-C will explore **Hybrid RAG & Context Management**, combining graph-based retrieval, vector search, and long-context strategies for robust knowledge grounding.  
```