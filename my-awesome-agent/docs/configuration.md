# Configuration Reference

This document provides comprehensive information about configuring the my-awesome-agent application.

## Environment Variables

### Required Variables

| Variable | Description | Default | Example |
|----------|-------------|---------|--------|
| `GOOGLE_CLOUD_PROJECT` | Google Cloud project ID | None | `my-project-123` |
| `GOOGLE_CLOUD_LOCATION` | Google Cloud region | `global` | `us-central1` |
| `GOOGLE_GENAI_USE_VERTEXAI` | Enable Vertex AI usage | `True` | `True` |

### Optional Variables

| Variable | Description | Default | Example |
|----------|-------------|---------|--------|
| `LOGLEVEL` | Logging level | `INFO` | `DEBUG` |
| `OPENTELEMETRY_ENDPOINT` | OpenTelemetry collector endpoint | None | `http://localhost:4317` |
| `GOOGLE_API_KEY` | Google AI Studio API key (alternative to Vertex AI) | None | `AIza...` |

### Setting Environment Variables

#### Local Development (.env file)
Create a `.env` file in the project root:

```bash
GOOGLE_CLOUD_PROJECT=your-project-id
GOOGLE_CLOUD_LOCATION=us-central1
GOOGLE_GENAI_USE_VERTEXAI=True
LOGLEVEL=DEBUG
```

#### Production (Cloud Run)
Set environment variables in your deployment configuration:

```yaml
env:
  - name: GOOGLE_CLOUD_PROJECT
    value: "your-project-id"
  - name: GOOGLE_CLOUD_LOCATION
    value: "us-central1"
  - name: GOOGLE_GENAI_USE_VERTEXAI
    value: "True"
```

## Model Configuration

### Available Models

The agent supports various Gemini model versions:

| Model | Use Case | Context Window | Speed |
|-------|----------|----------------|-------|
| `gemini-2.5-flash` | Fast responses, simple tasks | 1M tokens | Fast |
| `gemini-2.5-pro` | Complex reasoning, detailed analysis | 2M tokens | Moderate |
| `gemini-1.5-flash` | Balanced performance | 1M tokens | Fast |
| `gemini-1.5-pro` | Advanced reasoning | 2M tokens | Slower |

### Model Parameters

You can customize model behavior by modifying the agent configuration:

```python
from google.genai import types as genai_types

gen_config = genai_types.GenerateContentConfig(
    temperature=0.7,          # Creativity level (0.0-1.0)
    top_p=0.9,               # Nucleus sampling
    top_k=40,                # Top-k sampling
    max_output_tokens=1024,   # Maximum response length
    stop_sequences=["END"]    # Stop generation at these sequences
)

root_agent = Agent(
    name="root_agent",
    model="gemini-2.5-flash",
    generate_content_config=gen_config,
    instruction="Your custom instruction here",
    tools=[get_weather, get_current_time],
)
```

### Safety Settings

Configure content safety filters:

```python
from google.genai import types as genai_types

safety_settings = [
    genai_types.SafetySetting(
        category=genai_types.HarmCategory.HARM_CATEGORY_HARASSMENT,
        threshold=genai_types.HarmBlockThreshold.BLOCK_MEDIUM_AND_ABOVE,
    ),
    genai_types.SafetySetting(
        category=genai_types.HarmCategory.HARM_CATEGORY_HATE_SPEECH,
        threshold=genai_types.HarmBlockThreshold.BLOCK_LOW_AND_ABOVE,
    ),
]

root_agent = Agent(
    # ... other config ...
    safety_settings=safety_settings,
)
```

## Tool Configuration

### Built-in Tools

The agent comes with two pre-configured tools:

#### Weather Tool Configuration
```python
def get_weather(query: str) -> str:
    """Simulates a web search. Use it get information on weather.

    Args:
        query: A string containing the location to get weather information for.

    Returns:
        A string with the simulated weather information for the queried location.
    """
    # Configuration options:
    SUPPORTED_LOCATIONS = {
        "san francisco": "It's 60 degrees and foggy.",
        "sf": "It's 60 degrees and foggy.",
    }
    DEFAULT_WEATHER = "It's 90 degrees and sunny."
    
    return SUPPORTED_LOCATIONS.get(query.lower(), DEFAULT_WEATHER)
```

#### Time Tool Configuration
```python
def get_current_time(query: str) -> str:
    """Simulates getting the current time for a city.

    Args:
        city: The name of the city to get the current time for.

    Returns:
        A string with the current time information.
    """
    # Configuration options:
    SUPPORTED_TIMEZONES = {
        "san francisco": "America/Los_Angeles",
        "sf": "America/Los_Angeles",
    }
    
    # Add more timezones as needed
    # "new york": "America/New_York",
    # "london": "Europe/London",
```

### Adding Custom Tools

To add new tools to your agent:

1. **Define the tool function:**
```python
def my_custom_tool(parameter: str) -> dict:
    """Description of what this tool does.
    
    Args:
        parameter: Description of the parameter.
        
    Returns:
        dict: Description of the return value with 'status' and 'data' keys.
    """
    try:
        # Tool logic here
        result = process_parameter(parameter)
        return {"status": "success", "data": result}
    except Exception as e:
        return {"status": "error", "message": str(e)}
```

2. **Add to agent tools list:**
```python
root_agent = Agent(
    name="root_agent",
    model="gemini-2.5-flash",
    instruction="Your instruction here",
    tools=[get_weather, get_current_time, my_custom_tool],  # Add your tool
)
```

### Tool Best Practices

- **Clear docstrings**: The LLM uses docstrings to understand when and how to use tools
- **Type hints**: Use proper type annotations for all parameters
- **Error handling**: Return structured error responses
- **Single responsibility**: Each tool should do one thing well
- **Consistent returns**: Use consistent return formats across tools

## Deployment Configuration

### Development Environment

Edit `deployment/terraform/dev/vars/env.tfvars`:

```hcl
project_name = "my-awesome-agent"
region = "us-central1"
staging_project_id = "your-dev-project"

# Optional configurations
enable_apis = true
create_service_accounts = true
```

### Production Environment

Edit `deployment/terraform/vars/env.tfvars`:

```hcl
project_name = "my-awesome-agent"
prod_project_id = "your-prod-project"
staging_project_id = "your-staging-project"
cicd_runner_project_id = "your-cicd-project"
region = "us-central1"
host_connection_name = "your-github-connection"
repository_name = "your-repo-name"

# Security configurations
enable_iap = true
allow_unauthenticated = false

# Monitoring configurations
enable_monitoring = true
log_retention_days = 30
```

### Cloud Run Configuration

For Cloud Run deployments, you can customize:

```yaml
# In deployment/cd/staging.yaml or deploy-to-prod.yaml
env:
  - name: GOOGLE_CLOUD_PROJECT
    value: ${PROJECT_ID}
  - name: GOOGLE_CLOUD_LOCATION
    value: ${REGION}
  - name: GOOGLE_GENAI_USE_VERTEXAI
    value: "True"
  
# Resource limits
resources:
  limits:
    cpu: "2"
    memory: "4Gi"
  requests:
    cpu: "1"
    memory: "2Gi"

# Scaling
metadata:
  annotations:
    run.googleapis.com/execution-environment: gen2
    autoscaling.knative.dev/minScale: "0"
    autoscaling.knative.dev/maxScale: "100"
```

## Authentication and Security

### Google Cloud Authentication

#### Local Development
```bash
gcloud auth application-default login
```

#### Production (Service Account)
The application automatically uses the service account attached to the compute resource.

### Required IAM Roles

For the service account running the agent:

- `Vertex AI User`: For Gemini model access
- `Storage Object Viewer`: For reading configuration files
- `Logging Writer`: For application logging
- `Trace Agent`: For OpenTelemetry tracing
- `BigQuery Data Editor`: For event logging (if enabled)

### API Access Configuration

Enable required APIs in your project:

```bash
gcloud services enable \
    aiplatform.googleapis.com \
    logging.googleapis.com \
    cloudtrace.googleapis.com \
    storage.googleapis.com
```

## Monitoring and Observability

### OpenTelemetry Configuration

The application uses OpenTelemetry for comprehensive observability:

```python
# In app/utils/tracing.py
TRACE_CONFIG = {
    "service_name": "my-awesome-agent",
    "service_version": "1.0.0",
    "enable_gcs_fallback": True,  # For large traces
    "max_trace_size": 1024 * 1024,  # 1MB
}
```

### Log Configuration

Customize logging behavior:

```python
import logging

# Set log level via environment variable
log_level = os.getenv("LOGLEVEL", "INFO")
logging.basicConfig(level=getattr(logging, log_level.upper()))
```

### BigQuery Event Logging

Events are automatically logged to BigQuery for long-term analysis. Configure the dataset and table names in your Terraform variables:

```hcl
bigquery_dataset = "agent_events"
bigquery_table = "conversation_logs"
```

## Testing Configuration

### Unit Test Configuration

Pytest configuration is in `pyproject.toml`:

```toml
[tool.pytest.ini_options]
pythonpath = "."
asyncio_default_fixture_loop_scope = "function"
testpaths = ["tests"]
markers = [
    "integration: marks tests as integration tests",
    "unit: marks tests as unit tests",
    "slow: marks tests as slow running",
]
```

### Load Test Configuration

Locust configuration for load testing:

```python
# In tests/load_test/load_test.py
class AgentUser(HttpUser):
    wait_time = between(1, 5)  # Wait 1-5 seconds between requests
    host = "https://your-agent-endpoint.run.app"
    
    # Configure authentication token
    def on_start(self):
        self.auth_token = os.environ.get("_AUTH_TOKEN")
```

## Troubleshooting Configuration Issues

### Common Configuration Problems

1. **Model Access Denied**
   - Check `GOOGLE_GENAI_USE_VERTEXAI` is set to "True"
   - Verify Vertex AI API is enabled
   - Confirm service account has `Vertex AI User` role

2. **Authentication Failures**
   - Ensure `GOOGLE_CLOUD_PROJECT` matches your actual project
   - Check that application default credentials are set
   - Verify service account key is properly configured

3. **Tool Execution Errors**
   - Review tool docstrings for clarity
   - Check parameter types and names
   - Verify external API connectivity

4. **Deployment Issues**
   - Check Terraform variable files
   - Verify required APIs are enabled
   - Confirm IAM permissions are correctly set

### Configuration Validation

Create a configuration validation script:

```python
#!/usr/bin/env python3
"""Validate agent configuration."""

import os
import sys
from google.auth import default

def validate_config():
    """Validate all required configuration."""
    required_vars = [
        "GOOGLE_CLOUD_PROJECT",
        "GOOGLE_CLOUD_LOCATION", 
        "GOOGLE_GENAI_USE_VERTEXAI"
    ]
    
    missing = []
    for var in required_vars:
        if not os.getenv(var):
            missing.append(var)
    
    if missing:
        print(f"Missing required environment variables: {missing}")
        return False
        
    # Test authentication
    try:
        credentials, project = default()
        print(f"Authentication successful. Project: {project}")
    except Exception as e:
        print(f"Authentication failed: {e}")
        return False
        
    return True

if __name__ == "__main__":
    if validate_config():
        print("Configuration is valid!")
        sys.exit(0)
    else:
        print("Configuration validation failed!")
        sys.exit(1)
```

Run with:
```bash
uv run python validate_config.py
```
