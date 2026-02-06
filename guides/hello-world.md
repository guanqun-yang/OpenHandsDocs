<!-- Source: https://docs.openhands.dev/sdk/guides/hello-world -->

# Hello World

The simplest possible OpenHands agent - configure an LLM, create an agent, and complete a task.

A ready-to-run example is available [here](#ready-to-run-example)!

## Your First Agent

This is the most basic example showing how to set up and run an OpenHands agent.

### 1. LLM Configuration

Configure the language model that will power your agent:

```python
llm = LLM(
    model=model,
    api_key=SecretStr(api_key),
    base_url=base_url,  # Optional
    service_id="agent"
)
```

### 2. Select an Agent

Use the preset agent with common built-in tools:

```python
agent = get_default_agent(llm=llm, cli_mode=True)
```

The default agent includes `BashTool`, `FileEditorTool`, etc.

**Tip:** For the complete list of available tools see the [tools package source code](https://github.com/OpenHands/software-agent-sdk/tree/main/openhands-tools/openhands/tools).

### 3. Start a Conversation

Start a conversation to manage the agent's lifecycle:

```python
conversation = Conversation(agent=agent, workspace=cwd)
conversation.send_message(
    "Write 3 facts about the current project into FACTS.txt."
)
conversation.run()
```

### 4. Expected Behavior

When you run this example:
- The agent analyzes the current directory
- Gathers information about the project
- Creates `FACTS.txt` with 3 relevant facts
- Completes and exits

Example output file:

```
FACTS.txt
---------
1. This is a Python project using the OpenHands Software Agent SDK.
2. The project includes examples demonstrating various agent capabilities.
3. The SDK provides tools for file manipulation, bash execution, and more.
```

## Ready-to-run Example

**Note:** This example is available on GitHub: [examples/01_standalone_sdk/01_hello_world.py](https://github.com/OpenHands/software-agent-sdk/blob/main/examples/01_standalone_sdk/01_hello_world.py)

```python
import os

from openhands.sdk import LLM, Agent, Conversation, Tool
from openhands.tools.file_editor import FileEditorTool
from openhands.tools.task_tracker import TaskTrackerTool
from openhands.tools.terminal import TerminalTool

llm = LLM(
    model=os.getenv(
        "LLM_MODEL", "anthropic/claude-sonnet-4-5-20250929"
    ),
    api_key=os.getenv("LLM_API_KEY"),
    base_url=os.getenv("LLM_BASE_URL", None),
)

agent = Agent(
    llm=llm,
    tools=[
        Tool(name=TerminalTool.name),
        Tool(name=FileEditorTool.name),
        Tool(name=TaskTrackerTool.name),
    ],
)

cwd = os.getcwd()
conversation = Conversation(agent=agent, workspace=cwd)
conversation.send_message(
    "Write 3 facts about the current project into FACTS.txt."
)
conversation.run()

print("All done!")
```

You can run the example code as-is.

**Note:** The model name should follow the [LiteLLM convention](https://models.litellm.ai/): `provider/model_name` (e.g., `anthropic/claude-sonnet-4-5-20250929`, `openai/gpt-4o`). The `LLM_API_KEY` should be the API key for your chosen provider.

### Running the Example

#### Bring-your-own provider key

```bash
export LLM_API_KEY="your-api-key"
export LLM_MODEL="anthropic/claude-sonnet-4-5-20250929"  # or openai/gpt-4o, etc.
cd software-agent-sdk
uv run python examples/01_standalone_sdk/01_hello_world.py
```

#### OpenHands Cloud

```bash
cd software-agent-sdk
uv run python examples/01_standalone_sdk/01_hello_world.py
```

**Tip:** ChatGPT Plus/Pro subscribers: You can use `LLM.subscription_login()` to authenticate with your ChatGPT account and access Codex models without consuming API credits. See the [LLM Subscriptions guide](/sdk/guides/llm-subscriptions) for details.

## Next Steps

- [Custom Tools](/sdk/guides/custom-tools) - Create custom tools for specialized needs
- [Model Context Protocol (MCP)](/sdk/guides/mcp) - Integrate external MCP servers
- [Security Analyzer](/sdk/guides/security) - Add security validation to tool usage
