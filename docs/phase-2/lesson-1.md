---
title: "2-A Agent Loop Deep Dive"
---

# Lesson 2-A: Agent Loop Deep Dive

**Learning Objectives**  
By the end of this lesson, you will able to:  
- Describe each phase of the ReAct loop (Reasoning ↔ Acting) in detail  
- Implement dead‐loop and overall timeout guards to prevent infinite cycles  
- Integrate retry, exponential backoff, and fallback strategies into your agent loop  
- Instrument the loop with structured logging and tracing for full observability  

---

## 1. Overview of the ReAct Loop

The **ReAct loop** alternates between **Reasoning** (chain-of-thought planning) and **Acting** (tool invocation or LLM function calls), then captures **Observation** before repeating.  

**Pseudocode Skeleton:**
```
while not done and iteration  GLOBAL_TIMEOUT:
        raise TimeoutError("Agent loop exceeded total time limit.")
    # per‐action timer inside executor.execute()
```

---

## 4. Retry and Backoff Strategies

### 4.1 Identifying Transient Failures  
- Network errors, rate limits, intermittent API errors  

### 4.2 Exponential Backoff  
- Retry delays: `delay = base * (2 ** attempt)` capped by `max_delay`  
- `max_retries` limit after which fallback is invoked  

```
import random

def retry_with_backoff(fn, max_retries=3, base_delay=1.0, max_delay=10.0):
    for attempt in range(max_retries):
        try:
            return fn()
        except TransientError:
            delay = min(base_delay * 2**attempt, max_delay)
            time.sleep(delay + random.uniform(0, 0.1))
    return fallback()
```

### 4.3 Fallback Actions  
- If retries exhausted, call a safe fallback tool or return a default response  
- Optionally escalate to human-in-loop  

---

## 5. Observability & Tracing

### 5.1 Structured Logging  
- Log each loop iteration as JSON:  
  ```
  {
    "timestamp": "...",
    "iteration": 3,
    "action": "fetch_pricing_docs",
    "params": {"competitor": "CompanyX"},
    "result": "...",
    "duration_ms": 345
  }
  ```

### 5.2 Integration with Tracing Dashboards  
- Use OpenTelemetry or LangSmith SDK to emit spans for planning, execution, and retrieval  
- Visualize trace graphs showing sequence, timing, and errors  

---

## 6. Mini-Project: Robust ReAct Loop Implementation

Enhance your Phase 0 calculator agent to include:  
1. **Max Iterations Guard:** stop after `N` steps with a clear message  
2. **Per-Action Timeout:** raise and catch timeouts in executor  
3. **Retry & Backoff:** handle transient errors up to 3 attempts  
4. **Fallback:** if retries fail, return `"Operation failed, please retry later."`  
5. **Logging:** write each iteration’s details to `agent_log.json`  

**Starter Skeleton:**
```
import time, json

MAX_STEPS = 5
GLOBAL_TIMEOUT = 60  # seconds

def planner(memory):
    # simple planner stub
    return "calculate", {"expr": "2+2"}

def executor(action, params):
    # implement per-action timeout and raise TransientError if needed
    return str(eval(params["expr"]))

def fallback():
    return "Fallback response."

def main():
    start_time = time.time()
    memory, logs = [], []
    for i in range(MAX_STEPS):
        if time.time() - start_time > GLOBAL_TIMEOUT:
            print("Agent loop timed out.")
            break
        action, params = planner(memory)
        # retry logic with backoff
        result = retry_with_backoff(lambda: executor(action, params))
        memory.append({"action": action, "result": result})
        logs.append({
            "iteration": i+1,
            "action": action,
            "params": params,
            "result": result,
            "timestamp": time.time()
        })
        if action == "finish":
            break
    with open("agent_log.json", "w") as f:
        json.dump(logs, f, indent=2)

if __name__ == "__main__":
    main()
```

---

## 7. Self-Check Questions

1. How does your agent detect and escape a dead loop?  
2. What is the difference between per-action and global timeout guards?  
3. Outline an exponential backoff algorithm and its parameters.  
4. Why is structured logging crucial for debugging agent workflows?  

---

**Next Up:**  
Lesson 2-B will cover **Prompt & Tool Chaining**—building branching logic, dynamic routing, and provider abstraction in agent workflows.  
```