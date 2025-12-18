# LangGraph + BAML Integration

Using BAML for type-safe extraction within LangGraph workflows.

## Why BAML + LangGraph

| LangGraph Provides | BAML Provides |
|-------------------|---------------|
| Graph orchestration | Type-safe LLM calls |
| State management | Schema validation |
| Conditional routing | Fuzzy JSON parsing |
| Cycles and loops | Retry/fallback handling |

## Integration Pattern

### State Definition

```python
from typing import TypedDict
from baml_client.types import ToolCall, ExtractedData

class AgentState(TypedDict):
    messages: list[dict]
    extracted: ExtractedData | None
    tool_calls: list[ToolCall]
    iteration: int
```

### BAML for Node Functions

```python
from langgraph.graph import StateGraph
from baml_client import b

def extract_node(state: AgentState) -> AgentState:
    """Use BAML for structured extraction."""
    result = b.ExtractData(state["messages"][-1]["content"])
    return {"extracted": result}

def route_node(state: AgentState) -> AgentState:
    """Use BAML union types for tool selection."""
    tools = b.SelectTools(state["messages"][-1]["content"])
    return {"tool_calls": tools}

# Build graph
graph = StateGraph(AgentState)
graph.add_node("extract", extract_node)
graph.add_node("route", route_node)
```

### Conditional Routing with BAML Types

```python
from baml_client.types import GetWeather, SearchWeb, Calculator

def route_by_tool(state: AgentState) -> str:
    """Route based on BAML union type."""
    if not state["tool_calls"]:
        return "end"

    tool = state["tool_calls"][0]
    if isinstance(tool, GetWeather):
        return "weather_node"
    elif isinstance(tool, SearchWeb):
        return "search_node"
    elif isinstance(tool, Calculator):
        return "calc_node"
    return "end"

graph.add_conditional_edges("route", route_by_tool)
```

## BAML Schema for LangGraph Agents

### Tool Selection Schema

```baml
class GetWeather {
  type "get_weather"
  location string
  @@stream.done
}

class SearchWeb {
  type "search"
  query string
  @@stream.done
}

class MessageToUser {
  type "message"
  content string @stream.with_state
}

class Resume {
  type "resume"
  @@stream.done
}

function SelectAction(
  state: AgentState,
  query: string
) -> (GetWeather | SearchWeb | MessageToUser | Resume)[] {
  client GPT4
  prompt #"
    {{ Instructions() }}
    Current state: {{ state }}
    Query: {{ query }}
    {{ ctx.output_format }}
  "#
}
```

### State Extraction

```baml
class AgentState {
  context string?
  last_tool_result string?
  iteration int
}

function ParseState(messages: string[]) -> AgentState {
  client GPT4
  prompt #"
    Extract agent state from conversation:
    {% for msg in messages %}
    {{ msg }}
    {% endfor %}
    {{ ctx.output_format }}
  "#
}
```

## Best Practices

1. **BAML for LLM Nodes** - Use BAML functions for any node that calls an LLM
2. **Type Guards for Routing** - Use `isinstance()` with BAML union types
3. **State Types Aligned** - Keep LangGraph state types aligned with BAML types
4. **Stream for UX** - Use `b.stream.FunctionName()` for long-running extractions

## Example Projects

See agent patterns in:
- `baml-examples/python-chatbot/server/baml_src/agent.baml` - Full agent with tool selection and state
- `baml-examples/form-filler/baml_src/form_filler.baml` - Multi-turn form extraction with union actions

## Streaming in LangGraph

```python
async def streaming_node(state: AgentState):
    """Stream BAML responses through LangGraph."""
    stream = b.stream.GenerateResponse(state["messages"])

    async for partial in stream:
        # Yield partial results for real-time UI
        yield {"partial_response": partial.content}

    final = await stream.get_final_response()
    return {"response": final}
```
