# MSFoundryAgentStreamEvent

> **Microsoft Foundry — Agent Streaming & Event Handling**
> Six production-ready patterns for real-time agent streaming using Azure AI Foundry's event-driven architecture — from basic token streaming to function calling, toolsets, file search, and custom event handler overrides.

---

## Overview

`MSFoundryAgentStreamEvent` demonstrates how to build **streaming-first agents** with Azure AI Foundry using event handlers. Rather than waiting for a full response, these patterns give you granular, real-time control over every stage of agent execution — token by token, tool call by tool call.

Each script is a self-contained, runnable reference implementation covering a distinct streaming pattern. Together they form a complete playbook for production agent observability and interactivity.

---

## Streaming Patterns Covered

| Script | Pattern | What it demonstrates |
|---|---|---|
| `agents_stream_eventhandler.py` | **Basic streaming** | Core event handler setup — stream tokens in real time as the agent responds |
| `agents_stream_iteration.py` | **Stream iteration** | Manually iterate over the stream object to process events in a loop |
| `agents_stream_eventhandler_with_functions.py` | **Streaming + Function calling** | Handle `tool_call` events mid-stream and invoke Python functions dynamically |
| `agents_stream_eventhandler_with_toolset.py` | **Streaming + Toolset** | Attach a pre-defined toolset to the agent and handle tool invocations via events |
| `agents_stream_file_search.py` | **Streaming + File search** | Stream responses grounded in uploaded documents using the file search tool |
| `agents_stream_with_base_override_eventhandler.py` | **Custom event handler override** | Subclass the base event handler to intercept and customise specific event types |

---

## Architecture

```
Azure AI Foundry Agent
        │
        ▼
  Stream context manager
        │
        ├── on_message_delta()     ← token-by-token text output
        ├── on_tool_call_created() ← tool invocation started
        ├── on_tool_call_delta()   ← tool arguments streaming
        ├── on_run_step_created()  ← new reasoning step
        ├── on_run_step_done()     ← step completed
        └── on_done()             ← stream complete
```

The `AgentEventHandler` (or your custom subclass) receives these events and lets you react in real time — print tokens, log tool calls, update a UI, or trigger downstream actions.

---

## Prerequisites

- Python 3.10+
- Azure subscription with **Azure AI Foundry** access
- An AI Foundry project with an agent deployment
- `pip install -r requirements.txt`

---

## Quickstart

### 1. Clone the repo

```bash
git clone https://github.com/praveen11singh/MSFoundryAgentStreamEvent.git
cd MSFoundryAgentStreamEvent
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

### 3. Configure environment

```bash
cp .env.example .env
```

```env
# .env
PROJECT_CONNECTION_STRING=<your-foundry-project-connection-string>
MODEL_DEPLOYMENT_NAME=<your-model-deployment-name>
```

> **Note:** Uses `DefaultAzureCredential` — no hardcoded keys. Run `az login` locally or assign a managed identity in production.

### 4. Run any pattern

```bash
# Basic streaming
python agents_stream_eventhandler.py

# Streaming with function calling
python agents_stream_eventhandler_with_functions.py

# Streaming with file search
python agents_stream_file_search.py

# Custom event handler override
python agents_stream_with_base_override_eventhandler.py
```

---

## Pattern Deep-Dives

### 1. Basic Event Handler — `agents_stream_eventhandler.py`

The foundation pattern. Creates an agent, opens a streaming run, and wires up an `AgentEventHandler` to print tokens as they arrive.

```python
with agents_client.runs.stream(
    thread_id=thread.id,
    agent_id=agent.id,
    event_handler=MyEventHandler(),
) as stream:
    stream.until_done()
```

---

### 2. Stream Iteration — `agents_stream_iteration.py`

Instead of an event handler, iterate over the stream directly for full control over event processing order.

```python
with agents_client.runs.stream(...) as stream:
    for event_type, event_data, _ in stream:
        if event_type == AgentStreamEvent.MESSAGE_DELTA:
            print(event_data.delta.content[0].text.value, end="")
```

---

### 3. Streaming + Function Calling — `agents_stream_eventhandler_with_functions.py`

Intercepts `RequiresAction` events during streaming to invoke Python functions and submit tool outputs back to the run — all without breaking the stream.

```python
def on_run_step_created(self, run_step):
    if run_step.type == "tool_calls":
        # invoke your function, submit output, continue streaming
```

---

### 4. Streaming + Toolset — `agents_stream_eventhandler_with_toolset.py`

Attaches a `ToolSet` (function definitions + callables) to the agent at creation time. The event handler resolves tool calls automatically from the registered toolset.

```python
toolset = ToolSet()
toolset.add(functions)
agent = agents_client.create_agent(
    model=MODEL_DEPLOYMENT_NAME,
    toolset=toolset,
)
```

---

### 5. Streaming + File Search — `agents_stream_file_search.py`

Uploads documents to a vector store, attaches it to the agent, then streams grounded responses with real-time citations.

```python
agent = agents_client.create_agent(
    tools=[FileSearchTool(vector_store_ids=[vector_store.id])],
)
```

---

### 6. Custom Base Override — `agents_stream_with_base_override_eventhandler.py`

Subclasses `BaseAgentEventHandler` to override specific lifecycle hooks — useful for custom logging, telemetry, UI updates, or error handling.

```python
class CustomEventHandler(AgentEventHandler):
    def on_message_delta(self, delta):
        # custom token handling — write to DB, push to websocket, etc.
        super().on_message_delta(delta)
```

---

## Project Structure

```
MSFoundryAgentStreamEvent/
├── agents_stream_eventhandler.py                    # Pattern 1 — basic streaming
├── agents_stream_iteration.py                       # Pattern 2 — manual iteration
├── agents_stream_eventhandler_with_functions.py     # Pattern 3 — function calling
├── agents_stream_eventhandler_with_toolset.py       # Pattern 4 — toolset
├── agents_stream_file_search.py                     # Pattern 5 — file search
├── agents_stream_with_base_override_eventhandler.py # Pattern 6 — custom override
├── utils/                                           # Shared helpers
├── assets/                                          # Sample files for file search
├── .env.example                                     # Environment variable template
├── requirements.txt
└── README.md
```

---

## When to Use Each Pattern

| Scenario | Recommended pattern |
|---|---|
| Simple chatbot with real-time output | Basic event handler |
| Fine-grained event processing pipeline | Stream iteration |
| Agent that calls your existing Python functions | Function calling |
| Agent with multiple pre-registered tools | Toolset |
| RAG agent grounded in uploaded documents | File search |
| Custom telemetry / observability / UI integration | Base override |

---

## Part of the Microsoft Foundry Open-Source Series

| Repo | Focus |
|---|---|
| [MSFoundryAgentMemory](https://github.com/praveen11singh/MSFoundryAgentMemory) | Ephemeral, contextual & persistent memory patterns |
| [MSFoundryFunctionCall](https://github.com/praveen11singh/MSFoundryFunctionCall) | Auto & explicit function calling |
| [MSFoundryEvaluation](https://github.com/praveen11singh/MSFoundryEvaluation) | Agent evaluation with azure-ai-evaluation |
| [MSFoundryRedTeam](https://github.com/praveen11singh/MSFoundryRedTeam) | Red team jailbreak probing & RAI evaluators |
| [MSFoundryModelRouter](https://github.com/praveen11singh/MSFoundryModelRouter) | Dynamic multi-model routing |
| **MSFoundryAgentStreamEvent** | Agent streaming & event handling ← you are here |

---

## Author

**Praveen Kumar**
Azure Solutions Architect & AI Engineer · Associate Consultant @ TCS
[LinkedIn]([https://www.linkedin.com/in/praveen11singh](https://www.linkedin.com/in/praveen-kumar-b52a1a1a0/)) 
---
