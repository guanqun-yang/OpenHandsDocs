<!-- Source: https://docs.openhands.dev/sdk/guides/llm-registry -->

# LLM Registry

A ready-to-run example is available here!

Use the LLM registry to manage multiple LLM providers and dynamically switch between models.

## Using the Registry

You can add LLMs to the registry using the `.add()` method and retrieve them later using the `.get()` method.

```python
main_llm = LLM(
    usage_id="agent",
    model=model,
    base_url=base_url,
    api_key=SecretStr(api_key),
)

# define the registry and add an LLM
llm_registry = LLMRegistry()
llm_registry.add(main_llm)

...

# retrieve the LLM by its usage ID
llm = llm_registry.get("agent")
```

## Ready-to-run Example

This example is available on GitHub: `examples/01_standalone_sdk/05_use_llm_registry.py`

```python
import os
from pydantic import SecretStr
from openhands.sdk import (
    LLM,
    Agent,
    Conversation,
    Event,
    LLMConvertibleEvent,
    LLMRegistry,
    Message,
    TextContent,
    get_logger,
)
from openhands.sdk.tool import Tool
from openhands.tools.terminal import TerminalTool

logger = get_logger(__name__)

# Configure LLM using LLMRegistry
api_key = os.getenv("LLM_API_KEY")
assert api_key is not None, "LLM_API_KEY environment variable is not set."
model = os.getenv("LLM_MODEL", "anthropic/claude-sonnet-4-5-20250929")
base_url = os.getenv("LLM_BASE_URL")

# Create LLM instance
main_llm = LLM(
    usage_id="agent",
    model=model,
    base_url=base_url,
    api_key=SecretStr(api_key),
)

# Create LLM registry and add the LLM
llm_registry = LLMRegistry()
llm_registry.add(main_llm)

# Get LLM from registry
llm = llm_registry.get("agent")

# Tools
cwd = os.getcwd()
tools = [Tool(name=TerminalTool.name)]

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

conversation.send_message("Please echo 'Hello!'")
conversation.run()

print("=" * 100)
print("Conversation finished. Got the following LLM messages:")
for i, message in enumerate(llm_messages):
    print(f"Message {i}: {str(message)[:200]}")

print("=" * 100)
print(f"LLM Registry usage IDs: {llm_registry.list_usage_ids()}")

# Demonstrate getting the same LLM instance from registry
same_llm = llm_registry.get("agent")
print(f"Same LLM instance: {llm is same_llm}")

# Demonstrate requesting a completion directly from an LLM
resp = llm.completion(
    messages=[
        Message(role="user", content=[TextContent(text="Say hello in one word.")])
    ]
)

# Access the response content via OpenHands LLMResponse
msg = resp.message
texts = [c.text for c in msg.content if isinstance(c, TextContent)]
print(f"Direct completion response: {texts[0] if texts else str(msg)}")

# Report cost
cost = llm.metrics.accumulated_cost
print(f"EXAMPLE_COST: {cost}")
```

### Running the Example

You can run the example code as-is. The model name should follow the LiteLLM convention: `provider/model_name` (e.g., `anthropic/claude-sonnet-4-5-20250929`, `openai/gpt-4o`).

The `LLM_API_KEY` should be the API key for your chosen provider.

ChatGPT Plus/Pro subscribers: You can use `LLM.subscription_login()` to authenticate with your ChatGPT account and access Codex models without consuming API credits. See the LLM Subscriptions guide for details.

## Next Steps

- LLM Routing - Automatically route to different models
- LLM Metrics - Track token usage and costs
