---
title: "1-5 Model APIs – Brains of Your Agent"
---

# 🧠 Lesson 1-5: Model APIs – Brains of Your Agent

!!! note "Learning Objectives"
    By the end of this lesson, you will be able to:

    - Describe the major commercial and open-source LLM APIs (OpenAI, Anthropic, Hugging Face, Cohere)
    - Compare model capabilities: context window, multi-modality, safety, and instruction tuning
    - Evaluate trade-offs among cost, latency, and output quality for different providers
    - Programmatically switch between model APIs in your agent code
    - Implement A/B benchmarking to select the optimal model for each agent task

---

## 🌐 1. The LLM API Ecosystem

### 1.1 OpenAI GPT Family

!!! info "GPT-4 / GPT-4o"
    - Up to 128K token context, multimodal (vision, audio) support
    - Function-calling API for structured tool invocation
    - Pricing: $0.03/1K prompt tokens, $0.06/1K completion tokens (approx.)

### 1.2 Anthropic Claude

!!! info "Claude 4"
    - Stronger safety mitigations, high-quality long-form reasoning
    - Context up to 100K tokens, chat-first interface
    - Pricing: $0.10/1K input, $0.20/1K output (approx.)

### 1.3 Hugging Face Inference API

!!! info "Vicuna, LLaMA-3, Mistral"
    - Variety of open models hosted, free/community tiers and paid options
    - Context windows vary (8K to 128K) depending on model
    - Pricing: model-specific; can be as low as $0.001/1K tokens

### 1.4 Cohere Command Models

!!! info "Command Rerank, Command Light"
    - Optimized for classification and summarization tasks
    - Context windows ~16K tokens
    - Pricing: competitive, per-request bundles

---

## ⚖️ 2. Model Selection Criteria

!!! info "Model Comparison Table"
    | Criterion              | Considerations                                                                 |
    |------------------------|--------------------------------------------------------------------------------|
    | Context Window         | Longer windows allow larger prompts, document-grounded tasks                    |
    | Instruction Tuning     | Models fine-tuned for following instructions yield more reliable tool planning |
    | Multi-Modality         | Vision/audio support enables multimodal agent use cases                        |
    | Safety & Alignment     | Inherent guardrails, risk of harmful outputs                                   |
    | Latency & Throughput   | Response time per token; important for real-time agents                        |
    | Pricing Model          | Per-token vs. fixed-price, streaming discounts                                 |
    | Function Calling       | Native support for structured tool calls                                       |

---

## 🔄 3. Programmatic Model Switching

Abstract your agent's "LLM client" behind a common interface:

!!! example "LLM Client Abstraction"
    ```python
    class LLMClient:
        def __init__(self, provider: str, model: str, api_key: str):
            self.provider = provider
            if provider == "openai":
                import openai
                self.client = openai
                self.model = model
            elif provider == "anthropic":
                import anthropic
                self.client = anthropic.Client(api_key)
                self.model = model
            # Add Hugging Face, Cohere similarly

        def generate(self, prompt: str, **kwargs) -> str:
            if self.provider == "openai":
                resp = self.client.ChatCompletion.create(
                    model=self.model, messages=[{"role": "user", "content": prompt}], **kwargs
                )
                return resp.choices.message.content
            elif self.provider == "anthropic":
                resp = self.client.completions.create(
                    model=self.model, prompt=prompt, **kwargs
                )
                return resp.completion
            # Add other providers
    ```

!!! tip "Best Practices"
    - **Authentication:** Store API keys in environment variables or secret manager
    - **Error Handling:** Catch rate-limit and authentication errors, implement exponential backoff

---

## ⚡ 4. Streaming vs. Non-Streaming Calls

### 4.1 Non-Streaming

!!! info "Non-Streaming Characteristics"
    - Simplest: entire completion returned at once
    - Use for batch or background tasks

### 4.2 Streaming

!!! info "Streaming Characteristics"
    - Yields tokens as they are generated
    - Improves perceived responsiveness in UIs or chat agents
    - **OpenAI Example (Python):**
      ```python
      for chunk in openai.ChatCompletion.create(
          model="gpt-4o", messages=[...], stream=True
      ):
          print(chunk.choices.delta.get("content", ""), end="")
      ```

---

## 5. A/B Benchmarking & Cost Analysis

### 5.1 Designing a Benchmark Harness

!!! info "Benchmark Harness Details"
    - **Fixed Prompt Set:** 20–50 representative prompts for your agent’s domain
    - **Metrics:**
      - *Quality:* human-rated accuracy, relevance, coherence
      - *Latency:* average ms per token or per response
      - *Cost:* tokens used × per-token price

### 5.2 Automating Measurements

!!! example "Benchmarking Script"
    ```python
    import time
    import csv

    models = [("openai", "gpt-4o"), ("anthropic", "claude-4")]
    results = []

    for provider, model in models:
        client = LLMClient(provider, model, api_key=os.getenv("API_KEY"))
        for prompt in test_prompts:
            start = time.time()
            resp = client.generate(prompt)
            latency = time.time() - start
            tokens = estimate_tokens(prompt + resp)
            cost = cost_per_token(provider, model) * tokens
            results.append([provider, model, prompt, resp, latency, cost])

    with open("benchmark.csv", "w", newline="") as f:
        writer = csv.writer(f)
        writer.writerow(["Provider", "Model", "Prompt", "Response", "Latency", "Cost"])
        writer.writerows(results)
    ```

---

## 💻 6. Mini-Project: Model Comparison Script

!!! success "Model Comparison Challenge"
    **Build a script that:**
    1. Reads a set of prompts from `prompts.txt`
    2. Sends each prompt to two different models (e.g., GPT-4o vs. Claude 4)
    3. Records response, latency, and token usage
    4. Outputs a summary table of average latency and cost per prompt

---

## ❓ 7. Self-Check Questions

!!! question "Knowledge Check"
    1. What factors would lead you to choose a cheaper open-source model over a premium API?
    2. How does streaming output change your agent's architecture?
    3. In your LLMClient abstraction, how would you add support for a new provider?
    4. Which metrics are most critical when benchmarking summarization versus conversational tasks?

---

## 🧭 Navigation

!!! success "Phase Complete!"
    **[Phase 2: Agentic Workflows & Reliability →](../phase-2/)**

    Phase 2 begins: diving into **Agentic Workflows & Reliability**—building robust ReAct loops, advanced RAG pipelines, and systematic evaluation for production-grade agents.  