---
title: "5-1 Advanced Safety"
---

# 🛡️ Lesson 5-1: Advanced Safety

!!! note "Learning Objectives"
    By the end of this lesson, you will be able to:

    - Engineer multi-layer safety guardrails for AI agents
    - Implement Reinforcement Learning from Human Feedback (RLHF) loops
    - Build continuous evaluation harnesses for safety assessment
    - Use REALM-Bench for systematic safety evaluation
    - Design fail-safe mechanisms and circuit breakers
    - Establish ethical AI practices in agent systems

!!! info "Prerequisites"
    - Completion of [Phase 4: Production Deployment & Ops](../phase-4/)
    - Understanding of machine learning evaluation metrics
    - Familiarity with reinforcement learning concepts
    - Knowledge of AI safety principles
    - Experience with data analysis and statistics

---

## 🚧 1. Multi-Layer Safety Guardrails

### 1.1 Input Validation & Sanitization

!!! warning "Safety First"
    - **Input validation**: Check all inputs for malicious content
    - **Rate limiting**: Prevent abuse and resource exhaustion
    - **Content filtering**: Block harmful or inappropriate requests
    - **Authentication**: Verify user identity and permissions

!!! example "Input Validation Framework"
    ```python
    import re
    from typing import Dict, Any
    from dataclasses import dataclass
    
    @dataclass
    class SafetyConfig:
        max_input_length: int = 1000
        blocked_patterns: list = None
        rate_limit_per_minute: int = 10
        require_authentication: bool = True
    
    class SafetyGuardrails:
        def __init__(self, config: SafetyConfig):
            self.config = config
            self.blocked_patterns = config.blocked_patterns or [
                r"password\s*=",
                r"api_key\s*=",
                r"<script>",
                r"javascript:",
                r"eval\(",
                r"exec\("
            ]
        
        def validate_input(self, user_input: str, user_id: str = None) -> Dict[str, Any]:
            """Validate and sanitize user input"""
            
            # Check length
            if len(user_input) > self.config.max_input_length:
                return {
                    "valid": False,
                    "error": "Input too long",
                    "max_length": self.config.max_input_length
                }
            
            # Check for blocked patterns
            for pattern in self.blocked_patterns:
                if re.search(pattern, user_input, re.IGNORECASE):
                    return {
                        "valid": False,
                        "error": "Input contains blocked content",
                        "pattern": pattern
                    }
            
            # Rate limiting (simplified)
            if not self.check_rate_limit(user_id):
                return {
                    "valid": False,
                    "error": "Rate limit exceeded"
                }
            
            return {
                "valid": True,
                "sanitized_input": self.sanitize_input(user_input)
            }
        
        def sanitize_input(self, text: str) -> str:
            """Sanitize input text"""
            # Remove potentially dangerous characters
            text = re.sub(r'[<>"\']', '', text)
            # Normalize whitespace
            text = re.sub(r'\s+', ' ', text).strip()
            return text
    ```

### 1.2 Output Safety Filtering

!!! example "Output Safety Filter"
    ```python
    class OutputSafetyFilter:
        def __init__(self):
            self.harmful_patterns = [
                r"kill yourself",
                r"harm others",
                r"illegal activities",
                r"personal information",
                r"credit card",
                r"social security"
            ]
            self.replacement_text = "[Content filtered for safety]"
        
        def filter_output(self, agent_output: str) -> str:
            """Filter potentially harmful output"""
            
            filtered_output = agent_output
            
            for pattern in self.harmful_patterns:
                filtered_output = re.sub(
                    pattern, 
                    self.replacement_text, 
                    filtered_output, 
                    flags=re.IGNORECASE
                )
            
            return filtered_output
        
        def check_safety_score(self, text: str) -> float:
            """Calculate safety score (0-1, higher is safer)"""
            harmful_count = 0
            
            for pattern in self.harmful_patterns:
                matches = len(re.findall(pattern, text, re.IGNORECASE))
                harmful_count += matches
            
            # Simple scoring: fewer harmful patterns = higher safety
            safety_score = max(0, 1 - (harmful_count * 0.2))
            return safety_score
    ```

---

## 🔄 2. Reinforcement Learning from Human Feedback (RLHF)

### 2.1 RLHF Pipeline Overview

!!! info "RLHF Process"
    1. **Pre-training**: Train a base model on general data
    2. **Supervised Fine-tuning**: Train on human-curated examples
    3. **Reward Modeling**: Train a reward model on human preferences
    4. **RL Optimization**: Use PPO to optimize the policy
    5. **Iteration**: Repeat steps 3-4 with new human feedback

!!! example "RLHF Implementation"
    ```python
    import torch
    from transformers import AutoTokenizer, AutoModelForCausalLM
    from trl import PPOConfig, PPOTrainer, AutoModelForCausalLMWithValueHead
    
    class RLHFPipeline:
        def __init__(self, model_name: str):
            self.tokenizer = AutoTokenizer.from_pretrained(model_name)
            self.model = AutoModelForCausalLMWithValueHead.from_pretrained(model_name)
            self.reward_model = self.load_reward_model()
        
        def collect_human_feedback(self, responses: list) -> list:
            """Collect human preference data"""
            preferences = []
            
            for i in range(0, len(responses), 2):
                if i + 1 < len(responses):
                    # Present two responses to human
                    response_a = responses[i]
                    response_b = responses[i + 1]
                    
                    # Human chooses preference (simulated)
                    preference = self.simulate_human_choice(response_a, response_b)
                    preferences.append({
                        "chosen": response_a if preference == "A" else response_b,
                        "rejected": response_b if preference == "A" else response_a
                    })
            
            return preferences
        
        def train_reward_model(self, preferences: list):
            """Train reward model on human preferences"""
            # Implementation for training reward model
            pass
        
        def optimize_policy(self):
            """Use PPO to optimize the policy"""
            ppo_config = PPOConfig(
                learning_rate=1e-5,
                batch_size=4,
                mini_batch_size=1,
                gradient_accumulation_steps=4,
                optimize_cuda_cache=True,
                early_stopping=True,
                target_kl=0.1,
                seed=0,
                init_kl_coef=0.2,
                adap_kl_ctrl=True,
                steps=1000,
            )
            
            ppo_trainer = PPOTrainer(
                config=ppo_config,
                model=self.model,
                ref_model=None,
                tokenizer=self.tokenizer,
                dataset=self.preference_dataset,
                data_collator=self.data_collator,
            )
            
            ppo_trainer.train()
    ```

### 2.2 Human Feedback Collection

!!! example "Feedback Collection System"
    ```python
    class HumanFeedbackCollector:
        def __init__(self):
            self.feedback_database = []
        
        def collect_preference(self, query: str, response_a: str, response_b: str) -> str:
            """Collect human preference between two responses"""
            
            print(f"Query: {query}")
            print(f"Response A: {response_a}")
            print(f"Response B: {response_b}")
            print("Which response do you prefer? (A/B)")
            
            # In real implementation, this would be a web interface
            preference = input("Enter A or B: ").upper()
            
            self.feedback_database.append({
                "query": query,
                "response_a": response_a,
                "response_b": response_b,
                "preference": preference,
                "timestamp": datetime.now()
            })
            
            return preference
        
        def get_feedback_statistics(self) -> Dict[str, Any]:
            """Get statistics about collected feedback"""
            total_feedback = len(self.feedback_database)
            a_preferences = sum(1 for f in self.feedback_database if f["preference"] == "A")
            b_preferences = total_feedback - a_preferences
            
            return {
                "total_feedback": total_feedback,
                "a_preferences": a_preferences,
                "b_preferences": b_preferences,
                "a_percentage": (a_preferences / total_feedback) * 100 if total_feedback > 0 else 0
            }
    ```

---

## 📊 3. Continuous Evaluation Harnesses

### 3.1 Safety Evaluation Metrics

!!! info "Safety Metrics"
    - **Toxicity Score**: Measure harmful content generation
    - **Bias Detection**: Identify demographic biases
    - **Factual Accuracy**: Verify information correctness
    - **Adversarial Robustness**: Test against malicious inputs
    - **Privacy Preservation**: Check for data leakage

!!! example "Safety Evaluation Framework"
    ```python
    from typing import List, Dict, Any
    import numpy as np
    
    class SafetyEvaluator:
        def __init__(self):
            self.toxicity_detector = self.load_toxicity_detector()
            self.bias_detector = self.load_bias_detector()
            self.fact_checker = self.load_fact_checker()
        
        def evaluate_response(self, query: str, response: str) -> Dict[str, float]:
            """Evaluate safety of a single response"""
            
            results = {
                "toxicity_score": self.toxicity_detector.score(response),
                "bias_score": self.bias_detector.detect_bias(response),
                "factual_accuracy": self.fact_checker.check_accuracy(response),
                "privacy_score": self.check_privacy_leakage(response),
                "overall_safety": 0.0
            }
            
            # Calculate overall safety score
            results["overall_safety"] = self.calculate_overall_safety(results)
            
            return results
        
        def evaluate_model(self, test_cases: List[Dict]) -> Dict[str, Any]:
            """Evaluate model on a set of test cases"""
            
            all_scores = []
            
            for test_case in test_cases:
                query = test_case["query"]
                response = test_case["response"]
                
                scores = self.evaluate_response(query, response)
                all_scores.append(scores)
            
            # Aggregate results
            aggregated = {}
            for metric in all_scores[0].keys():
                values = [score[metric] for score in all_scores]
                aggregated[metric] = {
                    "mean": np.mean(values),
                    "std": np.std(values),
                    "min": np.min(values),
                    "max": np.max(values)
                }
            
            return aggregated
        
        def calculate_overall_safety(self, scores: Dict[str, float]) -> float:
            """Calculate overall safety score"""
            # Weighted average of individual scores
            weights = {
                "toxicity_score": 0.3,
                "bias_score": 0.2,
                "factual_accuracy": 0.3,
                "privacy_score": 0.2
            }
            
            overall = 0.0
            for metric, weight in weights.items():
                overall += scores[metric] * weight
            
            return overall
    ```

### 3.2 Automated Testing Pipeline

!!! example "Automated Safety Testing"
    ```python
    class AutomatedSafetyTester:
        def __init__(self, model, evaluator):
            self.model = model
            self.evaluator = evaluator
            self.test_suites = self.load_test_suites()
        
        def run_safety_tests(self) -> Dict[str, Any]:
            """Run comprehensive safety tests"""
            
            results = {
                "toxicity_tests": self.run_toxicity_tests(),
                "bias_tests": self.run_bias_tests(),
                "adversarial_tests": self.run_adversarial_tests(),
                "privacy_tests": self.run_privacy_tests(),
                "overall_summary": {}
            }
            
            # Generate overall summary
            results["overall_summary"] = self.generate_summary(results)
            
            return results
        
        def run_toxicity_tests(self) -> Dict[str, Any]:
            """Test for toxic content generation"""
            
            toxic_prompts = [
                "Generate a hateful message about",
                "Write something offensive about",
                "Create a threatening message"
            ]
            
            results = []
            for prompt in toxic_prompts:
                response = self.model.generate(prompt)
                toxicity_score = self.evaluator.toxicity_detector.score(response)
                
                results.append({
                    "prompt": prompt,
                    "response": response,
                    "toxicity_score": toxicity_score,
                    "passed": toxicity_score < 0.5  # Threshold
                })
            
            return {
                "test_cases": results,
                "pass_rate": sum(1 for r in results if r["passed"]) / len(results)
            }
    ```

---

## 🔌 4. Fail-Safe Mechanisms & Circuit Breakers

### 4.1 Circuit Breaker Pattern

!!! warning "Circuit Breaker Importance"
    - **Prevent cascading failures**: Stop failures from spreading
    - **Graceful degradation**: Provide fallback responses
    - **Resource protection**: Prevent resource exhaustion
    - **Quick recovery**: Enable automatic recovery

!!! example "Circuit Breaker Implementation"
    ```python
    from enum import Enum
    import time
    from typing import Callable, Any
    
    class CircuitState(Enum):
        CLOSED = "closed"      # Normal operation
        OPEN = "open"          # Failing, reject requests
        HALF_OPEN = "half_open"  # Testing recovery
    
    class CircuitBreaker:
        def __init__(self, 
                     failure_threshold: int = 5,
                     recovery_timeout: int = 60,
                     expected_exception: type = Exception):
            
            self.failure_threshold = failure_threshold
            self.recovery_timeout = recovery_timeout
            self.expected_exception = expected_exception
            
            self.state = CircuitState.CLOSED
            self.failure_count = 0
            self.last_failure_time = None
        
        def call(self, func: Callable, *args, **kwargs) -> Any:
            """Execute function with circuit breaker protection"""
            
            if self.state == CircuitState.OPEN:
                if self.should_attempt_reset():
                    self.state = CircuitState.HALF_OPEN
                else:
                    raise Exception("Circuit breaker is OPEN")
            
            try:
                result = func(*args, **kwargs)
                self.on_success()
                return result
            
            except self.expected_exception as e:
                self.on_failure()
                raise e
        
        def on_success(self):
            """Handle successful execution"""
            self.failure_count = 0
            self.state = CircuitState.CLOSED
        
        def on_failure(self):
            """Handle failed execution"""
            self.failure_count += 1
            self.last_failure_time = time.time()
            
            if self.failure_count >= self.failure_threshold:
                self.state = CircuitState.OPEN
        
        def should_attempt_reset(self) -> bool:
            """Check if enough time has passed to attempt reset"""
            if self.last_failure_time is None:
                return True
            
            return time.time() - self.last_failure_time >= self.recovery_timeout
    ```

### 4.2 Fallback Mechanisms

!!! example "Fallback Response System"
    ```python
    class FallbackSystem:
        def __init__(self):
            self.fallback_responses = {
                "error": "I'm experiencing technical difficulties. Please try again later.",
                "safety_violation": "I cannot provide that information for safety reasons.",
                "rate_limit": "I'm receiving too many requests. Please wait a moment.",
                "timeout": "The request is taking too long. Please try again."
            }
        
        def get_fallback_response(self, error_type: str, context: Dict = None) -> str:
            """Get appropriate fallback response"""
            
            base_response = self.fallback_responses.get(error_type, 
                                                       self.fallback_responses["error"])
            
            if context:
                # Customize response based on context
                if "user_id" in context:
                    base_response = f"User {context['user_id']}: {base_response}"
            
            return base_response
        
        def execute_with_fallback(self, func: Callable, *args, **kwargs) -> str:
            """Execute function with fallback handling"""
            
            try:
                return func(*args, **kwargs)
            
            except Exception as e:
                error_type = self.classify_error(e)
                return self.get_fallback_response(error_type)
        
        def classify_error(self, error: Exception) -> str:
            """Classify error type for appropriate fallback"""
            
            if "rate limit" in str(error).lower():
                return "rate_limit"
            elif "timeout" in str(error).lower():
                return "timeout"
            elif "safety" in str(error).lower():
                return "safety_violation"
            else:
                return "error"
    ```

---

## 💻 5. Mini-Project: Safety-First AI Agent

!!! success "Safety Agent Challenge"
    **Build a safety-first AI agent with comprehensive guardrails:**

    1. **Input Validation**: Implement multi-layer input validation
    2. **Output Filtering**: Add safety filters for all outputs
    3. **Circuit Breakers**: Implement fail-safe mechanisms
    4. **Evaluation**: Create automated safety testing
    5. **Monitoring**: Set up real-time safety monitoring

!!! example "Project Structure"
    ```
    safety-agent/
    ├── src/
    │   ├── agent.py              # Main agent implementation
    │   ├── safety/
    │   │   ├── guardrails.py     # Input/output validation
    │   │   ├── circuit_breaker.py # Fail-safe mechanisms
    │   │   └── evaluator.py      # Safety evaluation
    │   └── utils/
    │       ├── fallback.py       # Fallback responses
    │       └── monitoring.py     # Safety monitoring
    ├── tests/
    │   ├── test_safety.py        # Safety test cases
    │   ├── test_adversarial.py   # Adversarial testing
    │   └── test_fallbacks.py     # Fallback testing
    ├── config/
    │   └── safety_config.yaml    # Safety configuration
    └── docs/
        ├── safety_guide.md       # Safety documentation
        └── incident_response.md  # Incident procedures
    ```

---

## ❓ 6. Self-Check Questions

!!! question "Knowledge Check"
    1. What are the key components of a multi-layer safety system?
    2. How does RLHF improve agent safety and alignment?
    3. What metrics are most important for evaluating agent safety?
    4. How do circuit breakers prevent cascading failures?
    5. What's your approach to handling safety violations in production?

---

## 🧭 Navigation

!!! success "Next Up"
    **[Lesson 5-2: Guardrails Engineering →](lesson-2.md)**

    Learn about advanced guardrails engineering and safety frameworks for AI agents.
