<!-- Source: https://docs.openhands.dev/sdk/guides/convo-send-message-while-running -->

# Send Message While Running

Send additional messages to a running agent mid-execution to provide corrections, updates, or additional context.

## How It Works

The key steps are:

1. Start `conversation.run()` in a background thread
2. Send additional messages using `conversation.send_message()` while the agent is processing
3. Use `thread.join()` to wait for completion

The agent receives and incorporates the new message mid-execution, allowing for real-time corrections and dynamic guidance:

```python
import threading
import time
from datetime import datetime

# Start agent processing in background
thread = threading.Thread(target=conversation.run)
thread.start()

# Wait then send second message while agent is processing
time.sleep(2)  # Give agent time to start working
second_time = timestamp()

conversation.send_message(
    f"Please also add this second sentence to document.txt: "
    f"'Message 3 sent at {second_time}, written at [CURRENT_TIME].'"
)

# Wait for completion
thread.join()
```

## Ready-to-run Example

This example is available on GitHub: [examples/01_standalone_sdk/18_send_message_while_processing.py](https://github.com/OpenHands/software-agent-sdk/blob/main/examples/01_standalone_sdk/18_send_message_while_processing.py)

### Demonstration Flow

1. Send initial message asking agent to:
   - Write "Message 1 sent at [time], written at [CURRENT_TIME]"
   - Wait 3 seconds
   - Write "Message 2 sent at [time], written at [CURRENT_TIME]"

2. Start agent processing in a background thread

3. While agent is busy (during the 3-second delay), send a second message asking to add:
   - "Message 3 sent at [time], written at [CURRENT_TIME]"

4. Verify that all three lines are processed and included in the final document

### Expected Evidence

The final document will contain three lines with dual timestamps:

- "Message 1 sent at HH:MM:SS, written at HH:MM:SS" (from initial message, written immediately)
- "Message 2 sent at HH:MM:SS, written at HH:MM:SS" (from initial message, written after 3-second delay)
- "Message 3 sent at HH:MM:SS, written at HH:MM:SS" (from second message sent during delay)

The timestamps will show that Message 3 was sent while the agent was running, but was still successfully processed and written to the document. This proves that:

- The second user message was sent while the agent was processing the first task
- The agent successfully received and processed the second message
- The agent's event system allows for real-time message integration during processing

## Running the Example

You can run the example code as-is.

**Note:** The model name should follow the [LiteLLM convention](https://models.litellm.ai/): `provider/model_name` (e.g., `anthropic/claude-sonnet-4-5-20250929`, `openai/gpt-4o`). The `LLM_API_KEY` should be the API key for your chosen provider.

### Bring-your-own provider key

```bash
export LLM_API_KEY="your-api-key"
export LLM_MODEL="anthropic/claude-sonnet-4-5-20250929"  # or openai/gpt-4o, etc.
cd software-agent-sdk
uv run python examples/01_standalone_sdk/18_send_message_while_processing.py
```

### OpenHands Cloud

```bash
cd software-agent-sdk
uv run python examples/01_standalone_sdk/18_send_message_while_processing.py
```

**Tip:** ChatGPT Plus/Pro subscribers: You can use `LLM.subscription_login()` to authenticate with your ChatGPT account and access Codex models without consuming API credits. See the [LLM Subscriptions guide](/sdk/guides/llm-subscriptions) for details.

## Next Steps

- [Pause and Resume](/sdk/guides/convo-pause-and-resume) - Control execution flow
- [Async Operations](/sdk/guides/convo-async) - Non-blocking operations
