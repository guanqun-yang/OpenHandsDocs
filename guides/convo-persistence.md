<!-- Source: https://docs.openhands.dev/sdk/guides/convo-persistence -->

# Persistence

Save conversation state to disk and restore it later for long-running or multi-session workflows.

A ready-to-run example is available [here](#ready-to-run-example)!

## How to use Persistence

### Saving State

Create a conversation with a unique ID to enable persistence:

```python
import uuid

conversation_id = uuid.uuid4()
persistence_dir = "./.conversations"

conversation = Conversation(
    agent=agent,
    callbacks=[conversation_callback],
    workspace=cwd,
    persistence_dir=persistence_dir,
    conversation_id=conversation_id,
)

conversation.send_message("Start long task")
conversation.run()
# State automatically saved
```

### Restoring State

Restore a conversation using the same ID and persistence directory:

```python
# Later, in a different session
del conversation

# Deserialize the conversation
print("Deserializing conversation...")
conversation = Conversation(
    agent=agent,
    callbacks=[conversation_callback],
    workspace=cwd,
    persistence_dir=persistence_dir,
    conversation_id=conversation_id,
)

conversation.send_message("Continue task")
conversation.run()
# Continues from saved state
```

## What Gets Persisted

The conversation state includes information that allows seamless restoration:

- **Message History**: Complete event log including user messages, agent responses, and system events
- **Agent Configuration**: LLM settings, tools, MCP servers, and agent parameters
- **Execution State**: Current agent status (idle, running, paused, etc.), iteration count, and stuck detection settings
- **Tool Outputs**: Results from bash commands, file operations, and other tool executions
- **Statistics**: LLM usage metrics like token counts and API calls
- **Workspace Context**: Working directory and file system state
- **Activated Skills**: Skills that have been enabled during the conversation
- **Secrets**: Managed credentials and API keys

For the complete implementation details, see the `ConversationState` class in the source code.

## Persistence Directory Structure

When you set a `persistence_dir`, your conversation will be persisted to a directory structure where each conversation has its own subdirectory. By default, the persistence directory is `workspace/conversations/` (unless you specify a custom path).

```
workspace/conversations/
  <conversation-id-1>/
    base_state.json
    events/
      event-00000-<event-id>.json
      event-00001-<event-id>.json
      ...
  <conversation-id-2>/
    ...
```

Each conversation directory contains:

- **base_state.json**: The core conversation state including agent configuration, execution status, statistics, and metadata
- **events/**: A subdirectory containing individual event files, each named with a sequential index and event ID (e.g., `event-00000-abc123.json`)

The collection of event files in the `events/` directory represents the same trajectory data you would find in the `trajectory.json` file from OpenHands V0, but split into individual files for better performance and granular access.

## Ready-to-run Example

This example is available on GitHub: [examples/01_standalone_sdk/10_persistence.py](https://github.com/OpenHands/software-agent-sdk/blob/main/examples/01_standalone_sdk/10_persistence.py)

You can run the example code as-is.

**Note:** The model name should follow the [LiteLLM convention](https://models.litellm.ai/): `provider/model_name` (e.g., `anthropic/claude-sonnet-4-5-20250929`, `openai/gpt-4o`). The `LLM_API_KEY` should be the API key for your chosen provider.

### Running the Example

#### Bring-your-own provider key

```bash
export LLM_API_KEY="your-api-key"
export LLM_MODEL="anthropic/claude-sonnet-4-5-20250929"  # or openai/gpt-4o, etc.
cd software-agent-sdk
uv run python examples/01_standalone_sdk/10_persistence.py
```

#### OpenHands Cloud

```bash
cd software-agent-sdk
uv run python examples/01_standalone_sdk/10_persistence.py
```

**Tip:** ChatGPT Plus/Pro subscribers: You can use `LLM.subscription_login()` to authenticate with your ChatGPT account and access Codex models without consuming API credits. See the [LLM Subscriptions guide](/sdk/guides/llm-subscriptions) for details.

## Next Steps

- [Pause and Resume](/sdk/guides/convo-pause-and-resume) - Control execution flow
- [Async Operations](/sdk/guides/convo-async) - Non-blocking operations
