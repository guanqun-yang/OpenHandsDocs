<!-- Source: https://docs.openhands.dev/sdk/guides/agent-interactive-terminal -->

# Interactive Terminal

The BashTool provides agents with the ability to interact with terminal applications that require back-and-forth communication, such as Python's interactive mode, ipython, database CLIs, and other REPL environments. This enables agents to execute commands within these interactive sessions, receive output, and send follow-up commands based on the results.

A ready-to-run example is available [here](#ready-to-run-example)!

## How It Works

```python
cwd = os.getcwd()
register_tool("BashTool", BashTool)
tools = [
    Tool(
        name="BashTool",
        params={"no_change_timeout_seconds": 3},
    )
]
```

The BashTool is configured with a `no_change_timeout_seconds` parameter that determines how long to wait for terminal updates before sending the output back to the agent. In the example above, the agent should:

1. Enters Python's interactive mode by running `python3`
2. Executes Python code to get the current time
3. Exits the Python interpreter

The BashTool maintains the session state throughout these interactions, allowing the agent to send multiple commands within the same terminal session.

Review the [BashTool and terminal source code](https://github.com/OpenHands/software-agent-sdk/tree/main/openhands-tools/openhands/tools) to better understand how the interactive session is configured and managed.

## Ready-to-run Example

This example is available on GitHub: [examples/01_standalone_sdk/06_interactive_terminal_w_reasoning.py](https://github.com/OpenHands/software-agent-sdk/blob/main/examples/01_standalone_sdk/06_interactive_terminal_w_reasoning.py)

```python
import os

from pydantic import SecretStr

from openhands.sdk import (
    LLM,
    Agent,
    Conversation,
    Event,
    LLMConvertibleEvent,
    get_logger,
)
from openhands.sdk.tool import Tool
from openhands.tools.terminal import TerminalTool

logger = get_logger(__name__)

# Configure LLM
api_key = os.getenv("LLM_API_KEY")
assert api_key is not None, "LLM_API_KEY environment variable is not set."
model = os.getenv("LLM_MODEL", "anthropic/claude-sonnet-4-5-20250929")
base_url = os.getenv("LLM_BASE_URL")

llm = LLM(
    usage_id="agent",
    model=model,
    base_url=base_url,
    api_key=SecretStr(api_key),
)

# Tools
cwd = os.getcwd()
tools = [
    Tool(
        name=TerminalTool.name,
        params={"no_change_timeout_seconds": 3},
    )
]

# Agent
agent = Agent(llm=llm, tools=tools)

llm_messages = []  # collect raw LLM messages

def conversation_callback(event: Event):
    if isinstance(event, LLMConvertibleEvent):
        llm_messages.append(event.to_llm_message())

conversation = Conversation(
    agent=agent,
    callbacks=[conversation_callback],
    workspace=cwd
)

conversation.send_message(
    "Enter python interactive mode by directly running `python3`, then tell me "
    "the current time, and exit python interactive mode."
)
conversation.run()

print("=" * 100)
print("Conversation finished. Got the following LLM messages:")
for i, message in enumerate(llm_messages):
    print(f"Message {i}: {str(message)[:200]}")
```

You can run the example code as-is.

**Note:** The model name should follow the [LiteLLM convention](https://models.litellm.ai/): `provider/model_name` (e.g., `anthropic/claude-sonnet-4-5-20250929`, `openai/gpt-4o`). The `LLM_API_KEY` should be the API key for your chosen provider.

### Running the Example

#### Bring-your-own provider key

```bash
export LLM_API_KEY="your-api-key"
export LLM_MODEL="anthropic/claude-sonnet-4-5-20250929"  # or openai/gpt-4o, etc.
cd software-agent-sdk
uv run python examples/01_standalone_sdk/06_interactive_terminal_w_reasoning.py
```

#### OpenHands Cloud

```bash
cd software-agent-sdk
uv run python examples/01_standalone_sdk/06_interactive_terminal_w_reasoning.py
```

**Tip:** ChatGPT Plus/Pro subscribers: You can use `LLM.subscription_login()` to authenticate with your ChatGPT account and access Codex models without consuming API credits. See the [LLM Subscriptions guide](/sdk/guides/llm-subscriptions) for details.

## Next Steps

- [Custom Tools](/sdk/guides/custom-tools) - Create your own tools for specific use cases
