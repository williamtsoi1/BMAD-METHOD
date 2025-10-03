# Implement ADK Agent

## Purpose

Implement an AI agent using Google's Agent Development Kit (ADK) following code-first best practices.

## Prerequisites

- Google ADK installed: `pip install google-adk`
- Gemini API access configured
- Agent design document or story with requirements

## Task Steps

### 1. Analyze Requirements

**Action:** Review the agent requirements from the story file.

**Key Questions:**

- What is the agent's primary purpose?
- What capabilities/tools does it need?
- Is this a single agent or multi-agent system?
- Does it need structured input/output?
- What's the expected data flow?

**Output:** Clear understanding of agent type and architecture needed.

---

### 2. Choose Agent Pattern

**Decision Tree:**

```
Simple task with clear instruction?
  └─> Use LlmAgent with instruction + optional tools

Need structured I/O for workflow integration?
  └─> Use LlmAgent with input_schema/output_schema + output_key

Multiple specialized agents working together?
  └─> Use LlmAgent with sub_agents (multi-agent system)

Standard sequential/loop/parallel workflow?
  └─> Use SequentialAgent, LoopAgent, or ParallelAgent

Complex conditional logic or custom orchestration?
  └─> Create custom agent inheriting from BaseAgent
```

**Output:** Selected agent pattern documented.

---

### 3. Set Up Project Structure

**Action:** Create necessary files and directories.

**Standard Structure:**

```
project/
├── agents/
│   ├── __init__.py
│   └── my_agent.py          # Agent implementation
├── tools/
│   ├── __init__.py
│   └── custom_tools.py      # Custom tool functions
├── tests/
│   ├── test_agent.py        # Agent tests
│   └── test_tools.py        # Tool tests
├── eval/
│   └── test_cases.evalset.json  # Evaluation set
├── requirements.txt
└── README.md
```

**Output:** Project structure created.

---

### 4. Implement Basic Agent

**For LlmAgent:**

```python
from google.adk.agents import LlmAgent

agent = LlmAgent(
    name="my_agent",
    model="gemini-2.0-flash",
    instruction="""
    Clear, specific instructions for the agent.
    Include:
    - What the agent should do
    - How to handle different inputs
    - Expected output format
    - Any constraints or guidelines
    """,
    description="Brief description of what this agent does",
    # Add tools, schemas, etc. as needed
)
```

**For Custom Agent:**

```python
from google.adk.agents import BaseAgent, LlmAgent
from google.adk.sessions import InvocationContext
from typing import AsyncGenerator
from google.adk.events import Event

class MyCustomAgent(BaseAgent):
    """Custom agent with specific orchestration logic."""

    # Pydantic field declarations
    sub_agent_1: LlmAgent
    sub_agent_2: LlmAgent
    model_config = {"arbitrary_types_allowed": True}

    def __init__(self, name: str, sub_agent_1: LlmAgent, sub_agent_2: LlmAgent):
        super().__init__(
            name=name,
            sub_agent_1=sub_agent_1,
            sub_agent_2=sub_agent_2,
            sub_agents=[sub_agent_1, sub_agent_2]
        )

    async def _run_async_impl(
        self, ctx: InvocationContext
    ) -> AsyncGenerator[Event, None]:
        # Custom orchestration logic
        async for event in self.sub_agent_1.run_async(ctx):
            yield event

        # Conditional or complex logic here
        result = ctx.session.state.get("some_key")
        if result == "condition":
            async for event in self.sub_agent_2.run_async(ctx):
                yield event
```

**Output:** Agent implementation file created.

---

### 5. Add Tools (if needed)

**Action:** Implement custom tool functions.

**Pattern:**

```python
def my_tool(param1: str, param2: int = 0) -> str:
    """
    Clear description of what this tool does.

    The LLM uses this docstring to decide when to call this tool.

    Args:
        param1: Description of parameter 1
        param2: Description of parameter 2 (optional)

    Returns:
        Description of return value
    """
    # Implementation
    result = f"Processed {param1} with {param2}"
    return result

# Add to agent
agent = LlmAgent(
    # ... other params
    tools=[my_tool]
)
```

**Best Practices:**

- One tool = one responsibility
- Clear naming (tool name visible to LLM)
- Type hints required
- Comprehensive docstrings
- Proper error handling
- Return meaningful values

**Output:** Tool functions implemented and integrated.

---

### 6. Add Schemas (if needed)

**Action:** Define structured I/O using Pydantic.

**Pattern:**

```python
from pydantic import BaseModel, Field

class MyInput(BaseModel):
    query: str = Field(description="The user's query")
    context: str = Field(description="Additional context")

class MyOutput(BaseModel):
    result: str = Field(description="The processed result")
    confidence: float = Field(description="Confidence score 0-1")

agent = LlmAgent(
    # ... other params
    input_schema=MyInput,
    output_schema=MyOutput,
    output_key="agent_result"  # Stores in session.state
)
```

**CRITICAL:** Agents with `output_schema` CANNOT use tools.

**Output:** Schemas defined and integrated.

---

### 7. Configure Multi-Agent System (if applicable)

**Action:** Set up agent hierarchy and data flow.

**Pattern:**

```python
# Define specialists
specialist_1 = LlmAgent(
    name="specialist_1",
    model="gemini-2.0-flash",
    instruction="Specialist 1 instructions",
    output_key="spec1_result"
)

specialist_2 = LlmAgent(
    name="specialist_2",
    model="gemini-2.0-flash",
    instruction="Read spec1_result from session state and process it.",
    output_key="spec2_result"
)

# Create coordinator
coordinator = LlmAgent(
    name="coordinator",
    model="gemini-2.0-flash",
    instruction="Route tasks to specialists.",
    sub_agents=[specialist_1, specialist_2]
)
```

**Key Points:**

- Use `output_key` for agent-to-agent communication
- Reference session state keys in agent instructions
- Pass `sub_agents` list to parent

**Output:** Multi-agent system configured.

---

### 8. Add Configuration

**Action:** Configure generation parameters if needed.

```python
from google.genai import types

agent = LlmAgent(
    # ... other params
    generate_content_config=types.GenerateContentConfig(
        temperature=0.2,         # Deterministic vs creative
        max_output_tokens=500,   # Response length limit
        top_p=0.9,
        top_k=40
    )
)
```

**Output:** Generation config optimized.

---

### 9. Create Runner Setup

**Action:** Implement agent execution code.

```python
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.genai import types

# Setup session service
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
async def run_agent(query: str):
    user_message = types.Content(
        role='user',
        parts=[types.Part(text=query)]
    )

    async for event in runner.run_async(
        user_id="user_123",
        session_id="session_456",
        new_message=user_message
    ):
        if event.is_final_response():
            print(event.content.parts[0].text)
```

**Output:** Agent can be executed.

---

### 10. Write Tests

**Action:** Create comprehensive tests.

**Unit Tests for Tools:**

```python
def test_my_tool():
    result = my_tool("input", 5)
    assert result == expected_result
```

**Integration Tests for Agent:**

```python
import asyncio
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService

async def test_agent():
    # Setup
    session_service = InMemorySessionService()
    session_service.create_session(
        app_name="test_app",
        user_id="test_user",
        session_id="test_session"
    )

    runner = Runner(
        agent=my_agent,
        app_name="test_app",
        session_service=session_service
    )

    # Test
    user_message = types.Content(
        role='user',
        parts=[types.Part(text="Test query")]
    )

    responses = []
    async for event in runner.run_async(
        user_id="test_user",
        session_id="test_session",
        new_message=user_message
    ):
        if event.is_final_response():
            responses.append(event.content.parts[0].text)

    # Assertions
    assert len(responses) > 0
    assert "expected content" in responses[0].lower()

# Run test
asyncio.run(test_agent())
```

**Output:** Tests implemented and passing.

---

### 11. Create Evaluation Set

**Action:** Create `.evalset.json` file for ADK evaluation.

**Format:**

```json
{
  "test_cases": [
    {
      "name": "Basic query test",
      "input": "What is the capital of France?",
      "expected_output": "Paris"
    },
    {
      "name": "Complex query test",
      "input": "Compare the capitals of France and Germany",
      "expected_output_contains": ["Paris", "Berlin"]
    }
  ]
}
```

**Run Evaluation:**

```bash
adk eval path/to/agent path/to/test.evalset.json
```

**Output:** Evaluation set created and tests passing.

---

### 12. Document Implementation

**Action:** Create comprehensive documentation.

**Include:**

- Agent purpose and capabilities
- Usage examples
- Tool descriptions
- Session state keys used
- Configuration options
- Deployment instructions

**Output:** README.md created.

---

### 13. Update Story File

**Action:** Update Dev Agent Record sections in story.

**Update:**

- [x] Mark task checkboxes as completed
- Update File List with all new/modified files
- Add entries to Change Log
- Document any debugging in Debug Log
- Add Completion Notes

**Output:** Story file updated.

---

### 14. Final Verification

**Action:** Run through ADK implementation checklist.

**Verify:**

- All story requirements met
- Tests passing
- Code follows ADK best practices
- Documentation complete
- Ready for deployment

**Output:** Implementation ready for review.

---

## Common Patterns

### Pattern: Simple Q&A Agent

```python
LlmAgent with instruction + optional tools
```

### Pattern: Data Processing Agent

```python
LlmAgent with input_schema + output_schema + output_key
```

### Pattern: Research Agent

```python
LlmAgent with google_search tool
```

### Pattern: Multi-Stage Pipeline

```python
SequentialAgent with multiple LlmAgent sub-agents
```

### Pattern: Iterative Refinement

```python
LoopAgent with critic + reviser agents
```

### Pattern: Complex Workflow

```python
Custom BaseAgent with conditional logic in _run_async_impl
```

## Tips

✅ **Start Simple:** Use LlmAgent when possible, only go custom when needed.

✅ **Clear Instructions:** The instruction prompt is critical - be specific and detailed.

✅ **Tool Docstrings:** LLM reads these to decide when to call tools.

✅ **State Management:** Use output_key and session.state for data flow.

✅ **Test Early:** Don't wait until the end to test.

✅ **Iterate:** Start with basic functionality, then enhance.

## Completion Criteria

- [ ] Agent implemented following appropriate pattern
- [ ] All tools/schemas properly configured
- [ ] Tests written and passing
- [ ] Evaluation set created and passing
- [ ] Documentation complete
- [ ] Story file updated
- [ ] Code follows ADK best practices
