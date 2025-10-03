# Google ADK Implementation Guide

## Overview

Google Agent Development Kit (ADK) is a code-first Python framework for building, evaluating, and deploying AI agents. This guide provides comprehensive implementation patterns and best practices.

## Core Concepts

### 1. Agent Types

#### LlmAgent

The foundational agent type that wraps an LLM with instructions, tools, and configuration.

**Use When:**

- Building straightforward agents with clear instructions
- Agent needs tools but no complex orchestration
- Standard single-agent architecture

**Key Parameters:**

- `name`: Agent identifier
- `model`: Gemini model (e.g., "gemini-2.0-flash")
- `instruction`: System prompt defining agent behavior
- `description`: Brief description of agent purpose
- `tools`: List of callable functions/tools
- `input_schema`: Pydantic model for structured input
- `output_schema`: Pydantic model for structured output
- `output_key`: Key to store agent output in session.state
- `sub_agents`: Child agents for delegation
- `generate_content_config`: LLM generation parameters

#### BaseAgent

Abstract base class for creating custom agents with arbitrary orchestration logic.

**Use When:**

- Need conditional logic in agent workflow
- Dynamic agent selection based on runtime conditions
- Complex state management requirements
- Workflow doesn't fit standard patterns

**Implementation Requirements:**

- Inherit from `BaseAgent`
- Implement `_run_async_impl` async generator method
- Declare sub-agents as Pydantic fields
- Set `model_config = {"arbitrary_types_allowed": True}`
- Pass sub_agents list to `super().__init__()`

#### Workflow Agents

**SequentialAgent**: Execute sub-agents in order

- Use for linear pipelines
- Each agent runs after previous completes
- State flows through agents via session.state

**LoopAgent**: Iterate sub-agents with max iterations

- Use for refinement loops (critic → reviser)
- Specify max_iterations to prevent infinite loops
- Agents run sequentially per iteration

**ParallelAgent**: Execute sub-agents concurrently

- Use when agents don't depend on each other
- Results collected after all complete
- Good for parallel analysis (sentiment + entities + summary)

---

### 2. Tools System

#### Tool Basics

Tools extend agent capabilities beyond LLM knowledge. The LLM decides when to call tools based on:

- Tool function name
- Tool docstring
- Parameter descriptions
- User query context

#### Creating Tools

**Essential Requirements:**

```python
def tool_name(param: type, optional_param: type = default) -> return_type:
    """
    Clear description the LLM uses to decide when to call this tool.

    Include detailed explanation of what the tool does, when to use it,
    and what it returns.

    Args:
        param: Detailed parameter description
        optional_param: Optional parameter with default

    Returns:
        Description of return value
    """
    # Implementation with error handling
    try:
        result = # ... your logic
        return result
    except Exception as e:
        return f"Error: {str(e)}"
```

**Automatic Wrapping:**
ADK automatically wraps Python functions as `FunctionTool` instances. Just pass the function directly to the agent's `tools` list.

#### Built-in Tools

ADK provides pre-built tools:

- `google_search` - Web search capability
- `code_executor` - Execute Python code
- More in ADK docs

Import and use directly:

```python
from google.adk.tools import google_search

agent = LlmAgent(
    name="researcher",
    tools=[google_search, custom_tool]
)
```

---

### 3. Session and State Management

#### Session Basics

Sessions maintain conversation context and shared state.

**InMemorySessionService**: For development and testing

```python
from google.adk.sessions import InMemorySessionService

session_service = InMemorySessionService()
session_service.create_session(
    app_name="my_app",
    user_id="user_123",
    session_id="session_456"
)
```

**Persistent Sessions**: For production (database-backed)

#### State Dictionary

`ctx.session.state` is a dictionary for sharing data between agents.

**Writing to State:**

```python
# Via output_key in LlmAgent
agent = LlmAgent(
    name="writer",
    instruction="Generate content",
    output_key="content"  # Auto-saves response to state["content"]
)

# Manually in custom agent
ctx.session.state["my_key"] = "my_value"
```

**Reading from State:**

```python
# In agent instruction
instruction = """
Review the content stored in session state under key 'content'.
Provide feedback on the content.
"""

# In custom agent code
value = ctx.session.state.get("my_key")
if value == "some_condition":
    # conditional logic
```

**Best Practices:**

- Use descriptive state keys
- Document all state keys used by agent
- Check for key existence with `.get()` to avoid KeyError
- Clear state when starting new workflows

---

### 4. Structured I/O with Schemas

#### Input Schema

Defines expected input structure. When set, user message must be JSON matching schema.

```python
from pydantic import BaseModel, Field

class QueryInput(BaseModel):
    query: str = Field(description="User's question")
    context: str = Field(description="Additional context")
    max_results: int = Field(default=5, description="Maximum results")

agent = LlmAgent(
    name="search_agent",
    input_schema=QueryInput,
    instruction="Parse the JSON input and use query field to search..."
)
```

#### Output Schema

Enforces structured JSON output. **CRITICAL:** Agents with `output_schema` CANNOT use tools.

```python
class SearchOutput(BaseModel):
    results: list[str] = Field(description="Search results")
    count: int = Field(description="Number of results")
    summary: str = Field(description="Summary of findings")

agent = LlmAgent(
    name="structured_search",
    output_schema=SearchOutput,
    output_key="search_results",
    instruction="Respond ONLY with JSON matching the schema..."
    # DO NOT add tools parameter - it won't work with output_schema
)
```

#### When to Use Schemas

**Use input_schema when:**

- Receiving data from another system
- Need structured workflow inputs
- Validating input format

**Use output_schema when:**

- Downstream systems need structured data
- Integrating with APIs/databases
- Ensuring consistent output format
- Agent is part of automated pipeline

**Don't use output_schema when:**

- Agent needs to call tools
- Natural language responses preferred
- User-facing conversational agent

---

### 5. Multi-Agent Patterns

#### Parent-Child Hierarchy

```python
# Define specialists
specialist_a = LlmAgent(
    name="specialist_a",
    instruction="Handle task A",
    output_key="result_a"
)

specialist_b = LlmAgent(
    name="specialist_b",
    instruction="Handle task B using result_a from state",
    output_key="result_b"
)

# Create coordinator
coordinator = LlmAgent(
    name="coordinator",
    instruction="Route tasks to appropriate specialist",
    sub_agents=[specialist_a, specialist_b]
)
```

**Data Flow:**

1. Coordinator receives user request
2. Delegates to specialist_a
3. specialist_a writes to `state["result_a"]`
4. specialist_b reads from `state["result_a"]`
5. specialist_b writes to `state["result_b"]`
6. Coordinator synthesizes final response

#### Sequential Pipeline

```python
from google.adk.agents import SequentialAgent

pipeline = SequentialAgent(
    name="content_pipeline",
    sub_agents=[
        generator_agent,   # Generates content
        editor_agent,      # Edits content
        formatter_agent    # Formats content
    ]
)
```

Data flows through `session.state` using `output_key`.

#### Refinement Loop

```python
from google.adk.agents import LoopAgent

refinement = LoopAgent(
    name="refiner",
    sub_agents=[
        critic_agent,   # Reviews content, suggests improvements
        reviser_agent   # Applies improvements
    ],
    max_iterations=3
)
```

Iterates up to `max_iterations`, refining output each cycle.

#### Parallel Analysis

```python
from google.adk.agents import ParallelAgent

analyzer = ParallelAgent(
    name="multi_analyzer",
    sub_agents=[
        sentiment_agent,
        entity_agent,
        summary_agent
    ]
)
```

All agents run concurrently, results collected when all finish.

---

### 6. Custom Agent Orchestration

#### When to Create Custom Agents

- Conditional logic based on runtime results
- Dynamic agent selection
- Complex state management
- Workflow patterns not covered by Sequential/Loop/Parallel
- Integration with external systems mid-workflow

#### Implementation Pattern

```python
from google.adk.agents import BaseAgent, LlmAgent
from google.adk.sessions import InvocationContext
from typing import AsyncGenerator
from google.adk.events import Event

class ConditionalWorkflow(BaseAgent):
    """Agent with conditional branching logic."""

    # REQUIRED: Pydantic field declarations
    analyzer: LlmAgent
    path_a_agent: LlmAgent
    path_b_agent: LlmAgent
    model_config = {"arbitrary_types_allowed": True}

    def __init__(
        self,
        name: str,
        analyzer: LlmAgent,
        path_a_agent: LlmAgent,
        path_b_agent: LlmAgent
    ):
        super().__init__(
            name=name,
            analyzer=analyzer,
            path_a_agent=path_a_agent,
            path_b_agent=path_b_agent,
            sub_agents=[analyzer, path_a_agent, path_b_agent]
        )

    async def _run_async_impl(
        self, ctx: InvocationContext
    ) -> AsyncGenerator[Event, None]:
        """Custom orchestration with conditional logic."""

        # Step 1: Analyze request
        async for event in self.analyzer.run_async(ctx):
            yield event

        # Step 2: Conditional routing
        analysis_result = ctx.session.state.get("analysis_category")

        if analysis_result == "category_a":
            async for event in self.path_a_agent.run_async(ctx):
                yield event
        elif analysis_result == "category_b":
            async for event in self.path_b_agent.run_async(ctx):
                yield event
        else:
            # Error handling
            ctx.session.state["error"] = "Unknown category"
```

**Key Points:**

- Must be `async def` returning `AsyncGenerator[Event, None]`
- Yield all events from sub-agents
- Use `ctx.session.state` for decisions and data
- Declare all sub-agents as Pydantic fields
- Set `model_config` for arbitrary types
- Pass `sub_agents` list to super().**init**()

---

### 7. Configuration and Tuning

#### Generation Config

```python
from google.genai import types

agent = LlmAgent(
    name="tuned_agent",
    model="gemini-2.0-flash",
    generate_content_config=types.GenerateContentConfig(
        temperature=0.2,        # 0.0 = deterministic, 1.0+ = creative
        max_output_tokens=500,  # Limit response length
        top_p=0.9,             # Nucleus sampling
        top_k=40,              # Top-k sampling
        # safety_settings=[...]  # Content safety filters
    )
)
```

**Temperature Guide:**

- 0.0-0.3: Factual, consistent, deterministic
- 0.4-0.7: Balanced
- 0.8-1.0+: Creative, varied, unpredictable

#### Context Control

```python
agent = LlmAgent(
    name="stateless_agent",
    include_contents='none'  # Don't send conversation history
)
```

Use `include_contents='none'` for:

- Stateless operations
- Sensitive data processing
- Enforcing specific context

---

### 8. Running Agents

#### Runner Setup

```python
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.genai import types

# Create session service
session_service = InMemorySessionService()
session_service.create_session(
    app_name="my_app",
    user_id="user_123",
    session_id="session_456"
)

# Create runner
runner = Runner(
    agent=my_agent,
    app_name="my_app",
    session_service=session_service
)

# Run agent
async def execute_agent(user_query: str):
    message = types.Content(
        role='user',
        parts=[types.Part(text=user_query)]
    )

    async for event in runner.run_async(
        user_id="user_123",
        session_id="session_456",
        new_message=message
    ):
        # Process events
        if event.is_final_response():
            response = event.content.parts[0].text
            print(response)
            return response
```

#### Event Types

Events emitted during agent execution:

- Agent start/end events
- Tool call events
- Content generation events
- Final response event

Use `event.is_final_response()` to get agent's final answer.

---

### 9. Testing and Evaluation

#### Unit Testing Tools

```python
def test_my_tool():
    result = my_tool("input")
    assert result == expected_output
```

#### Integration Testing Agents

```python
@pytest.mark.asyncio
async def test_agent():
    # Setup runner and session
    # Send test message
    # Verify response
    pass
```

#### Evaluation Sets

Create `.evalset.json` files:

```json
{
  "test_cases": [
    {
      "name": "Test case name",
      "input": "Test input",
      "expected_output_contains": ["expected", "phrases"]
    }
  ]
}
```

Run evaluations:

```bash
adk eval path/to/agent path/to/tests.evalset.json
```

---

### 10. Deployment

#### Development UI

```bash
adk serve path/to/agent
```

Launches local web UI for testing.

#### Production Deployment

**Options:**

1. **Cloud Run**: Containerize agent, deploy to Google Cloud Run
2. **Vertex AI Agent Engine**: Managed deployment on Vertex AI
3. **Custom**: Deploy anywhere Python runs

**Best Practices:**

- Externalize configuration (environment variables)
- Use persistent session storage
- Implement logging and monitoring
- Set up error tracking
- Configure rate limiting
- Secure API keys

---

## Design Principles

### 1. Code-First Approach

Everything is Python code - agents, tools, orchestration. No low-code builders, no JSON configs.

### 2. Modular Architecture

Compose complex systems from simple, focused agents. Each agent has one clear purpose.

### 3. Explicit State Management

State is explicit via `session.state` dictionary. No hidden global state.

### 4. Tool-Augmented LLMs

LLMs reason, tools act. Clear separation of concerns.

### 5. Testing First-Class

Built-in support for testing at all levels - unit, integration, evaluation.

---

## Common Patterns Reference

### Simple Q&A Agent

```python
LlmAgent(
    name="qa",
    model="gemini-2.0-flash",
    instruction="Answer questions concisely"
)
```

### Research Agent with Tools

```python
LlmAgent(
    name="researcher",
    tools=[google_search, scrape_url],
    instruction="Research topics using available tools"
)
```

### Structured Data Processor

```python
LlmAgent(
    name="processor",
    input_schema=InputModel,
    output_schema=OutputModel,
    output_key="processed_data"
)
```

### Multi-Agent Coordinator

```python
LlmAgent(
    name="coordinator",
    sub_agents=[specialist1, specialist2],
    instruction="Route requests to specialists"
)
```

### Sequential Pipeline

```python
SequentialAgent(
    name="pipeline",
    sub_agents=[step1, step2, step3]
)
```

### Refinement Loop

```python
LoopAgent(
    name="refiner",
    sub_agents=[critic, reviser],
    max_iterations=3
)
```

### Conditional Workflow

```python
class CustomAgent(BaseAgent):
    async def _run_async_impl(self, ctx):
        # Conditional logic
        if condition:
            # Path A
        else:
            # Path B
```

---

## Best Practices Summary

✅ **DO:**

- Use LlmAgent for straightforward tasks
- Provide clear, detailed instructions
- Use tools for external data/actions
- Leverage session.state for multi-agent communication
- Test tools independently
- Create evaluation sets
- Start simple, add complexity only when needed
- Document state keys and data flow

❌ **DON'T:**

- Mix output_schema with tools (mutually exclusive)
- Create custom agents unnecessarily
- Ignore error handling in tools
- Skip testing
- Hardcode credentials
- Overcomplicate agent hierarchies
- Forget to yield events in custom agents

---

## Resources

- **Official Docs**: https://google.github.io/adk-docs
- **GitHub**: https://github.com/google/adk-python
- **Samples**: https://github.com/google/adk-samples
- **ADK Web UI**: https://github.com/google/adk-web

---

## Quick Reference

### Installation

```bash
pip install google-adk
```

### Basic Agent

```python
from google.adk.agents import LlmAgent
agent = LlmAgent(name="agent", model="gemini-2.0-flash", instruction="...")
```

### With Tools

```python
agent = LlmAgent(tools=[my_tool], ...)
```

### Multi-Agent

```python
agent = LlmAgent(sub_agents=[agent1, agent2], ...)
```

### Custom Agent

```python
class MyAgent(BaseAgent):
    async def _run_async_impl(self, ctx): ...
```

### Run Agent

```python
runner = Runner(agent=agent, app_name="app", session_service=service)
async for event in runner.run_async(...): ...
```
