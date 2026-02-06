<!-- Source: https://docs.openhands.dev/sdk/guides/llm-reasoning -->

# Reasoning

View your agent's internal reasoning process for debugging, transparency, and understanding decision-making. This guide demonstrates two provider-specific approaches:

- Anthropic Extended Thinking - Claude's thinking blocks for complex reasoning
- OpenAI Reasoning via Responses API - GPT's reasoning effort parameter

## Anthropic Extended Thinking

A ready-to-run example is available here!

Anthropic's Claude models support extended thinking, which allows you to access the model's internal reasoning process through thinking blocks. This is useful for understanding how Claude approaches complex problems step-by-step.

### How It Works

The key to accessing thinking blocks is to register a callback that checks for `thinking_blocks` in LLM messages:

```python
def show_thinking(event: Event):
    if isinstance(event, LLMConvertibleEvent):
        message = event.to_llm_message()
        if hasattr(message, "thinking_blocks") and message.thinking_blocks:
            print(f"🧠 Found {len(message.thinking_blocks)} thinking blocks")
            for block in message.thinking_blocks:
                if isinstance(block, RedactedThinkingBlock):
                    print(f"Redacted: {block.data}")
                elif isinstance(block, ThinkingBlock):
                    print(f"Thinking: {block.thinking}")

conversation = Conversation(agent=agent, callbacks=[show_thinking])
```

### Understanding Thinking Blocks

Claude uses thinking blocks to reason through complex problems step-by-step. There are two types:

- **ThinkingBlock**: Contains the full reasoning text from Claude's internal thought process
- **RedactedThinkingBlock**: Contains redacted or summarized thinking data

By registering a callback with your conversation, you can intercept and display these thinking blocks in real-time, giving you insight into how Claude is approaching the problem.

### Ready-to-run Example - Anthropic

This example is available on GitHub: `examples/01_standalone_sdk/22_anthropic_thinking.py`

```python
"""Example demonstrating Anthropic's extended thinking feature with thinking blocks."""

import os
from pydantic import SecretStr
from openhands.sdk import (
    LLM,
    Agent,
    Conversation,
    Event,
    LLMConvertibleEvent,
    RedactedThinkingBlock,
    ThinkingBlock,
)
from openhands.sdk.tool import Tool
from openhands.tools.terminal import TerminalTool

# Configure LLM for Anthropic Claude with extended thinking
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

# Setup agent with bash tool
agent = Agent(llm=llm, tools=[Tool(name=TerminalTool.name)])

# Callback to display thinking blocks
def show_thinking(event: Event):
    if isinstance(event, LLMConvertibleEvent):
        message = event.to_llm_message()
        if hasattr(message, "thinking_blocks") and message.thinking_blocks:
            print(f"\n🧠 Found {len(message.thinking_blocks)} thinking blocks")
            for i, block in enumerate(message.thinking_blocks):
                if isinstance(block, RedactedThinkingBlock):
                    print(f" Block {i + 1}: {block.data}")
                elif isinstance(block, ThinkingBlock):
                    print(f" Block {i + 1}: {block.thinking}")

conversation = Conversation(
    agent=agent,
    callbacks=[show_thinking],
    workspace=os.getcwd()
)

conversation.send_message(
    "Calculate compound interest for $10,000 at 5% annually, "
    "compounded quarterly for 3 years. Show your work.",
)
conversation.run()

conversation.send_message(
    "Now, write that number to RESULTS.txt.",
)
conversation.run()

print("✅ Done!")

# Report cost
cost = llm.metrics.accumulated_cost
print(f"EXAMPLE_COST: {cost}")
```

### Running the Example

You can run the example code as-is. The model name should follow the LiteLLM convention: `provider/model_name` (e.g., `anthropic/claude-sonnet-4-5-20250929`, `openai/gpt-4o`).

The `LLM_API_KEY` should be the API key for your chosen provider.

#### Bring-your-own provider key

```bash
export LLM_API_KEY="your-api-key"
export LLM_MODEL="anthropic/claude-sonnet-4-5-20250929" # or openai/gpt-4o, etc.
cd software-agent-sdk
uv run python examples/01_standalone_sdk/22_anthropic_thinking.py
```

#### OpenHands Cloud

ChatGPT Plus/Pro subscribers: You can use `LLM.subscription_login()` to authenticate with your ChatGPT account and access Codex models without consuming API credits. See the LLM Subscriptions guide for details.

## OpenAI Reasoning via Responses API

A ready-to-run example is available here!

OpenAI's latest models (e.g., GPT-5, GPT-5-Codex) support a Responses API that provides access to the model's reasoning process. By setting the `reasoning_effort` parameter, you can control how much reasoning the model performs and access those reasoning traces.

### How It Works

Configure the LLM with the `reasoning_effort` parameter to enable reasoning:

```python
llm = LLM(
    model="openhands/gpt-5-codex",
    api_key=SecretStr(api_key),
    base_url=base_url,
    # Enable reasoning with effort level
    reasoning_effort="high",
)
```

The `reasoning_effort` parameter can be set to `"none"`, `"low"`, `"medium"`, or `"high"` to control the amount of reasoning performed by the model.

Then capture reasoning traces in your callback:

```python
def conversation_callback(event: Event):
    if isinstance(event, LLMConvertibleEvent):
        msg = event.to_llm_message()
        llm_messages.append(msg)
```

### Understanding Reasoning Traces

The OpenAI Responses API provides reasoning traces that show how the model approached the problem. These traces are available in the LLM messages and can be inspected to understand the model's decision-making process. Unlike Anthropic's thinking blocks, OpenAI's reasoning is more tightly integrated with the response generation process.

### Ready-to-run Example - OpenAI

This example is available on GitHub: `examples/01_standalone_sdk/23_responses_reasoning.py`

```python
"""
Example: Responses API path via LiteLLM in a Real Agent Conversation
- Runs a real Agent/Conversation to verify /responses path works
- Demonstrates rendering of Responses reasoning within normal conversation events
"""

from __future__ import annotations

import os
from pydantic import SecretStr
from openhands.sdk import (
    Conversation,
    Event,
    LLMConvertibleEvent,
    get_logger,
)
from openhands.sdk.llm import LLM
from openhands.tools.preset.default import get_default_agent

logger = get_logger(__name__)

api_key = os.getenv("LLM_API_KEY") or os.getenv("OPENAI_API_KEY")
assert api_key, "Set LLM_API_KEY or OPENAI_API_KEY in your environment."

model = "openhands/gpt-5-mini-2025-08-07"  # Use a model that supports Responses API
base_url = os.getenv("LLM_BASE_URL")

llm = LLM(
    model=model,
    api_key=SecretStr(api_key),
    base_url=base_url,
    # Responses-path options
    reasoning_effort="high",
    # Logging / behavior tweaks
    log_completions=False,
    usage_id="agent",
)

print("\n=== Agent Conversation using /responses path ===")

agent = get_default_agent(
    llm=llm,
    cli_mode=True,  # disable browser tools for env simplicity
)

llm_messages = []  # collect raw LLM-convertible messages for inspection

def conversation_callback(event: Event):
    if isinstance(event, LLMConvertibleEvent):
        llm_messages.append(event.to_llm_message())

conversation = Conversation(
    agent=agent,
    callbacks=[conversation_callback],
    workspace=os.getcwd(),
)

# Keep the tasks short for demo purposes
conversation.send_message("Read the repo and write one fact into FACTS.txt.")
conversation.run()

conversation.send_message("Now delete FACTS.txt.")
conversation.run()

print("=" * 100)
print("Conversation finished. Got the following LLM messages:")
for i, message in enumerate(llm_messages):
    ms = str(message)
    print(f"Message {i}: {ms[:200]}{'...' if len(ms) > 200 else ''}")

# Report cost
cost = llm.metrics.accumulated_cost
print(f"EXAMPLE_COST: {cost}")
```

### Running the Example

You can run the example code as-is. The model name should follow the LiteLLM convention: `provider/model_name` (e.g., `anthropic/claude-sonnet-4-5-20250929`, `openai/gpt-4o`).

The `LLM_API_KEY` should be the API key for your chosen provider.

#### Bring-your-own provider key

```bash
export LLM_API_KEY="your-api-key"
export LLM_MODEL="anthropic/claude-sonnet-4-5-20250929" # or openai/gpt-4o, etc.
cd software-agent-sdk
uv run python examples/01_standalone_sdk/23_responses_reasoning.py
```

#### OpenHands Cloud

ChatGPT Plus/Pro subscribers: You can use `LLM.subscription_login()` to authenticate with your ChatGPT account and access Codex models without consuming API credits. See the LLM Subscriptions guide for details.

## Use Cases

- **Debugging**: Understand why the agent made specific decisions or took certain actions.
- **Transparency**: Show users how the AI arrived at its conclusions.
- **Quality Assurance**: Identify flawed reasoning patterns or logic errors.
- **Learning**: Study how models approach complex problems.

## Next Steps

- Interactive Terminal - Display reasoning in real-time
- LLM Metrics - Track token usage and performance
- Custom Tools - Add specialized capabilities
