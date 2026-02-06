<!-- Source: https://docs.openhands.dev/sdk/guides/agent-custom -->

# Creating Custom Agent

This guide demonstrates how to create custom agents tailored for specific use cases. Using the planning agent as a concrete example, you'll learn how to design specialized agents with custom tool sets, system prompts, and configurations that optimize performance for particular workflows.

This example is available on GitHub: [examples/01_standalone_sdk/24_planning_agent_workflow.py](https://github.com/OpenHands/software-agent-sdk/blob/main/examples/01_standalone_sdk/24_planning_agent_workflow.py)

## Overview

The example showcases a two-phase workflow where a custom planning agent (with read-only tools) analyzes tasks and creates structured plans, followed by an execution agent that implements those plans with full editing capabilities.

### Task

Create a Python web scraper that extracts article titles and URLs from a news website, handles rate limiting, and saves results to JSON.

## Two-Phase Workflow

### Phase 1: Planning

The planning agent analyzes the task and creates a detailed implementation plan:

```python
planning_agent = get_planning_agent(llm=llm)
planning_conversation = Conversation(
    agent=planning_agent,
    workspace=str(workspace_dir),
)

planning_conversation.send_message(
    f"Please analyze this web scraping task and create a detailed "
    f"implementation plan:\n\n{task}"
)
planning_conversation.run()
```

### Phase 2: Execution

The execution agent implements the plan with full editing capabilities:

```python
execution_agent = get_default_agent(llm=llm, cli_mode=True)
execution_conversation = Conversation(
    agent=execution_agent,
    workspace=str(workspace_dir),
)

execution_conversation.send_message(execution_prompt)
execution_conversation.run()
```

## Anatomy of a Custom Agent

The planning agent demonstrates the two key components for creating specialized agents:

### 1. Custom Tool Selection

Choose tools that match your agent's specific role. Here's how the planning agent defines its tools:

```python
def register_planning_tools() -> None:
    """Register the planning agent tools."""
    from openhands.tools.glob import GlobTool
    from openhands.tools.grep import GrepTool
    from openhands.tools.planning_file_editor import PlanningFileEditorTool

    register_tool("GlobTool", GlobTool)
    logger.debug("Tool: GlobTool registered.")
    register_tool("GrepTool", GrepTool)
    logger.debug("Tool: GrepTool registered.")
    register_tool("PlanningFileEditorTool", PlanningFileEditorTool)
    logger.debug("Tool: PlanningFileEditorTool registered.")

def get_planning_tools() -> list[Tool]:
    """Get the planning agent tool specifications."""
    register_planning_tools()
    return [
        Tool(name="GlobTool"),
        Tool(name="GrepTool"),
        Tool(name="PlanningFileEditorTool"),
    ]
```

The planning agent uses:
- **GlobTool**: For discovering files and directories matching patterns
- **GrepTool**: For searching specific content across files
- **PlanningFileEditorTool**: For writing structured plans to PLAN.md only

This read-only approach (except for PLAN.md) keeps the agent focused on analysis without implementation distractions.

### 2. System Prompt Customization

Custom agents can use specialized system prompts to guide behavior. The planning agent uses `system_prompt_planning.j2` with injected plan structure that enforces:

- **Objective**: Clear goal statement
- **Context Summary**: Relevant system components and constraints
- **Approach Overview**: High-level strategy and rationale
- **Implementation Steps**: Detailed step-by-step execution plan
- **Testing and Validation**: Verification methods and success criteria

## Implementation Reference

For a complete implementation example showing all these components working together, refer to the planning agent preset source code.

## Running the Example

You can run the example code as-is.

**Note:** The model name should follow the [LiteLLM convention](https://models.litellm.ai/): `provider/model_name` (e.g., `anthropic/claude-sonnet-4-5-20250929`, `openai/gpt-4o`). The `LLM_API_KEY` should be the API key for your chosen provider.

### Bring-your-own provider key

```bash
export LLM_API_KEY="your-api-key"
export LLM_MODEL="anthropic/claude-sonnet-4-5-20250929"  # or openai/gpt-4o, etc.
cd software-agent-sdk
uv run python examples/01_standalone_sdk/24_planning_agent_workflow.py
```

### OpenHands Cloud

```bash
cd software-agent-sdk
uv run python examples/01_standalone_sdk/24_planning_agent_workflow.py
```

**Tip:** ChatGPT Plus/Pro subscribers: You can use `LLM.subscription_login()` to authenticate with your ChatGPT account and access Codex models without consuming API credits. See the [LLM Subscriptions guide](/sdk/guides/llm-subscriptions) for details.

## Next Steps

- [Custom Tools](/sdk/guides/custom-tools) - Create specialized tools for your use case
- [Context Condenser](/sdk/guides/context-condenser) - Optimize context management
- [MCP Integration](/sdk/guides/mcp) - Add MCP
