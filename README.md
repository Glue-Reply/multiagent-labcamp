# ADK Python Quickstart

A basic setup guide for building agents with [Agent Development Kit (ADK)](https://adk.dev/get-started/python/).

**Prerequisites:** Python 3.10+, `pip`

---

## 1. Installation

Create and activate a virtual environment (recommended):

```bash
python -m venv .venv
source .venv/bin/activate
```

Install ADK:

```bash
pip install google-adk
```

---

## 2. Create an Agent Project

```bash
adk create my_agent
```

This generates the following structure:

```
my_agent/
    agent.py      # main agent code
    .env          # API keys or project IDs
    __init__.py
```

---

## 3. Set Your API Key (If not prompted)

Get a Gemini API key from [Google AI Studio](https://aistudio.google.com/app/apikey), then add it to `my_agent/.env`:

```bash
echo 'GOOGLE_API_KEY="YOUR_API_KEY"' > my_agent/.env
```

---

## 4. Update the Agent

Edit `my_agent/agent.py` to define tools and the root agent:

```python
from google.adk.agents.llm_agent import Agent

# Mock tool implementation
def get_current_time(city: str) -> dict:
    """Returns the current time in a specified city."""
    return {"status": "success", "city": city, "time": "10:30 AM"}

root_agent = Agent(
    model='gemini-flash-latest',
    name='root_agent',
    description="Tells the current time in a specified city.",
    instruction="You are a helpful assistant that tells the current time in cities. Use the 'get_current_time' tool for this purpose.",
    tools=[get_current_time],
)
```

---

## 5. Run Your Agent

**Command-line interface:**

```bash
adk run my_agent
```

**Web interface** (access at http://localhost:8000):

```bash
adk web --port 8000
```

> Note: Run `adk web` from the parent directory containing your `my_agent/` folder.

---

---

## 6. A2A — Agent-to-Agent Protocol

A2A is the standard for multi-agent communication in ADK. It lets independent agents — potentially running on different machines, written in different languages, or maintained by different teams — discover and call each other over a network.

### When to use A2A vs. local sub-agents

| Scenario | Recommendation |
|---|---|
| Separate service / different team | A2A |
| Cross-language agents | A2A |
| Microservices architecture | A2A |
| Internal code organisation / shared memory | Local sub-agent |
| Performance-critical, tightly coupled logic | Local sub-agent |

### Install the A2A extras

```bash
pip install google-adk[a2a]
```

### Expose an agent via A2A

Wrap an existing `root_agent` with `to_a2a()` and serve it with `uvicorn`:

```python
from google.adk.a2a.utils.agent_to_a2a import to_a2a

# Exposes root_agent as an A2A server; auto-generates the agent card
a2a_app = to_a2a(root_agent, port=8001)
```

```bash
uvicorn my_agent.agent:a2a_app --host localhost --port 8001
```

Verify the auto-generated agent card at: http://localhost:8001/.well-known/agent-card.json

### Consume a remote agent

Use `RemoteA2aAgent` as a sub-agent inside another ADK agent:

```python
from google.adk.agents import Agent
from google.adk.agents.remote_a2a_agent import RemoteA2aAgent

remote = RemoteA2aAgent(
    agent_card_url="http://localhost:8001/.well-known/agent-card.json",
)

root_agent = Agent(
    model='gemini-flash-latest',
    name='root_agent',
    description="Orchestrates remote agents.",
    instruction="Use the remote agent to handle specialised tasks.",
    tools=[remote],
)
```

### Run both agents

```bash
# Terminal 1 – start the remote agent
uvicorn my_agent.agent:a2a_app --host localhost --port 8001

# Terminal 2 – start the consuming agent web UI
adk web --port 8000
```

### Further reading

- [Introduction to A2A](https://adk.dev/a2a/intro/)
- [Quickstart: Exposing an agent](https://adk.dev/a2a/quickstart-exposing/)
- [Quickstart: Consuming an agent](https://adk.dev/a2a/quickstart-consuming/)

---

## Next Steps

- [Build your agent](https://adk.dev/tutorials/)
- [ADK Python API Reference](https://adk.dev/api-reference/python/)
