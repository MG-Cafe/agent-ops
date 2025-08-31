# Developer Guide

This guide provides comprehensive instructions for developing, testing, and deploying the my-awesome-agent application.

## Getting Started

### Prerequisites

Before you begin development, ensure you have the following tools installed:

- **uv**: Python package manager - [Install Guide](https://docs.astral.sh/uv/getting-started/installation/)
- **Google Cloud SDK**: For GCP services - [Install Guide](https://cloud.google.com/sdk/docs/install)
- **Terraform**: For infrastructure deployment - [Install Guide](https://developer.hashicorp.com/terraform/downloads)
- **make**: Build automation tool (pre-installed on most Unix-based systems)
- **Git**: Version control system

### Initial Setup

1. **Clone the repository:**
   ```bash
   git clone <repository-url>
   cd my-awesome-agent
   ```

2. **Install dependencies:**
   ```bash
   make install
   ```

3. **Set up authentication:**
   ```bash
   gcloud auth application-default login
   ```

4. **Configure environment variables:**
   Create a `.env` file in the project root:
   ```bash
   GOOGLE_CLOUD_PROJECT=your-project-id
   GOOGLE_CLOUD_LOCATION=us-central1
   GOOGLE_GENAI_USE_VERTEXAI=True
   ```

## Development Workflow

### 1. Code Development

#### Agent Logic
The main agent configuration is in `app/agent.py`. This file contains:
- Agent initialization
- Tool definitions
- Environment setup

#### Tool Development
When creating new tools:

1. **Follow the function signature pattern:**
   ```python
   def my_tool(parameter: str) -> str:
       """Clear description of the tool's purpose.
       
       Args:
           parameter: Description of what this parameter does.
           
       Returns:
           Description of what the function returns.
       """
       # Implementation
       return result
   ```

2. **Add comprehensive docstrings** - these are used by the LLM to understand when and how to use your tool

3. **Handle errors gracefully** - return meaningful error messages

4. **Keep tools focused** - each tool should have a single, clear purpose

#### Code Style
This project uses several linting tools to maintain code quality:

- **Ruff**: Fast Python linter and formatter
- **MyPy**: Static type checking
- **Codespell**: Spelling checker

Run all linting checks:
```bash
make lint
```

### 2. Testing

#### Unit Tests
Unit tests are located in `tests/unit/`. Run them with:
```bash
make test
```

#### Integration Tests
Integration tests are in `tests/integration/`. These test the agent's behavior end-to-end.

#### Manual Testing
For interactive testing during development:

```bash
make playground
```

This launches a Streamlit interface for testing your agent locally.

#### Programmatic Testing
Create test scripts for automated validation:

```python
# test_script.py
import asyncio
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from app.agent import root_agent
from google.genai import types as genai_types

async def test_agent():
    session_service = InMemorySessionService()
    await session_service.create_session(
        app_name="test", user_id="test_user", session_id="test_session"
    )
    
    runner = Runner(agent=root_agent, app_name="test", session_service=session_service)
    
    async for event in runner.run_async(
        user_id="test_user",
        session_id="test_session",
        new_message=genai_types.Content(
            role="user", 
            parts=[genai_types.Part.from_text(text="What's the weather in SF?")]
        ),
    ):
        if event.is_final_response():
            print(event.content.parts[0].text)

if __name__ == "__main__":
    asyncio.run(test_agent())
```

Run with:
```bash
uv run python test_script.py
```

#### Load Testing
For performance testing, see `tests/load_test/README.md` for detailed instructions on running Locust-based load tests.

### 3. Evaluation

The project includes evaluation capabilities using the ADK evaluation framework:

1. **Create evaluation datasets** in the `tests/` directory with `.evalset.json` extension
2. **Run evaluations** using the ADK CLI:
   ```bash
   adk eval app/ tests/my-agent.evalset.json
   ```

### 4. Development Environment Deployment

For testing in a cloud environment:

1. **Configure your development project:**
   ```bash
   gcloud config set project your-dev-project-id
   ```

2. **Deploy infrastructure:**
   ```bash
   make setup-dev-env
   ```

3. **Deploy the agent:**
   ```bash
   make backend
   ```

## Deployment

### Local Development
- Use `make playground` for interactive testing
- Use `adk web` for advanced debugging with trace visualization
- Use `adk run` for command-line interaction

### Development Environment
- Deploy to a personal GCP project for integration testing
- Uses the same infrastructure as production but with dev-specific settings

### Production Deployment
For production deployment, this project supports two main approaches:

#### Option 1: Automated CI/CD Setup
Use the Agent Starter Pack CLI for full automation:
```bash
uvx agent-starter-pack setup-cicd \
  --staging-project your-staging-project \
  --prod-project your-prod-project \
  --repository-name your-repo-name \
  --repository-owner your-github-username \
  --git-provider github \
  --auto-approve
```

#### Option 2: Manual Deployment
Follow the instructions in `deployment/README.md` for manual infrastructure setup.

## Debugging

### Common Issues

1. **Authentication Problems:**
   - Ensure `gcloud auth application-default login` has been run
   - Check that the service account has necessary permissions
   - Verify `GOOGLE_CLOUD_PROJECT` is set correctly

2. **Model Access Issues:**
   - Ensure Vertex AI API is enabled in your project
   - Check that your account has `Vertex AI User` role
   - Verify the model name is correct and available in your region

3. **Tool Execution Failures:**
   - Check tool parameters and types
   - Verify network connectivity for external API calls
   - Review tool docstrings for clarity

### Debugging Tools

1. **ADK Web UI:**
   ```bash
   adk web
   ```
   Provides visual debugging with trace information.

2. **Verbose Logging:**
   Enable detailed logging by setting environment variables:
   ```bash
   export LOGLEVEL=DEBUG
   ```

3. **Event Stream Inspection:**
   Add logging to your agent runs to see intermediate steps:
   ```python
   async for event in runner.run_async(...):
       print(f"Event: {event.author}, Content: {event.content}")
       if event.actions:
           print(f"Actions: {event.actions}")
   ```

## Best Practices

### Code Organization
- Keep agent logic in `app/agent.py`
- Place utility functions in `app/utils/`
- Store tests in appropriate `tests/` subdirectories
- Use clear, descriptive names for functions and variables

### Tool Design
- Write comprehensive docstrings
- Handle errors gracefully
- Keep tools focused on single responsibilities
- Use type hints for all parameters and return values

### Testing Strategy
- Write unit tests for individual tools
- Create integration tests for agent workflows
- Use evaluation datasets for regression testing
- Perform load testing before production deployment

### Security
- Never hardcode API keys or secrets
- Use environment variables for configuration
- Follow principle of least privilege for service accounts
- Regularly update dependencies

### Performance
- Monitor agent response times
- Optimize tool execution
- Use appropriate model sizes for your use case
- Implement caching where appropriate

## Troubleshooting

### Getting Help

If you encounter issues not covered in this guide:

1. Check the [ADK documentation](https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/overview)
2. Review the project's issue tracker
3. Consult the [Agent Starter Pack documentation](https://googlecloudplatform.github.io/agent-starter-pack/)
4. Search for solutions in the Google Cloud community forums

### Common Error Messages

**"Permission denied"**
- Solution: Check authentication and IAM permissions

**"Model not found"**
- Solution: Verify model name and ensure it's available in your region

**"Tool execution failed"**
- Solution: Check tool parameters and network connectivity

**"Session not found"**
- Solution: Ensure session service is properly initialized

For more specific troubleshooting, enable debug logging and review the detailed error messages.
