<!-- Source: https://docs.openhands.dev/sdk/guides/agent-delegation -->

# Sub-Agent Delegation

Agent delegation allows a main agent to spawn multiple sub-agents and delegate tasks to them for parallel processing. Each sub-agent runs independently with its own conversation context and returns results that the main agent can consolidate and process further.

A ready-to-run example is available [here](#ready-to-run-example)!

## Overview

This pattern is useful when:
- Breaking down complex problems into independent subtasks
- Processing multiple related tasks in parallel
- Separating concerns between different specialized sub-agents
- Improving throughput for parallelizable work

## How It Works

The delegation system consists of two main operations:

### 1. Spawning Sub-Agents

Before delegating work, the agent must first spawn sub-agents with meaningful identifiers:

```python
# Agent uses the delegate tool to spawn sub-agents
{
    "command": "spawn",
    "ids": ["lodging", "activities"]
}
```

Each spawned sub-agent:
- Gets a unique identifier that the agent specifies (e.g., "lodging", "activities")
- Inherits the same LLM configuration as the parent agent
- Operates in the same workspace as the main agent
- Maintains its own independent conversation context

### 2. Delegating Tasks

Once sub-agents are spawned, the agent can delegate tasks to them:

```python
# Agent uses the delegate tool to assign tasks
{
    "command": "delegate",
    "tasks": {
        "lodging": "Find the best budget-friendly areas to stay in London",
        "activities": "List top 5 must-see attractions and hidden gems in London"
    }
}
```

The delegate operation:
- Runs all sub-agent tasks in parallel using threads
- Blocks until all sub-agents complete their work
- Returns a single consolidated observation with all results
- Handles errors gracefully and reports them per sub-agent

## Setting Up the DelegateTool

### 1. Register the Tool

```python
from openhands.sdk.tool import register_tool
from openhands.tools.delegate import DelegateTool

register_tool("DelegateTool", DelegateTool)
```

### 2. Add to Agent Tools

```python
from openhands.sdk import Tool
from openhands.tools.preset.default import get_default_tools

tools = get_default_tools(enable_browser=False)
tools.append(Tool(name="DelegateTool"))
agent = Agent(llm=llm, tools=tools)
```

### 3. Configure Maximum Sub-Agents (Optional)

The user can limit the maximum number of concurrent sub-agents:

```python
from openhands.tools.delegate import DelegateTool

class CustomDelegateTool(DelegateTool):
    @classmethod
    def create(cls, conv_state, max_children: int = 3):
        # Only allow up to 3 sub-agents
        return super().create(conv_state, max_children=max_children)

register_tool("DelegateTool", CustomDelegateTool)
```

## Tool Commands

### spawn

Initialize sub-agents with meaningful identifiers.

**Parameters:**
- `command`: "spawn"
- `ids`: List of string identifiers (e.g., ["research", "implementation", "testing"])

**Returns:** A message indicating the sub-agents were successfully spawned.

**Example:**
```python
{
    "command": "spawn",
    "ids": ["research", "implementation", "testing"]
}
```

### delegate

Send tasks to specific sub-agents and wait for results.

**Parameters:**
- `command`: "delegate"
- `tasks`: Dictionary mapping sub-agent IDs to task descriptions

**Returns:** A consolidated message containing all results from the sub-agents.

**Example:**
```python
{
    "command": "delegate",
    "tasks": {
        "research": "Find best practices for async code",
        "implementation": "Refactor the MyClass class",
        "testing": "Write unit tests for the refactored code"
    }
}
```

## Ready-to-run Example

This example is available on GitHub: [examples/01_standalone_sdk/25_agent_delegation.py](https://github.com/OpenHands/software-agent-sdk/blob/main/examples/01_standalone_sdk/25_agent_delegation.py)

The example demonstrates agent delegation where a main agent delegates tasks to sub-agents for parallel processing. Each sub-agent runs independently and returns its results to the main agent, which then merges both analyses into a single consolidated report.

### Running the Example

You can run the example code as-is.

**Note:** The model name should follow the [LiteLLM convention](https://models.litellm.ai/): `provider/model_name` (e.g., `anthropic/claude-sonnet-4-5-20250929`, `openai/gpt-4o`). The `LLM_API_KEY` should be the API key for your chosen provider.

#### Bring-your-own provider key

```bash
export LLM_API_KEY="your-api-key"
export LLM_MODEL="anthropic/claude-sonnet-4-5-20250929"  # or openai/gpt-4o, etc.
cd software-agent-sdk
uv run python examples/01_standalone_sdk/25_agent_delegation.py
```

#### OpenHands Cloud

```bash
cd software-agent-sdk
uv run python examples/01_standalone_sdk/25_agent_delegation.py
```

**Tip:** ChatGPT Plus/Pro subscribers: You can use `LLM.subscription_login()` to authenticate with your ChatGPT account and access Codex models without consuming API credits. See the [LLM Subscriptions guide](/sdk/guides/llm-subscriptions) for details.

## Key Features

### User-Defined Agent Types

You can define custom agent types with specialized skills and system messages:

```python
def create_lodging_planner(llm: LLM) -> Agent:
    """Create a lodging planner focused on London stays."""
    skills = [
        Skill(
            name="lodging_planning",
            content=(
                "You specialize in finding great places to stay in London. "
                "Provide 3-4 hotel recommendations with neighborhoods, quick pros/cons, "
                "and notes on transit convenience. Keep options varied by budget."
            ),
        )
    ]
    return Agent(
        llm=llm,
        tools=[],
        agent_context=AgentContext(
            skills=skills,
            system_message_suffix="Focus only on London lodging recommendations.",
        ),
    )
```

### Registering Custom Agent Types

```python
from openhands.tools.delegate import register_agent

register_agent(
    name="lodging_planner",
    factory_func=create_lodging_planner,
    description="Finds London lodging options with transit-friendly picks.",
)
```

## Next Steps

- [Custom Tools](/sdk/guides/custom-tools) - Create specialized tools
- [Context Condenser](/sdk/guides/context-condenser) - Optimize context management
