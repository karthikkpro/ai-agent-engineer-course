---
title: "4-1 Production Deployment"
---

# 🚀 Lesson 4-1: Production Deployment

!!! note "Learning Objectives"
    By the end of this lesson, you will be able to:

    - Containerize AI agents with Docker and Kubernetes
    - Deploy serverless agent architectures on cloud platforms
    - Establish robust CI/CD pipelines for agent systems
    - Implement comprehensive observability and monitoring
    - Create incident response runbooks and procedures
    - Manage production agent environments at scale

!!! info "Prerequisites"
    - Completion of [Phase 3: Multi-Agent Orchestration](../phase-3/)
- Experience with containerization (Docker basics)
- Understanding of cloud platforms (AWS, GCP, or Azure)
- Familiarity with CI/CD concepts
- Basic DevOps knowledge

---

## 🐳 1. Containerization for AI Agents

### 1.1 Docker Best Practices

!!! tip "Containerization Strategy"
    - **Multi-stage builds**: Separate build and runtime environments
    - **Security scanning**: Scan for vulnerabilities in base images
    - **Resource limits**: Set CPU and memory constraints
    - **Health checks**: Implement proper health check endpoints

!!! example "Dockerfile for AI Agent"
    ```dockerfile
    # Multi-stage build for AI agent
    FROM python:3.11-slim as builder
    
    # Install build dependencies
    RUN apt-get update && apt-get install -y \
        build-essential \
        && rm -rf /var/lib/apt/lists/*
    
    # Copy requirements and install dependencies
    COPY requirements.txt .
    RUN pip install --no-cache-dir -r requirements.txt
    
    # Production stage
    FROM python:3.11-slim
    
    # Install runtime dependencies
    RUN apt-get update && apt-get install -y \
        curl \
        && rm -rf /var/lib/apt/lists/*
    
    # Copy installed packages from builder
    COPY --from=builder /usr/local/lib/python3.11/site-packages /usr/local/lib/python3.11/site-packages
    
    # Copy application code
    COPY app/ /app/
    WORKDIR /app
    
    # Set environment variables
    ENV PYTHONPATH=/app
    ENV PYTHONUNBUFFERED=1
    
    # Health check
    HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
        CMD curl -f http://localhost:8000/health || exit 1
    
    # Run the application
    CMD ["python", "main.py"]
    ```

### 1.2 Kubernetes Deployment

!!! example "Kubernetes Manifest"
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: ai-agent-deployment
      labels:
        app: ai-agent
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: ai-agent
      template:
        metadata:
          labels:
            app: ai-agent
        spec:
          containers:
          - name: ai-agent
            image: your-registry/ai-agent:latest
            ports:
            - containerPort: 8000
            resources:
              requests:
                memory: "512Mi"
                cpu: "250m"
              limits:
                memory: "1Gi"
                cpu: "500m"
            env:
            - name: OPENAI_API_KEY
              valueFrom:
                secretKeyRef:
                  name: ai-agent-secrets
                  key: openai-api-key
            livenessProbe:
              httpGet:
                path: /health
                port: 8000
              initialDelaySeconds: 30
              periodSeconds: 10
            readinessProbe:
              httpGet:
                path: /ready
                port: 8000
              initialDelaySeconds: 5
              periodSeconds: 5
    ```

---

## ☁️ 2. Serverless Architectures

### 2.1 AWS Lambda Deployment

!!! info "Serverless Benefits"
    - **Auto-scaling**: Automatically scale based on demand
    - **Cost-effective**: Pay only for execution time
    - **Managed infrastructure**: No server management required
    - **Event-driven**: Triggered by various AWS services

!!! example "Lambda Function for AI Agent"
    ```python
    import json
    import os
    from langchain.agents import initialize_agent, AgentType
    from langchain.llms import OpenAI
    
    def lambda_handler(event, context):
        """AWS Lambda handler for AI agent"""
        
        # Initialize the agent
        llm = OpenAI(
            temperature=0,
            openai_api_key=os.environ['OPENAI_API_KEY']
        )
        
        tools = [
            # Define your tools here
        ]
        
        agent = initialize_agent(
            tools,
            llm,
            agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
            verbose=True
        )
        
        # Extract query from event
        query = event.get('query', '')
        
        try:
            # Run the agent
            response = agent.run(query)
            
            return {
                'statusCode': 200,
                'body': json.dumps({
                    'response': response,
                    'request_id': context.aws_request_id
                })
            }
        except Exception as e:
            return {
                'statusCode': 500,
                'body': json.dumps({
                    'error': str(e),
                    'request_id': context.aws_request_id
                })
            }
    ```

### 2.2 API Gateway Integration

!!! example "API Gateway Configuration"
    ```yaml
    # serverless.yml
    service: ai-agent-service
    
    provider:
      name: aws
      runtime: python3.11
      region: us-east-1
      environment:
        OPENAI_API_KEY: ${env:OPENAI_API_KEY}
    
    functions:
      aiAgent:
        handler: handler.lambda_handler
        events:
          - http:
              path: /agent
              method: post
              cors: true
        memorySize: 1024
        timeout: 30
        reservedConcurrency: 10
    ```

---

## 🔄 3. CI/CD Pipelines

### 3.1 GitHub Actions Pipeline

!!! example "CI/CD Workflow"
    ```yaml
    name: AI Agent CI/CD
    
    on:
      push:
        branches: [ main ]
      pull_request:
        branches: [ main ]
    
    jobs:
      test:
        runs-on: ubuntu-latest
        steps:
        - uses: actions/checkout@v3
        
        - name: Set up Python
          uses: actions/setup-python@v4
          with:
            python-version: '3.11'
        
        - name: Install dependencies
          run: |
            pip install -r requirements.txt
            pip install -r requirements-dev.txt
        
        - name: Run tests
          run: |
            pytest tests/ --cov=app --cov-report=xml
            env:
              OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
        
        - name: Upload coverage
          uses: codecov/codecov-action@v3
          with:
            file: ./coverage.xml
    
      build-and-deploy:
        needs: test
        runs-on: ubuntu-latest
        if: github.ref == 'refs/heads/main'
        steps:
        - uses: actions/checkout@v3
        
        - name: Build Docker image
          run: |
            docker build -t ai-agent:${{ github.sha }} .
        
        - name: Push to registry
          run: |
            echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
            docker tag ai-agent:${{ github.sha }} your-registry/ai-agent:latest
            docker push your-registry/ai-agent:latest
        
        - name: Deploy to Kubernetes
          run: |
            kubectl set image deployment/ai-agent-deployment ai-agent=your-registry/ai-agent:latest
    ```

### 3.2 Security Scanning

!!! warning "Security Considerations"
    - **Dependency scanning**: Check for vulnerabilities in dependencies
    - **Container scanning**: Scan Docker images for security issues
    - **Secret management**: Use secure secret management systems
    - **Access control**: Implement proper RBAC and IAM policies

!!! example "Security Scanning Step"
    ```yaml
    - name: Security scan
      run: |
        # Scan dependencies
        pip-audit -r requirements.txt
        
        # Scan Docker image
        trivy image ai-agent:${{ github.sha }}
        
        # Run SAST
        bandit -r app/ -f json -o bandit-report.json
    ```

---

## 📊 4. Observability & Monitoring

### 4.1 Prometheus Metrics

!!! example "Custom Metrics"
    ```python
    from prometheus_client import Counter, Histogram, Gauge
    import time
    
    # Define metrics
    REQUEST_COUNT = Counter('ai_agent_requests_total', 'Total requests', ['endpoint'])
    REQUEST_DURATION = Histogram('ai_agent_request_duration_seconds', 'Request duration')
    ACTIVE_AGENTS = Gauge('ai_agent_active_agents', 'Number of active agents')
    
    class MetricsMiddleware:
        def __init__(self, app):
            self.app = app
        
        def __call__(self, environ, start_response):
            start_time = time.time()
            
            def custom_start_response(status, headers, exc_info=None):
                duration = time.time() - start_time
                REQUEST_DURATION.observe(duration)
                REQUEST_COUNT.labels(endpoint=environ.get('PATH_INFO')).inc()
                return start_response(status, headers, exc_info)
            
            return self.app(environ, custom_start_response)
    ```

### 4.2 Distributed Tracing

!!! example "OpenTelemetry Integration"
    ```python
    from opentelemetry import trace
    from opentelemetry.exporter.jaeger.thrift import JaegerExporter
    from opentelemetry.sdk.trace import TracerProvider
    from opentelemetry.sdk.trace.export import BatchSpanProcessor
    
    # Set up tracing
    trace.set_tracer_provider(TracerProvider())
    tracer = trace.get_tracer(__name__)
    
    jaeger_exporter = JaegerExporter(
        agent_host_name="localhost",
        agent_port=6831,
    )
    span_processor = BatchSpanProcessor(jaeger_exporter)
    trace.get_tracer_provider().add_span_processor(span_processor)
    
    def agent_function():
        with tracer.start_as_current_span("agent_execution") as span:
            span.set_attribute("agent.type", "calculator")
            span.set_attribute("user.query", "2+2")
            
            # Agent logic here
            result = "4"
            
            span.set_attribute("agent.result", result)
            return result
    ```

---

## 🚨 5. Incident Response

### 5.1 Runbook Templates

!!! example "Incident Response Runbook"
    ```markdown
    # AI Agent Incident Response Runbook
    
    ## Incident Severity Levels
    
    - **P0 (Critical)**: Complete service outage, data loss
    - **P1 (High)**: Major functionality broken, high error rates
    - **P2 (Medium)**: Minor functionality issues, degraded performance
    - **P3 (Low)**: Cosmetic issues, minor bugs
    
    ## Response Steps
    
    1. **Detection**: Monitor alerts and metrics
    2. **Assessment**: Determine severity and impact
    3. **Communication**: Notify stakeholders
    4. **Investigation**: Root cause analysis
    5. **Resolution**: Implement fix
    6. **Recovery**: Verify service restoration
    7. **Post-mortem**: Document lessons learned
    
    ## Common Scenarios
    
    ### High Latency
    - Check resource utilization
    - Review recent deployments
    - Scale up if necessary
    - Check external API dependencies
    
    ### High Error Rate
    - Check logs for error patterns
    - Verify API key validity
    - Check rate limits
    - Review recent code changes
    ```

### 5.2 Alerting Configuration

!!! example "Prometheus Alert Rules"
    ```yaml
    groups:
    - name: ai-agent-alerts
      rules:
      - alert: HighErrorRate
        expr: rate(ai_agent_errors_total[5m]) > 0.1
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value }} errors per second"
      
      - alert: HighLatency
        expr: histogram_quantile(0.95, rate(ai_agent_request_duration_seconds_bucket[5m])) > 5
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "High latency detected"
          description: "95th percentile latency is {{ $value }} seconds"
      
      - alert: ServiceDown
        expr: up{job="ai-agent"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "AI Agent service is down"
          description: "Service has been down for more than 1 minute"
    ```

---

## 💻 6. Mini-Project: Production-Ready Agent Deployment

!!! success "Production Deployment Challenge"
    **Deploy a production-ready AI agent with full observability:**

    1. **Containerize**: Create a Docker image for your agent
    2. **Deploy**: Use Kubernetes or serverless platform
    3. **Monitor**: Set up Prometheus metrics and Grafana dashboards
    4. **Alert**: Configure alerting for critical issues
    5. **Document**: Create runbooks and deployment guides

!!! example "Project Structure"
    ```
    production-agent/
    ├── app/
    │   ├── main.py
    │   ├── agent.py
    │   └── metrics.py
    ├── k8s/
    │   ├── deployment.yaml
    │   ├── service.yaml
    │   └── ingress.yaml
    ├── monitoring/
    │   ├── prometheus-rules.yaml
    │   ├── grafana-dashboard.json
    │   └── alertmanager-config.yaml
    ├── ci/
    │   └── github-actions.yml
    ├── docs/
    │   ├── runbook.md
    │   └── deployment-guide.md
    └── Dockerfile
    ```

---

## ❓ 7. Self-Check Questions

!!! question "Knowledge Check"
    1. What are the benefits of containerizing AI agents?
    2. How would you handle secrets management in a production environment?
    3. What metrics are most important for monitoring AI agents?
    4. How do you implement proper health checks for agent services?
    5. What's your incident response process for agent failures?

---

## 🧭 Navigation

!!! success "Next Up"
    **[Lesson 4-2: Containerization & Serverless →](lesson-2.md)**

    Learn about advanced containerization strategies and serverless architectures for AI agents.
