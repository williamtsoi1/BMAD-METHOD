# Add ADK Tools

## Purpose

Add custom tools to Google ADK agents to extend their capabilities beyond LLM knowledge.

## When to Use

- Agent needs to call external APIs
- Agent requires real-time data
- Agent must perform calculations
- Agent needs to interact with databases
- Agent should execute specific actions

## Tool Implementation Guide

### 1. Understand Tool Basics

**What is a Tool?**
A Python function that the LLM can call to perform specific actions or retrieve information.

**How ADK Uses Tools:**

1. LLM reads tool name and docstring
2. LLM decides when to call tool based on user query
3. LLM generates function call with parameters
4. ADK executes the function
5. Result returned to LLM
6. LLM incorporates result into response

---

### 2. Define Tool Function

**Basic Pattern:**

```python
def tool_name(param1: str, param2: int = 0) -> str:
    """
    Clear, descriptive explanation of what this tool does.

    The LLM uses this docstring to decide when to call this tool,
    so make it comprehensive and specific.

    Args:
        param1: Description of param1 purpose and format
        param2: Description of param2 (optional parameter example)

    Returns:
        Description of what the function returns
    """
    # Implementation here
    result = f"Processed {param1} with value {param2}"
    return result
```

**Critical Requirements:**

- ✅ Type hints on ALL parameters and return value
- ✅ Comprehensive docstring (LLM reads this!)
- ✅ Clear, descriptive function name
- ✅ Single responsibility per tool
- ✅ Return meaningful values

---

### 3. Tool Design Patterns

#### Pattern: API Call Tool

```python
import requests

def get_weather(city: str) -> str:
    """
    Retrieves current weather information for a specified city.

    Args:
        city: Name of the city (e.g., "San Francisco", "Tokyo")

    Returns:
        Weather description string including temperature and conditions
    """
    try:
        # Replace with actual API
        response = requests.get(
            f"https://api.weather.com/v1/current",
            params={"city": city, "apikey": os.getenv("WEATHER_API_KEY")}
        )
        response.raise_for_status()
        data = response.json()
        return f"Weather in {city}: {data['temp']}°C, {data['conditions']}"
    except Exception as e:
        return f"Error retrieving weather: {str(e)}"
```

#### Pattern: Database Query Tool

```python
import sqlite3

def lookup_customer(customer_id: str) -> str:
    """
    Looks up customer information from the database.

    Args:
        customer_id: Unique customer identifier

    Returns:
        Customer information as formatted string, or error message
    """
    try:
        conn = sqlite3.connect('customers.db')
        cursor = conn.cursor()
        cursor.execute(
            "SELECT name, email, status FROM customers WHERE id=?",
            (customer_id,)
        )
        result = cursor.fetchone()
        conn.close()

        if result:
            name, email, status = result
            return f"Customer: {name}, Email: {email}, Status: {status}"
        else:
            return f"Customer {customer_id} not found"
    except Exception as e:
        return f"Database error: {str(e)}"
```

#### Pattern: Calculation Tool

```python
from datetime import datetime, timedelta

def calculate_delivery_date(
    order_date: str,
    shipping_method: str = "standard"
) -> str:
    """
    Calculates estimated delivery date based on order date and shipping method.

    Args:
        order_date: Order date in YYYY-MM-DD format
        shipping_method: Either "standard" (5-7 days) or "express" (2-3 days)

    Returns:
        Estimated delivery date in YYYY-MM-DD format
    """
    try:
        order = datetime.strptime(order_date, "%Y-%m-%d")

        if shipping_method == "express":
            delivery = order + timedelta(days=2)
        else:
            delivery = order + timedelta(days=5)

        return delivery.strftime("%Y-%m-%d")
    except ValueError:
        return "Error: Invalid date format. Use YYYY-MM-DD"
```

#### Pattern: Action Tool

```python
def send_email(to: str, subject: str, body: str) -> str:
    """
    Sends an email to the specified recipient.

    Args:
        to: Recipient email address
        subject: Email subject line
        body: Email body content

    Returns:
        Success or error message
    """
    try:
        # Replace with actual email service
        # Example: SendGrid, AWS SES, etc.
        success = email_service.send(
            to=to,
            subject=subject,
            body=body
        )

        if success:
            return f"Email sent successfully to {to}"
        else:
            return "Failed to send email"
    except Exception as e:
        return f"Email error: {str(e)}"
```

---

### 4. Add Tools to Agent

**Simple Addition:**

```python
from google.adk.agents import LlmAgent

agent = LlmAgent(
    name="customer_service_agent",
    model="gemini-2.0-flash",
    instruction="""
    You are a customer service agent.
    Use the lookup_customer tool to find customer information.
    Use the send_email tool to send updates to customers.
    """,
    tools=[lookup_customer, send_email]
)
```

**With Built-in Tools:**

```python
from google.adk.tools import google_search

agent = LlmAgent(
    name="research_agent",
    model="gemini-2.0-flash",
    instruction="Use Google Search and weather data to answer questions.",
    tools=[
        google_search,        # Built-in ADK tool
        get_weather,          # Custom tool
        calculate_delivery_date  # Custom tool
    ]
)
```

---

### 5. Tool Best Practices

#### ✅ DO:

**Clear Naming:**

```python
# Good
def get_customer_order_history(customer_id: str) -> str:

# Bad
def fetch(id: str) -> str:
```

**Comprehensive Docstrings:**

```python
def calculate_tax(amount: float, state: str) -> float:
    """
    Calculates sales tax for a purchase based on state tax rates.

    This tool applies the correct state sales tax rate to the given amount.
    It handles all 50 US states and returns the total tax amount.

    Args:
        amount: Purchase amount in USD (e.g., 100.00)
        state: Two-letter state code (e.g., "CA", "NY")

    Returns:
        Tax amount in USD rounded to 2 decimal places

    Examples:
        >>> calculate_tax(100.00, "CA")
        8.25  # 8.25% California tax
    """
```

**Error Handling:**

```python
def get_stock_price(symbol: str) -> str:
    """Retrieves current stock price for given symbol."""
    try:
        # API call
        price = api.get_price(symbol)
        return f"{symbol}: ${price:.2f}"
    except APIError as e:
        return f"API Error: {str(e)}"
    except ValueError:
        return f"Invalid symbol: {symbol}"
    except Exception as e:
        return f"Unexpected error: {str(e)}"
```

**Single Responsibility:**

```python
# Good - focused tools
def get_customer_info(customer_id: str) -> str:
    """Gets customer information."""

def update_customer_email(customer_id: str, new_email: str) -> str:
    """Updates customer email address."""

# Bad - doing too much
def manage_customer(customer_id: str, action: str, **kwargs) -> str:
    """Does everything with customers."""
```

#### ❌ DON'T:

**Missing Type Hints:**

```python
# Bad
def get_data(param):
    return "result"
```

**Vague Docstrings:**

```python
# Bad
def process(input: str) -> str:
    """Processes input."""
```

**No Error Handling:**

```python
# Bad
def call_api(url: str) -> str:
    response = requests.get(url)  # What if this fails?
    return response.text
```

---

### 6. Testing Tools

**Unit Test Pattern:**

```python
import pytest

def test_calculate_delivery_date_standard():
    result = calculate_delivery_date("2024-01-01", "standard")
    assert result == "2024-01-06"

def test_calculate_delivery_date_express():
    result = calculate_delivery_date("2024-01-01", "express")
    assert result == "2024-01-03"

def test_calculate_delivery_date_invalid():
    result = calculate_delivery_date("invalid-date", "standard")
    assert "Error" in result

def test_get_weather_success():
    # Mock API response
    result = get_weather("San Francisco")
    assert "Weather in San Francisco" in result

def test_get_weather_api_error():
    # Test error handling
    result = get_weather("InvalidCity123")
    assert "Error" in result
```

**Integration Test with Agent:**

```python
import asyncio
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.genai import types

async def test_agent_uses_tool():
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
        instruction="Use the weather tool to answer weather questions.",
        tools=[get_weather]
    )

    runner = Runner(
        agent=agent,
        app_name="test_app",
        session_service=session_service
    )

    # Test
    user_message = types.Content(
        role='user',
        parts=[types.Part(text="What's the weather in Tokyo?")]
    )

    final_response = None
    async for event in runner.run_async(
        user_id="test_user",
        session_id="test_session",
        new_message=user_message
    ):
        if event.is_final_response():
            final_response = event.content.parts[0].text

    # Verify tool was used and response contains weather info
    assert final_response is not None
    assert "Tokyo" in final_response
    # Check that weather info is in response

asyncio.run(test_agent_uses_tool())
```

---

### 7. Advanced Tool Patterns

#### Stateful Tools (Using Session State)

```python
def save_preference(key: str, value: str) -> str:
    """
    Saves a user preference to session state.

    Args:
        key: Preference name (e.g., "language", "theme")
        value: Preference value

    Returns:
        Confirmation message
    """
    # Note: Access to session requires special setup
    # Typically done via context passed to tool
    return f"Saved {key}={value}"
```

#### Async Tools

```python
async def fetch_data_async(url: str) -> str:
    """
    Asynchronously fetches data from a URL.

    Args:
        url: URL to fetch from

    Returns:
        Response content or error message
    """
    import aiohttp
    try:
        async with aiohttp.ClientSession() as session:
            async with session.get(url) as response:
                content = await response.text()
                return content
    except Exception as e:
        return f"Error: {str(e)}"
```

#### Tools with Complex Returns

```python
from typing import Dict, Any
import json

def get_product_details(product_id: str) -> str:
    """
    Retrieves detailed product information.

    Args:
        product_id: Unique product identifier

    Returns:
        JSON string containing product details
    """
    # Fetch from database/API
    product_data = {
        "id": product_id,
        "name": "Example Product",
        "price": 29.99,
        "in_stock": True,
        "categories": ["electronics", "accessories"]
    }

    # Return as JSON string for LLM to parse
    return json.dumps(product_data)
```

---

### 8. Tool Organization

**File Structure:**

```
project/
├── tools/
│   ├── __init__.py
│   ├── weather.py       # Weather-related tools
│   ├── database.py      # Database tools
│   ├── email.py         # Email tools
│   └── calculations.py  # Calculation tools
```

**Example tools/**init**.py:**

```python
from .weather import get_weather, get_forecast
from .database import lookup_customer, get_order
from .email import send_email
from .calculations import calculate_tax, calculate_delivery_date

__all__ = [
    'get_weather',
    'get_forecast',
    'lookup_customer',
    'get_order',
    'send_email',
    'calculate_tax',
    'calculate_delivery_date'
]
```

---

## Completion Checklist

- [ ] Tool functions defined with clear names
- [ ] All parameters have type hints
- [ ] Comprehensive docstrings written
- [ ] Error handling implemented
- [ ] Single responsibility per tool
- [ ] Tools return appropriate types
- [ ] Unit tests written and passing
- [ ] Integration tests with agent passing
- [ ] Tools added to agent configuration
- [ ] Agent instruction mentions when to use tools
- [ ] Documentation updated

## Common Issues

**Issue:** LLM not calling tool

- **Fix:** Improve docstring clarity, make sure it matches user query patterns

**Issue:** Tool function fails

- **Fix:** Add error handling, validate inputs, return error messages not exceptions

**Issue:** Type errors

- **Fix:** Ensure all parameters and returns have proper type hints

**Issue:** Tool returns unhelpful data

- **Fix:** Format return value as clear string LLM can understand

## Remember

Tools are the bridge between LLM reasoning and real-world actions. Make them:

- **Reliable:** Handle errors gracefully
- **Clear:** LLM must understand when to call them
- **Focused:** One job per tool
- **Tested:** Verify independently and with agent
