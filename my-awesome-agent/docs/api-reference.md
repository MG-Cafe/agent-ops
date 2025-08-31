# API Reference

This document provides a comprehensive reference for all functions, classes, and tools available in the my-awesome-agent project.

## Core Agent

### `root_agent`

The main agent instance that handles user interactions and coordinates tool usage.

**Configuration:**
- **Name:** `root_agent`
- **Model:** `gemini-2.5-flash`
- **Instruction:** "You are a helpful AI assistant designed to provide accurate and useful information."
- **Tools:** `[get_weather, get_current_time]`

## Tools

### Weather Tool

#### `get_weather(query: str) -> str`

Simulates a web search to get weather information for a specified location.

**Parameters:**
- `query` (str): A string containing the location to get weather information for.

**Returns:**
- `str`: A string with the simulated weather information for the queried location.

**Example:**
```python
result = get_weather("San Francisco")
print(result)  # "It's 60 degrees and foggy."
```

**Behavior:**
- Returns "It's 60 degrees and foggy." for San Francisco (including variations like "sf")
- Returns "It's 90 degrees and sunny." for all other locations

### Time Tool

#### `get_current_time(query: str) -> str`

Simulates getting the current time for a specified city.

**Parameters:**
- `query` (str): The name of the city to get the current time for.

**Returns:**
- `str`: A string with the current time information, or an error message if timezone information is unavailable.

**Example:**
```python
result = get_current_time("San Francisco")
print(result)  # "The current time for query San Francisco is 2024-01-15 14:30:45 PST-0800"
```

**Behavior:**
- Supports San Francisco timezone (America/Los_Angeles)
- Returns error message for unsupported cities
- Uses `zoneinfo.ZoneInfo` for accurate timezone handling

## Utility Modules

### Tracing (`app/utils/tracing.py`)

Provides OpenTelemetry integration for comprehensive observability.

**Features:**
- Google Cloud Trace integration
- BigQuery event logging
- Custom span handling for large payloads
- GCS linking for oversized traces

### Google Cloud Storage (`app/utils/gcs.py`)

Handles Google Cloud Storage operations for the application.

**Features:**
- Blob upload and download
- Bucket management
- Authentication handling

### Type Definitions (`app/utils/typing.py`)

Contains custom type definitions and type hints used throughout the application.

## Environment Configuration

### Required Environment Variables

- `GOOGLE_CLOUD_PROJECT`: Google Cloud project ID
- `GOOGLE_CLOUD_LOCATION`: Google Cloud region (default: "global")
- `GOOGLE_GENAI_USE_VERTEXAI`: Set to "True" to use Vertex AI

### Authentication

The application uses Google Cloud default authentication. Ensure you have authenticated using:

```bash
gcloud auth application-default login
```

## Error Handling

### Common Error Patterns

1. **Authentication Errors**: Ensure proper Google Cloud authentication
2. **Model Access Errors**: Verify Vertex AI API is enabled and permissions are correct
3. **Tool Execution Errors**: Check tool parameters and network connectivity

### Best Practices

- Always handle tool execution errors gracefully
- Provide meaningful error messages to users
- Log errors for debugging purposes
- Implement retry logic for transient failures

## Integration Examples

### Using the Agent Programmatically

```python
import asyncio
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from app.agent import root_agent
from google.genai import types as genai_types

async def query_agent(message: str):
    session_service = InMemorySessionService()
    await session_service.create_session(
        app_name="my-awesome-agent", 
        user_id="test_user", 
        session_id="test_session"
    )
    
    runner = Runner(
        agent=root_agent, 
        app_name="my-awesome-agent", 
        session_service=session_service
    )
    
    async for event in runner.run_async(
        user_id="test_user",
        session_id="test_session",
        new_message=genai_types.Content(
            role="user", 
            parts=[genai_types.Part.from_text(text=message)]
        ),
    ):
        if event.is_final_response():
            return event.content.parts[0].text

# Usage
result = asyncio.run(query_agent("What's the weather in San Francisco?"))
print(result)
```

### Custom Tool Development

To add new tools to the agent:

1. **Define the tool function:**
```python
def my_custom_tool(parameter: str) -> dict:
    """Description of what this tool does.
    
    Args:
        parameter: Description of the parameter.
        
    Returns:
        dict: Description of the return value.
    """
    # Tool implementation
    return {"result": "some_value"}
```

2. **Add to agent configuration:**
```python
root_agent = Agent(
    name="root_agent",
    model="gemini-2.5-flash",
    instruction="Your instruction here",
    tools=[get_weather, get_current_time, my_custom_tool],
)
```

## Version Information

- **ADK Version:** ~1.5.0
- **Python Version:** >=3.10,<3.13
- **Google Cloud AI Platform:** ~1.101.0

For the most up-to-date version information, see `pyproject.toml`.
