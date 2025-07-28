---
title: "2-1 Agent Loop Deep Dive"
---

# 🔄 Lesson 2-1: Agent Loop Deep Dive

!!! note "Learning Objectives"
    By the end of this lesson, you will able to:

    - Describe each phase of the ReAct loop (Reasoning ↔ Acting) in detail
    - Implement dead‐loop and overall timeout guards to prevent infinite cycles
    - Integrate retry, exponential backoff, and fallback strategies into your agent loop
    - Instrument the loop with structured logging and tracing for full observability

---

## 🔍 1. Overview of the ReAct Loop

The **ReAct loop** alternates between **Reasoning** (chain-of-thought planning) and **Acting** (tool invocation or LLM function calls), then captures **Observation** before repeating.

!!! example "ReAct Loop Pseudocode"
    ```python
    while not done and iteration < MAX_ITERATIONS:
        # Reasoning phase
        action, params = planner(memory)
        
        # Acting phase
        result = executor.execute(action, params)
        
        # Observation phase
        memory.append({
            "step": iteration,
            "action": action,
            "params": params,
            "result": result
        })
        
        iteration += 1
    ```

!!! tip "Key Components"
    - **Reasoning**: LLM decides what action to take next
    - **Acting**: Execute the chosen action (tool call, API call, etc.)
    - **Observation**: Capture and store the result
    - **Memory**: Maintain context across iterations

---

## 🛡️ 2. Dead-Loop Prevention

### 2.1 Max Iterations Guard

!!! warning "Infinite Loop Risk"
    Agents can get stuck in infinite loops if they don't recognize completion conditions.

!!! example "Max Iterations Implementation"
    ```python
    MAX_ITERATIONS = 10
    
    def agent_loop():
        iteration = 0
        while iteration < MAX_ITERATIONS:
            # ... agent logic ...
            iteration += 1
        
        if iteration >= MAX_ITERATIONS:
            raise MaxIterationsError("Agent exceeded maximum iterations")
    ```

### 2.2 Global Timeout Guard

!!! example "Global Timeout Implementation"
    ```python
    import time
    
    GLOBAL_TIMEOUT = 300  # 5 minutes
    
    def agent_loop():
        start_time = time.time()
        
        while not done:
            if time.time() - start_time > GLOBAL_TIMEOUT:
                raise TimeoutError("Agent loop exceeded total time limit.")
            # per‐action timer inside executor.execute()
    ```

---

## 🔄 3. Retry and Backoff Strategies

### 3.1 Identifying Transient Failures

!!! info "Transient Failure Types"
    - Network errors, rate limits, intermittent API errors

### 3.2 Exponential Backoff

!!! example "Exponential Backoff Implementation"
    ```python
    import random
    import time
    
    def retry_with_backoff(fn, max_retries=3, base_delay=1.0, max_delay=10.0):
        for attempt in range(max_retries):
            try:
                return fn()
            except TransientError:
                delay = min(base_delay * 2**attempt, max_delay)
                time.sleep(delay + random.uniform(0, 0.1))
        return fallback()
    ```

### 3.3 Fallback Actions

!!! tip "Fallback Strategy"
    - If retries exhausted, call a safe fallback tool or return a default response
    - Optionally escalate to human-in-loop

---

## 📊 4. Observability & Tracing

### 4.1 Structured Logging

!!! example "Structured Logging Format"
    ```json
    {
      "timestamp": "2024-01-15T10:30:00Z",
      "iteration": 3,
      "action": "fetch_pricing_docs",
      "params": {"competitor": "CompanyX"},
      "result": "Pricing data retrieved successfully",
      "duration_ms": 345
    }
    ```

### 4.2 Integration with Tracing Dashboards

!!! info "Tracing Integration"
    - Use OpenTelemetry or LangSmith SDK to emit spans for planning, execution, and retrieval
    - Visualize trace graphs showing sequence, timing, and errors

---

## 💻 5. Mini-Project: Robust ReAct Loop Implementation

!!! success "Robust ReAct Loop Challenge"
    Enhance your Phase 0 calculator agent to include:

    1. **Max Iterations Guard:** stop after `N` steps with a clear message
    2. **Per-Action Timeout:** raise and catch timeouts in executor
    3. **Retry & Backoff:** handle transient errors up to 3 attempts
    4. **Fallback:** if retries fail, return `"Operation failed, please retry later."`
    5. **Logging:** write each iteration's details to `agent_log.json`

!!! example "Starter Skeleton"
    ```python
    import time
    import json
    
    MAX_STEPS = 5
    GLOBAL_TIMEOUT = 60  # seconds
    
    def planner(memory):
        # simple planner stub
        return "calculate", {"expr": "2+2"}
    
    def executor(action, params):
        # implement per-action timeout and raise TransientError if needed
        return str(eval(params["expr"]))
    ```

---

## ❓ 6. Self-Check Questions

!!! question "Knowledge Check"
    1. What are the three main phases of the ReAct loop?
    2. Why is exponential backoff important for retry strategies?
    3. How would you implement a per-action timeout in your executor?
    4. What information should be logged for each agent iteration?

---

## 🧭 Navigation

!!! success "Next Up"
    **[Lesson 2-2: Prompt & Tool Chaining →](lesson-2.md)**

    Learn how to chain prompts and tools together for complex workflows.  