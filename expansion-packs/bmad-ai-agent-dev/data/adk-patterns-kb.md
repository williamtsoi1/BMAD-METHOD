# ADK Patterns Knowledge Base

## Overview

This knowledge base contains proven patterns and anti-patterns for implementing AI agents with Google's Agent Development Kit (ADK).

---

## Agent Architecture Patterns

### Pattern: Single-Purpose Agent

**When:** Agent has one clear, focused responsibility

**Structure:**

```python
simple_agent = LlmAgent(
    name="summarizer",
    model="gemini-2.0-flash",
    instruction="Summarize text concisely in 2-3 sentences.",
    description="Text summarization agent"
)
```

**Benefits:**

- Easy to test
- Clear behavior
- Maintainable
- Reusable

**Use Cases:**

- Text summarization
- Translation
- Classification
- Data extraction

---

### Pattern: Tool-Augmented Agent

**When:** Agent needs real-time data or external actions

**Structure:**

```python
def get_weather(city: str) -> str:
    """Get current weather for a city."""
    # Implementation
    return weather_data

research_agent = LlmAgent(
    name="researcher",
    model="gemini-2.0-flash",
    instruction="Use available tools to research topics thoroughly.",
    tools=[google_search, get_weather, fetch_article],
    description="Research agent with web access"
)
```

**Benefits:**

- LLM reasoning + real-world data
- Extends beyond training knowledge
- Action capability

**Best Practices:**

- Limit tools to 5-10 per agent (too many confuses LLM)
- Clear tool docstrings (LLM reads these)
- Handle tool errors gracefully
- Test tools independently

---

### Pattern: Structured Data Agent

**When:** Agent must output/input structured data

**Structure:**

```python
from pydantic import BaseModel, Field

class ExtractionInput(BaseModel):
    text: str = Field(description="Text to extract from")
    fields: list[str] = Field(description="Fields to extract")

class ExtractionOutput(BaseModel):
    extracted_data: dict = Field(description="Extracted field:value pairs")
    confidence: float = Field(description="Confidence 0-1")

extractor = LlmAgent(
    name="data_extractor",
    model="gemini-2.0-flash",
    instruction="Extract requested fields from text. Output JSON only.",
    input_schema=ExtractionInput,
    output_schema=ExtractionOutput,
    output_key="extraction_result"
)
```

**Benefits:**

- Guaranteed structure
- Type safety
- Workflow integration
- Validation

**Constraints:**

- **CANNOT use tools with output_schema**
- Must format instruction to guide JSON generation

---

### Pattern: Coordinator-Specialist

**When:** Multiple specialized agents handle different aspects

**Structure:**

```python
# Specialists
email_specialist = LlmAgent(
    name="email_handler",
    instruction="Handle email-related tasks",
    tools=[send_email, search_emails],
    output_key="email_result"
)

calendar_specialist = LlmAgent(
    name="calendar_handler",
    instruction="Handle calendar tasks",
    tools=[get_events, create_event],
    output_key="calendar_result"
)

# Coordinator
assistant = LlmAgent(
    name="personal_assistant",
    model="gemini-2.0-flash",
    instruction="""
    You coordinate between email and calendar specialists.
    Route email requests to email_handler.
    Route calendar requests to calendar_handler.
    Synthesize results for user.
    """,
    sub_agents=[email_specialist, calendar_specialist]
)
```

**Benefits:**

- Separation of concerns
- Specialized expertise
- Easier testing
- Scalable architecture

**Data Flow:**

1. User → Coordinator
2. Coordinator → Appropriate specialist
3. Specialist → Session state (via output_key)
4. Coordinator → Synthesized response

---

## Workflow Patterns

### Pattern: Sequential Pipeline

**When:** Multi-stage processing where each step builds on previous

**Structure:**

```python
from google.adk.agents import SequentialAgent

content_pipeline = SequentialAgent(
    name="content_creator",
    sub_agents=[
        LlmAgent(name="ideator", instruction="Generate ideas", output_key="ideas"),
        LlmAgent(name="writer", instruction="Write based on ideas from state", output_key="draft"),
        LlmAgent(name="editor", instruction="Edit draft from state", output_key="final")
    ]
)
```

**Benefits:**

- Clear linear flow
- Each step focused
- Easy to debug
- State preserved

**Use Cases:**

- Content creation pipelines
- Data processing workflows
- Multi-step analysis

---

### Pattern: Critic-Reviser Loop

**When:** Iterative refinement needed

**Structure:**

```python
from google.adk.agents import LoopAgent

refiner = LoopAgent(
    name="content_refiner",
    sub_agents=[
        LlmAgent(
            name="critic",
            instruction="Review content in state['current_content']. Provide specific improvement suggestions.",
            output_key="critique"
        ),
        LlmAgent(
            name="reviser",
            instruction="Revise content based on critique. Output improved version.",
            output_key="current_content"  # Overwrites for next iteration
        )
    ],
    max_iterations=3
)
```

**Benefits:**

- Automatic refinement
- Controlled iterations
- Quality improvement

**Best Practices:**

- Set reasonable max_iterations (3-5 typically)
- Use same state key for content to enable iteration
- Add stopping conditions if possible

**Use Cases:**

- Content improvement
- Code review and fixes
- Iterative problem solving

---

### Pattern: Parallel Analysis

**When:** Multiple independent analyses needed simultaneously

**Structure:**

```python
from google.adk.agents import ParallelAgent

multi_analyzer = ParallelAgent(
    name="document_analyzer",
    sub_agents=[
        LlmAgent(name="sentiment", instruction="Analyze sentiment", output_key="sentiment"),
        LlmAgent(name="entities", instruction="Extract entities", output_key="entities"),
        LlmAgent(name="summary", instruction="Summarize document", output_key="summary"),
        LlmAgent(name="topics", instruction="Identify topics", output_key="topics")
    ]
)
```

**Benefits:**

- Concurrent execution
- Faster results
- Independent analyses

**Use Cases:**

- Document analysis
- Multi-faceted evaluation
- Comprehensive assessment

---

### Pattern: Conditional Workflow

**When:** Path selection based on runtime conditions

**Structure:**

```python
from google.adk.agents import BaseAgent
from typing import AsyncGenerator
from google.adk.events import Event

class ConditionalRouter(BaseAgent):
    classifier: LlmAgent
    simple_handler: LlmAgent
    complex_handler: LlmAgent
    model_config = {"arbitrary_types_allowed": True}

    def __init__(self, name: str, classifier, simple_handler, complex_handler):
        super().__init__(
            name=name,
            classifier=classifier,
            simple_handler=simple_handler,
            complex_handler=complex_handler,
            sub_agents=[classifier, simple_handler, complex_handler]
        )

    async def _run_async_impl(self, ctx) -> AsyncGenerator[Event, None]:
        # Classify request
        async for event in self.classifier.run_async(ctx):
            yield event

        # Route based on classification
        complexity = ctx.session.state.get("complexity_level")

        if complexity == "simple":
            async for event in self.simple_handler.run_async(ctx):
                yield event
        else:
            async for event in self.complex_handler.run_async(ctx):
                yield event
```

**Benefits:**

- Dynamic routing
- Optimized processing
- Conditional logic

**Use Cases:**

- Request routing by complexity
- Error recovery paths
- Context-dependent processing

---

## State Management Patterns

### Pattern: State-Based Communication

**When:** Multiple agents need to share data

**Structure:**

```python
# Agent A writes
generator = LlmAgent(
    name="generator",
    instruction="Generate content",
    output_key="raw_content"  # Writes to state["raw_content"]
)

# Agent B reads and writes
analyzer = LlmAgent(
    name="analyzer",
    instruction="Analyze content from state['raw_content']. Provide insights.",
    output_key="analysis"  # Writes to state["analysis"]
)

# Agent C reads both
summarizer = LlmAgent(
    name="summarizer",
    instruction="Read state['raw_content'] and state['analysis']. Create summary.",
    output_key="final_summary"
)
```

**Benefits:**

- Explicit data flow
- Decoupled agents
- Clear dependencies

**Best Practices:**

- Document all state keys
- Use descriptive key names
- Check key existence before reading
- Clear state between workflows

---

### Pattern: Accumulator State

**When:** Collecting results across iterations or parallel agents

**Structure:**

```python
class AccumulatorAgent(BaseAgent):
    collectors: list[LlmAgent]
    model_config = {"arbitrary_types_allowed": True}

    async def _run_async_impl(self, ctx) -> AsyncGenerator[Event, None]:
        # Initialize accumulator
        ctx.session.state["collected_items"] = []

        # Run collectors
        for collector in self.collectors:
            async for event in collector.run_async(ctx):
                yield event

            # Append result to accumulator
            result = ctx.session.state.get("current_result")
            ctx.session.state["collected_items"].append(result)

        # Final accumulated result available
        all_items = ctx.session.state["collected_items"]
```

**Use Cases:**

- Gathering research from multiple sources
- Collecting analysis results
- Building composite responses

---

## Tool Design Patterns

### Pattern: API Wrapper Tool

**When:** Agent needs external API access

**Structure:**

```python
import requests
import os

def fetch_news(topic: str, max_results: int = 5) -> str:
    """
    Fetches recent news articles about a topic.

    Args:
        topic: Topic to search for
        max_results: Maximum number of articles (default 5)

    Returns:
        Formatted string with article titles and summaries
    """
    try:
        response = requests.get(
            "https://api.news.com/search",
            params={"q": topic, "limit": max_results},
            headers={"Authorization": f"Bearer {os.getenv('NEWS_API_KEY')}"}
        )
        response.raise_for_status()

        articles = response.json()["articles"]
        formatted = "\n\n".join([
            f"**{a['title']}**\n{a['summary']}"
            for a in articles
        ])
        return formatted

    except requests.RequestException as e:
        return f"Error fetching news: {str(e)}"
    except KeyError as e:
        return f"Unexpected API response format: {str(e)}"
```

**Best Practices:**

- Use environment variables for API keys
- Handle all exception types
- Return string (not raw objects)
- Format for LLM readability
- Set timeouts
- Cache when appropriate

---

### Pattern: Database Query Tool

**When:** Agent needs data persistence/retrieval

**Structure:**

```python
import sqlite3
from typing import Optional

def query_customer(
    customer_id: Optional[str] = None,
    email: Optional[str] = None
) -> str:
    """
    Queries customer database by ID or email.

    Args:
        customer_id: Customer unique identifier
        email: Customer email address

    Returns:
        Customer information or error message
    """
    if not customer_id and not email:
        return "Error: Must provide either customer_id or email"

    try:
        conn = sqlite3.connect('customers.db')
        cursor = conn.cursor()

        if customer_id:
            cursor.execute(
                "SELECT name, email, status, created_date FROM customers WHERE id=?",
                (customer_id,)
            )
        else:
            cursor.execute(
                "SELECT name, email, status, created_date FROM customers WHERE email=?",
                (email,)
            )

        result = cursor.fetchone()
        conn.close()

        if result:
            name, email, status, created = result
            return f"""
Customer: {name}
Email: {email}
Status: {status}
Customer Since: {created}
"""
        else:
            return "Customer not found"

    except sqlite3.Error as e:
        return f"Database error: {str(e)}"
```

**Best Practices:**

- Use parameterized queries (prevent SQL injection)
- Close connections properly
- Format results for readability
- Handle not found cases
- Comprehensive error handling

---

### Pattern: Calculation Tool

**When:** Agent needs computational capabilities

**Structure:**

```python
from datetime import datetime, timedelta
from typing import Literal

def calculate_business_days(
    start_date: str,
    end_date: str,
    include_end: bool = True
) -> str:
    """
    Calculates number of business days between two dates.

    Business days exclude weekends (Saturday and Sunday).

    Args:
        start_date: Start date in YYYY-MM-DD format
        end_date: End date in YYYY-MM-DD format
        include_end: Whether to include end date in count

    Returns:
        Number of business days or error message
    """
    try:
        start = datetime.strptime(start_date, "%Y-%m-%d")
        end = datetime.strptime(end_date, "%Y-%m-%d")

        if start > end:
            return "Error: Start date must be before end date"

        business_days = 0
        current = start

        while current <= end:
            if current.weekday() < 5:  # Monday = 0, Friday = 4
                business_days += 1
            current += timedelta(days=1)

        if not include_end and business_days > 0:
            business_days -= 1

        return f"{business_days} business days between {start_date} and {end_date}"

    except ValueError:
        return "Error: Invalid date format. Use YYYY-MM-DD"
```

**Best Practices:**

- Validate inputs
- Return human-readable results
- Document assumptions (e.g., what counts as business day)
- Handle edge cases

---

## Error Handling Patterns

### Pattern: Graceful Tool Failure

**Structure:**

```python
def resilient_tool(param: str) -> str:
    """Tool with comprehensive error handling."""
    try:
        # Primary operation
        result = perform_operation(param)
        return result

    except SpecificError as e:
        # Handle known error
        logging.warning(f"Known error in tool: {e}")
        return f"Could not complete operation: {str(e)}"

    except Exception as e:
        # Handle unexpected errors
        logging.error(f"Unexpected error in tool: {e}", exc_info=True)
        return "An unexpected error occurred. Please try again."
```

**Benefits:**

- Agent continues functioning
- User gets meaningful feedback
- Errors logged for debugging

---

### Pattern: Retry with Fallback

**Structure:**

```python
class ResilientAgent(BaseAgent):
    primary_agent: LlmAgent
    fallback_agent: LlmAgent
    model_config = {"arbitrary_types_allowed": True}

    async def _run_async_impl(self, ctx) -> AsyncGenerator[Event, None]:
        try:
            # Try primary approach
            async for event in self.primary_agent.run_async(ctx):
                yield event

            # Check if successful
            if ctx.session.state.get("success"):
                return

        except Exception as e:
            logging.warning(f"Primary agent failed: {e}")

        # Fall back to alternative
        ctx.session.state["fallback_mode"] = True
        async for event in self.fallback_agent.run_async(ctx):
            yield event
```

**Use Cases:**

- API unavailability
- Tool failures
- Quality issues

---

## Testing Patterns

### Pattern: Tool Unit Test

```python
import pytest
from unittest.mock import patch

def test_api_tool_success():
    """Test tool with successful API response."""
    with patch('tool_module.requests.get') as mock_get:
        mock_response = mock_get.return_value
        mock_response.json.return_value = {"data": "value"}
        mock_response.raise_for_status = lambda: None

        result = api_tool("query")

        assert "value" in result
        mock_get.assert_called_once()

def test_api_tool_error():
    """Test tool handles API errors."""
    with patch('tool_module.requests.get') as mock_get:
        mock_get.side_effect = requests.RequestException("API Error")

        result = api_tool("query")

        assert "Error" in result
```

---

### Pattern: Agent Integration Test

```python
@pytest.mark.asyncio
async def test_agent_workflow():
    """Test complete agent workflow."""
    session_service = InMemorySessionService()
    session_service.create_session("app", "user", "session")

    agent = create_test_agent()
    runner = Runner(agent, "app", session_service)

    # Execute
    message = types.Content(role='user', parts=[types.Part(text="test query")])

    final_response = None
    async for event in runner.run_async("user", "session", message):
        if event.is_final_response():
            final_response = event.content.parts[0].text

    # Verify
    assert final_response is not None
    assert "expected content" in final_response.lower()

    # Verify state
    session = session_service.get_session("app", "user", "session")
    assert "expected_key" in session.state
```

---

## Anti-Patterns (Avoid These)

### ❌ Anti-Pattern: Tools with output_schema

```python
# WRONG - output_schema and tools are mutually exclusive
bad_agent = LlmAgent(
    output_schema=MySchema,
    tools=[my_tool]  # Won't work!
)
```

**Fix:** Choose one - either structured output OR tools.

---

### ❌ Anti-Pattern: Overly Complex Custom Agent

```python
# WRONG - Using custom agent for simple sequential workflow
class UnnecessaryCustomAgent(BaseAgent):
    async def _run_async_impl(self, ctx):
        async for event in self.agent1.run_async(ctx):
            yield event
        async for event in self.agent2.run_async(ctx):
            yield event
```

**Fix:** Use `SequentialAgent` instead.

---

### ❌ Anti-Pattern: Vague Tool Docstrings

```python
# WRONG - LLM won't know when to call this
def process(data: str) -> str:
    """Processes data."""
    return result
```

**Fix:** Detailed docstrings explaining when, why, and how.

---

### ❌ Anti-Pattern: Too Many Tools

```python
# WRONG - 20+ tools confuses LLM
agent = LlmAgent(tools=[tool1, tool2, ..., tool25])  # Too many!
```

**Fix:** Group related tools, create specialized sub-agents.

---

### ❌ Anti-Pattern: Ignoring State Keys

```python
# WRONG - Assuming state key exists
value = ctx.session.state["key"]  # KeyError if not exists
```

**Fix:** Use `.get()` with default.

```python
value = ctx.session.state.get("key", "default")
```

---

## Quick Pattern Selection Guide

**Need to...**

- Answer questions: → `LlmAgent` with instruction
- Call APIs: → `LlmAgent` with tools
- Output JSON: → `LlmAgent` with output_schema (no tools)
- Sequential steps: → `SequentialAgent`
- Iterative refinement: → `LoopAgent`
- Parallel analysis: → `ParallelAgent`
- Conditional logic: → Custom `BaseAgent`
- Multiple specialists: → Coordinator with sub_agents
- Share data between agents: → output_key + session.state

---

This knowledge base provides proven patterns for building effective ADK agents. Choose the simplest pattern that meets your needs.
