# Setup ADK Tests

## Purpose

Create comprehensive testing infrastructure for Google ADK agents including unit tests, integration tests, and evaluation sets.

## Testing Strategy

ADK agents should be tested at three levels:

1. **Unit Tests** - Individual tools and functions
2. **Integration Tests** - Agent behavior and workflows
3. **Evaluation Sets** - End-to-end agent performance

---

## 1. Unit Testing Tools

### Test File Structure

```
tests/
├── __init__.py
├── test_tools.py           # Tool unit tests
├── test_agent_logic.py     # Agent logic tests
└── test_integration.py     # Integration tests
```

### Testing Individual Tools

**Pattern:**

```python
import pytest
from tools.weather import get_weather
from tools.calculations import calculate_delivery_date

class TestWeatherTool:
    def test_get_weather_valid_city(self):
        """Test weather tool with valid city."""
        result = get_weather("San Francisco")
        assert "San Francisco" in result
        assert "°" in result  # Temperature symbol

    def test_get_weather_invalid_city(self):
        """Test weather tool with invalid city."""
        result = get_weather("InvalidCity12345")
        assert "Error" in result or "not found" in result.lower()

    def test_get_weather_empty_input(self):
        """Test weather tool with empty input."""
        with pytest.raises(ValueError):
            get_weather("")

class TestCalculationTools:
    def test_delivery_date_standard_shipping(self):
        """Test standard shipping calculation."""
        result = calculate_delivery_date("2024-01-01", "standard")
        assert result == "2024-01-06"

    def test_delivery_date_express_shipping(self):
        """Test express shipping calculation."""
        result = calculate_delivery_date("2024-01-01", "express")
        assert result == "2024-01-03"

    def test_delivery_date_invalid_format(self):
        """Test invalid date format handling."""
        result = calculate_delivery_date("invalid-date", "standard")
        assert "Error" in result

    @pytest.mark.parametrize("date,method,expected", [
        ("2024-01-01", "standard", "2024-01-06"),
        ("2024-01-01", "express", "2024-01-03"),
        ("2024-12-25", "standard", "2024-12-30"),
    ])
    def test_delivery_date_parametrized(self, date, method, expected):
        """Test multiple delivery date scenarios."""
        result = calculate_delivery_date(date, method)
        assert result == expected
```

### Testing with Mocks

**Pattern for API Tools:**

```python
import pytest
from unittest.mock import patch, MagicMock
from tools.api import fetch_stock_price

class TestStockPriceTool:
    @patch('tools.api.requests.get')
    def test_fetch_stock_price_success(self, mock_get):
        """Test successful stock price fetch."""
        # Setup mock
        mock_response = MagicMock()
        mock_response.json.return_value = {'price': 150.25}
        mock_response.raise_for_status = MagicMock()
        mock_get.return_value = mock_response

        # Test
        result = fetch_stock_price("AAPL")

        # Verify
        assert "150.25" in result
        mock_get.assert_called_once()

    @patch('tools.api.requests.get')
    def test_fetch_stock_price_api_error(self, mock_get):
        """Test API error handling."""
        mock_get.side_effect = Exception("API Error")

        result = fetch_stock_price("AAPL")

        assert "Error" in result
```

---

## 2. Integration Testing Agents

### Testing LlmAgent Behavior

**Basic Agent Test:**

```python
import asyncio
import pytest
from google.adk.agents import LlmAgent
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.genai import types

@pytest.mark.asyncio
async def test_simple_agent_response():
    """Test agent provides appropriate response."""
    # Setup
    session_service = InMemorySessionService()
    session_service.create_session(
        app_name="test_app",
        user_id="test_user",
        session_id="test_session"
    )

    agent = LlmAgent(
        name="test_agent",
        model="gemini-2.0-flash",
        instruction="You are a helpful assistant. Answer questions concisely.",
        description="Test agent"
    )

    runner = Runner(
        agent=agent,
        app_name="test_app",
        session_service=session_service
    )

    # Execute
    user_message = types.Content(
        role='user',
        parts=[types.Part(text="What is 2+2?")]
    )

    final_response = None
    async for event in runner.run_async(
        user_id="test_user",
        session_id="test_session",
        new_message=user_message
    ):
        if event.is_final_response():
            final_response = event.content.parts[0].text

    # Verify
    assert final_response is not None
    assert "4" in final_response
```

### Testing Agent with Tools

**Pattern:**

```python
@pytest.mark.asyncio
async def test_agent_uses_tool_correctly():
    """Test agent calls tool when appropriate."""
    from tools.weather import get_weather

    # Setup
    session_service = InMemorySessionService()
    session_service.create_session(
        app_name="test_app",
        user_id="test_user",
        session_id="test_session"
    )

    agent = LlmAgent(
        name="weather_agent",
        model="gemini-2.0-flash",
        instruction="Use the get_weather tool to answer weather questions.",
        tools=[get_weather]
    )

    runner = Runner(
        agent=agent,
        app_name="test_app",
        session_service=session_service
    )

    # Execute
    user_message = types.Content(
        role='user',
        parts=[types.Part(text="What's the weather in Tokyo?")]
    )

    tool_called = False
    final_response = None

    async for event in runner.run_async(
        user_id="test_user",
        session_id="test_session",
        new_message=user_message
    ):
        # Check if tool was called
        if hasattr(event, 'tool_call'):
            tool_called = True

        if event.is_final_response():
            final_response = event.content.parts[0].text

    # Verify
    assert final_response is not None
    assert "Tokyo" in final_response
    # Response should contain weather information
```

### Testing Multi-Agent Systems

**Pattern:**

```python
@pytest.mark.asyncio
async def test_multi_agent_coordination():
    """Test multi-agent system with data flow."""
    # Setup
    session_service = InMemorySessionService()
    session_service.create_session(
        app_name="test_app",
        user_id="test_user",
        session_id="test_session"
    )

    # Create sub-agents
    generator = LlmAgent(
        name="generator",
        model="gemini-2.0-flash",
        instruction="Generate a short sentence about cats.",
        output_key="generated_text"
    )

    reviewer = LlmAgent(
        name="reviewer",
        model="gemini-2.0-flash",
        instruction="Review the text in session state 'generated_text' and rate it 1-10.",
        output_key="review_score"
    )

    # Create coordinator
    coordinator = LlmAgent(
        name="coordinator",
        model="gemini-2.0-flash",
        instruction="Coordinate text generation and review.",
        sub_agents=[generator, reviewer]
    )

    runner = Runner(
        agent=coordinator,
        app_name="test_app",
        session_service=session_service
    )

    # Execute
    user_message = types.Content(
        role='user',
        parts=[types.Part(text="Generate and review a sentence about cats.")]
    )

    async for event in runner.run_async(
        user_id="test_user",
        session_id="test_session",
        new_message=user_message
    ):
        pass  # Process all events

    # Verify session state
    session = session_service.get_session(
        app_name="test_app",
        user_id="test_user",
        session_id="test_session"
    )

    assert "generated_text" in session.state
    assert "review_score" in session.state
    assert len(session.state["generated_text"]) > 0
```

### Testing Custom Agents

**Pattern:**

```python
@pytest.mark.asyncio
async def test_custom_agent_orchestration():
    """Test custom agent with conditional logic."""
    from agents.custom_workflow import CustomWorkflowAgent

    # Setup agent with mocked sub-agents or real ones
    # ... agent setup ...

    # Test various paths through custom logic
    # Test state management
    # Test conditional branching
    # Test error handling
```

---

## 3. Evaluation Sets

### Creating Evaluation Set File

**Format: `test_cases.evalset.json`**

```json
{
  "metadata": {
    "name": "Weather Agent Evaluation",
    "version": "1.0",
    "description": "Tests for weather agent functionality"
  },
  "test_cases": [
    {
      "name": "Simple weather query",
      "input": "What's the weather in San Francisco?",
      "expected_output_contains": ["San Francisco", "weather"],
      "tags": ["basic", "weather"]
    },
    {
      "name": "Complex weather query",
      "input": "Compare the weather in Paris and London",
      "expected_output_contains": ["Paris", "London", "weather"],
      "tags": ["complex", "weather", "comparison"]
    },
    {
      "name": "Invalid city handling",
      "input": "What's the weather in InvalidCity12345?",
      "expected_output_contains": ["sorry", "not found", "error"],
      "tags": ["error-handling"]
    },
    {
      "name": "Forecast query",
      "input": "What will the weather be like tomorrow in Tokyo?",
      "expected_output_contains": ["Tokyo", "tomorrow", "forecast"],
      "tags": ["forecast", "weather"]
    }
  ]
}
```

### Running Evaluation Sets

**Command:**

```bash
adk eval path/to/agent path/to/test_cases.evalset.json
```

**Programmatic Evaluation:**

```python
from google.adk.evaluation import Evaluator

evaluator = Evaluator(agent=my_agent)
results = evaluator.evaluate("path/to/test_cases.evalset.json")

# Analyze results
print(f"Passed: {results.passed_count}/{results.total_count}")
print(f"Success Rate: {results.success_rate:.2%}")

for failure in results.failures:
    print(f"Failed: {failure.name}")
    print(f"Expected: {failure.expected}")
    print(f"Got: {failure.actual}")
```

---

## 4. Test Configuration

### pytest Configuration

**`pytest.ini`:**

```ini
[pytest]
asyncio_mode = auto
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
markers =
    slow: marks tests as slow (deselect with '-m "not slow"')
    integration: marks tests as integration tests
    unit: marks tests as unit tests
```

### Requirements for Testing

**`requirements-test.txt`:**

```
pytest>=7.0.0
pytest-asyncio>=0.21.0
pytest-cov>=4.0.0
pytest-mock>=3.10.0
google-adk>=1.0.0
```

---

## 5. Running Tests

### Run All Tests

```bash
pytest
```

### Run Specific Test File

```bash
pytest tests/test_tools.py
```

### Run Tests with Coverage

```bash
pytest --cov=agents --cov=tools --cov-report=html
```

### Run Only Unit Tests

```bash
pytest -m unit
```

### Run Only Integration Tests

```bash
pytest -m integration
```

### Run with Verbose Output

```bash
pytest -v
```

---

## 6. Continuous Integration

### GitHub Actions Example

**`.github/workflows/test.yml`:**

```yaml
name: Run Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2

      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.10'

      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install -r requirements-test.txt

      - name: Run unit tests
        run: pytest -m unit --cov=agents --cov=tools

      - name: Run integration tests
        run: pytest -m integration
        env:
          GEMINI_API_KEY: ${{ secrets.GEMINI_API_KEY }}

      - name: Run evaluation sets
        run: |
          adk eval agents/weather_agent eval/weather.evalset.json
```

---

## 7. Testing Best Practices

### ✅ DO:

- **Test tools independently** before testing with agents
- **Use mocks** for external APIs and services
- **Test error paths** not just happy paths
- **Create evaluation sets** for end-to-end validation
- **Test state management** in multi-agent systems
- **Use parametrized tests** for multiple scenarios
- **Measure coverage** and aim for >80%
- **Test asynchronously** for async agents

### ❌ DON'T:

- **Skip testing tools** - they're critical
- **Rely only on manual testing** - automate
- **Test against live APIs** in unit tests - mock them
- **Ignore edge cases** - test them
- **Skip integration tests** - agent behavior matters
- **Forget to test multi-agent data flow**

---

## 8. Test Documentation

### Document Test Strategy

**`tests/README.md`:**

````markdown
# Test Suite

## Structure

- `test_tools.py` - Unit tests for custom tools
- `test_agent_logic.py` - Agent behavior tests
- `test_integration.py` - End-to-end integration tests

## Running Tests

```bash
# All tests
pytest

# Unit tests only
pytest -m unit

# With coverage
pytest --cov
```
````

## Evaluation Sets

Located in `eval/` directory.

Run with:

```bash
adk eval agents/my_agent eval/test_cases.evalset.json
```

## Coverage Goals

- Tools: 100%
- Agents: 85%
- Overall: 90%

```

---

## Completion Checklist

- [ ] Unit tests created for all tools
- [ ] Integration tests created for agents
- [ ] Evaluation sets created for end-to-end testing
- [ ] All tests passing
- [ ] Test coverage >80%
- [ ] Mocks used for external dependencies
- [ ] Error cases tested
- [ ] Multi-agent data flow tested (if applicable)
- [ ] CI/CD pipeline configured (if applicable)
- [ ] Test documentation complete

## Common Testing Issues

**Issue:** Tests fail intermittently
- **Fix:** Ensure tests are deterministic, mock random/time-based behavior

**Issue:** Async tests not running
- **Fix:** Install `pytest-asyncio`, use `@pytest.mark.asyncio` decorator

**Issue:** Tools not being called in tests
- **Fix:** Verify agent instruction mentions tool, check tool docstring clarity

**Issue:** Coverage too low
- **Fix:** Add tests for uncovered paths, especially error handling

## Remember

Good tests are:
- **Fast** - Run quickly for rapid iteration
- **Isolated** - Don't depend on external services
- **Repeatable** - Same result every time
- **Comprehensive** - Cover happy paths and edge cases
```
