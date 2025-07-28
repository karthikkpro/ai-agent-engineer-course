---
title: "1-1 Python for Agents"
---

# 🐍 Lesson 1-1: Python for Agents

!!! note "Learning Objectives"
    By the end of this lesson, you'll be able to:

    - Implement tool abstractions as Python functions and registries
    - Integrate external APIs using HTTP requests and robust error handling
    - Structure agent code using asynchronous (async/await) patterns for performance
    - Validate agent tool I/O with Pydantic schemas for strong typing and reliability

---

## 🎯 1. Why Python for AI Agents?

Python is the language of choice for agent development due to:

- Mature AI/ML ecosystem (PyTorch, LangChain, Hugging Face, OpenAI SDKs)
- Rapid prototyping for bots, APIs, data pipelines, and orchestration layers
- Widespread libraries for HTTP, async, data validation, and API integration

!!! tip "Key Insight"
    Most open-source agent frameworks and LLM APIs (LangChain, CrewAI, FastAPI) use Python as their first-class citizen.

---

## 🛠️ 2. Core Python Primitives for Agents

Agent "tools" are simply Python functions with clear input/output signatures. A common pattern is to store them in a dictionary registry, allowing dynamic lookup and dispatch.

!!! example "Tool Abstraction and Registry"
    ```python
    def weather_tool(city: str) -> str:
        # Here you'd call a real API (see below)
        return f"Weather for {city} is sunny."

    def greet_tool(name: str) -> str:
        return f"Hello, {name}!"

    tool_registry = {
        "weather": weather_tool,
        "greet": greet_tool
    }

    # Dynamic dispatch based on tool name
    tool_name = "weather"
    result = tool_registry[tool_name]("London")
    print(result)  # Output: Weather for London is sunny.
    ```

!!! tip "Key Principles"
    - Tools are single-responsibility and reusable
    - Exceptions should be handled to avoid agent crashes

!!! example "Exception Handling"
    ```python
    def safe_tool(fn):
        def wrapper(*args, **kwargs):
            try:
                return fn(*args, **kwargs)
            except Exception as e:
                return f"Error: {e}"
        return wrapper

    @safe_tool
    def fragile_tool(x):
        return 10 / x  # Will error if x == 0
    ```

---

## 🌐 3. HTTP and API Integration

Most practical tools will query external APIs (weather, search, databases). Python's `requests` library is standard for HTTP, and robust error handling is critical.

!!! example "Minimal Weather API Example"
    ```python
    import requests

    def weather_tool(city: str) -> str:
        try:
            url = "https://wttr.in/{}?format=3".format(city)
            response = requests.get(url, timeout=3)
            response.raise_for_status()
            return response.text
        except Exception as e:
            return f"Could not get weather: {e}"
    ```

!!! tip "Best Practices"
    - Use `raise_for_status()` to catch HTTP errors
    - Always set a `timeout` to avoid agent stalls
    - Wrap API calls in try/except blocks for resilience

!!! warning "Important Note"
    All real-world agents interact with some form of remote API or service—error handling isn't optional!

---

## ⚡ 4. Asynchronous Programming (AsyncIO)

Agents often perform multiple slow actions (API calls, file reads) in parallel. Asynchronous programming radically speeds up workflows.

!!! tip "Why async/await?"
    - Non-blocking: agent can issue multiple HTTP requests without waiting for each one to finish
    - Crucial for data pipelines, multi-tool orchestration, or chat bots that need low-latency responses

!!! example "Async Example using httpx"
    ```python
    import httpx
    import asyncio

    async def async_weather_tool(city: str) -> str:
        async with httpx.AsyncClient() as client:
            try:
                r = await client.get(f"https://wttr.in/{city}?format=3", timeout=3.0)
                r.raise_for_status()
                return r.text
            except Exception as e:
                return f"Could not get weather: {e}"

    async def main():
        result = await async_weather_tool("London")
        print(result)

    # asyncio.run(main())
    ```

!!! info "Diagram Suggestion"
    Agent loop with branches for sequential (slow) vs. async (fast, concurrent) API calls. Show three tool calls issued in parallel (concurrent lines running to API cloud), converging back to a response funnel.

---

## 🔍 5. Data Validation: Pydantic Schemas

Robust agent systems require strong input and output validation—otherwise, bad data will break the pipeline. Pydantic—used by FastAPI and many AI frameworks—enforces clear schemas.

!!! example "Pydantic Example"
    ```python
    from pydantic import BaseModel

    class WeatherParams(BaseModel):
        city: str

    def weather_tool(params: WeatherParams) -> str:
        # params is now guaranteed to have a city attribute
        return f"Weather for {params.city} is sunny."
    ```

!!! tip "Pydantic Benefits"
    - Pydantic auto-converts and checks types (e.g., str, int, float)
    - Invalid inputs throw clear errors before tool runs

!!! info "Integration"
    Use Pydantic to define all tool function I/O contracts. Combined with FastAPI, you get automatic documentation and validation for any agent-exposed API.

---

## 💻 6. Mini-Project: Weather Query Agent

!!! success "Weather Query Agent Challenge"
    **Goal:** Build a CLI script or notebook cell that:
    - Accepts a city name from the user
    - Queries a weather API using your tool function
    - Prints back the weather (handle errors gracefully)

    **Steps:**
    - Write a Python function using `requests` to fetch weather info from an API (see above)
    - Wrap the function in a tool registry
    - Add basic error handling and print a user-friendly result
    - (Optionally) Add a Pydantic schema to cleanly parse the city input

    **Snippet:**
    ```python
    city = input("Enter city: ")
    print(weather_tool(city))
    ```

---

## ❓ 7. Self-Check Questions

!!! question "Knowledge Check"
    1. Why is using async/await important in agent tool execution?
    2. What gap does Pydantic fill for agent systems?
    3. How would you handle or log a "tool not found" situation in your registry-based agent code?
    4. What is the main advantage of using a tool registry pattern in agent code?

---

## 🧭 Navigation

!!! success "Next Up"
    **[Lesson 1-2: Prompt Engineering →](lesson-2.md)**

    In Lesson 1-2, you'll learn prompt engineering essentials for agents—crafting reusable templates, input/output structures, and reasoning chains for effective LLM-powered workflows.

---

[image: to be inserted – a diagram illustrating an agent workflow with Python tool functions, async callouts, and schematic tool registry]

*All code provided is suitable for running in a Jupyter notebook, CLI, or as Python modules—use these examples as your agent engineering foundation!*
```