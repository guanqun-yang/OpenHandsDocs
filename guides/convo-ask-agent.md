<!-- Source: https://docs.openhands.dev/sdk/guides/convo-ask-agent -->

# Ask Agent Questions

Use `ask_agent()` to get quick responses from the agent about the current conversation state without interrupting the main execution flow.

A ready-to-run example is available [here](#ready-to-run-example)!

## Key Features

### Context-Aware Responses

The agent has access to the full conversation history when answering questions:

```python
# Agent can reference what it has done so far
response = conversation.ask_agent(
    "Summarize the activity so far in 1 sentence."
)
print(f"Response: {response}")
```

### Non-Intrusive Operation

Questions don't interrupt the main conversation flow - they're processed separately:

```python
# Start main conversation
thread = threading.Thread(target=conversation.run)
thread.start()

# Ask questions without affecting main execution
response = conversation.ask_agent("How's the progress?")
```

### Works During and After Execution

You can ask questions while the agent is running or after it has completed:

```python
# During execution
time.sleep(2)  # Let agent start working
response1 = conversation.ask_agent("Have you finished running?")

# After completion
thread.join()
response2 = conversation.ask_agent("What did you accomplish?")
```

## Use Cases

- **Progress Monitoring**: Check on long-running tasks
- **Status Updates**: Get real-time information about agent activities
- **User Interfaces**: Provide sidebar information in chat applications

## Ready-to-run Example

This example is available on GitHub: [examples/01_standalone_sdk/28_ask_agent_example.py](https://github.com/OpenHands/software-agent-sdk/blob/main/examples/01_standalone_sdk/28_ask_agent_example.py)

Example demonstrating the ask_agent functionality for getting sidebar replies from the agent for a running conversation. This example shows how to use `ask_agent()` to get quick responses from the agent about the current conversation state without interrupting the main execution flow.

### Running the Example

You can run the example code as-is.

**Note:** The model name should follow the [LiteLLM convention](https://models.litellm.ai/): `provider/model_name` (e.g., `anthropic/claude-sonnet-4-5-20250929`, `openai/gpt-4o`). The `LLM_API_KEY` should be the API key for your chosen provider.

#### Bring-your-own provider key

```bash
export LLM_API_KEY="your-api-key"
export LLM_MODEL="anthropic/claude-sonnet-4-5-20250929"  # or openai/gpt-4o, etc.
cd software-agent-sdk
uv run python examples/01_standalone_sdk/28_ask_agent_example.py
```

#### OpenHands Cloud

```bash
cd software-agent-sdk
uv run python examples/01_standalone_sdk/28_ask_agent_example.py
```

**Tip:** ChatGPT Plus/Pro subscribers: You can use `LLM.subscription_login()` to authenticate with your ChatGPT account and access Codex models without consuming API credits. See the [LLM Subscriptions guide](/sdk/guides/llm-subscriptions) for details.

## Next Steps

- [Send Messages While Running](/sdk/guides/convo-send-message-while-running) - Interrupt and redirect agent execution
- [Pause and Resume](/sdk/guides/convo-pause-and-resume) - Control execution flow
- [Custom Visualizers](/sdk/guides/convo-custom-visualizer) - Monitor conversation progress
