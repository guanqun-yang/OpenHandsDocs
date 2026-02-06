<!-- Source: https://docs.openhands.dev/sdk/guides/agent-tom-agent -->

# Theory of Mind (TOM) Agent

Tom (Theory of Mind) Agent provides advanced user understanding capabilities that help your agent interpret vague instructions and adapt to user preferences over time. Built on research in user mental modeling, Tom agents can:

- Understand unclear or ambiguous user requests
- Provide personalized guidance based on user modeling
- Build long-term user preference profiles
- Adapt responses based on conversation history

This is particularly useful when:
- User instructions are vague or incomplete
- You need to infer user intent from minimal context
- Building personalized experiences across multiple conversations
- Understanding user preferences and working patterns

## Research Foundation

Tom agent is based on the TOM-SWE research paper on user mental modeling for software engineering agents:

**Citation:**
```bibtex
@misc{zhou2025tomsweusermentalmodeling,
  title={TOM-SWE: User Mental Modeling For Software Engineering Agents},
  author={Xuhui Zhou and Valerie Chen and Zora Zhiruo Wang and Graham Neubig and Maarten Sap and Xingyao Wang},
  year={2025},
  eprint={2510.21903},
  archivePrefix={arXiv},
  primaryClass={cs.SE},
  url={https://arxiv.org/abs/2510.21903},
}
```

**Paper:** [TOM-SWE on arXiv](https://arxiv.org/abs/2510.21903)

## Quick Start

This example is available on GitHub: [examples/01_standalone_sdk/30_tom_agent.py](https://github.com/OpenHands/software-agent-sdk/blob/main/examples/01_standalone_sdk/30_tom_agent.py)

```python
from openhands.sdk import LLM, Agent, Conversation
from openhands.sdk.tool import Tool
from openhands.tools.preset.default import get_default_tools
from openhands.tools.tom_consult import (
    SleeptimeComputeAction,
    SleeptimeComputeTool,
    TomConsultTool,
)

# Configure LLM
llm = LLM(
    model=os.getenv("LLM_MODEL", "anthropic/claude-sonnet-4-5-20250929"),
    api_key=os.getenv("LLM_API_KEY"),
    base_url=os.getenv("LLM_BASE_URL", None),
    usage_id="agent",
    drop_params=True,
)

# Build tools list with Tom tools
tools = get_default_tools(enable_browser=False)

# Configure Tom tools with parameters
tom_params = {
    "enable_rag": True,  # Enable RAG in Tom agent
}

# Add both Tom tools to the agent
tools.append(Tool(name=TomConsultTool.name, params=tom_params))
tools.append(Tool(name=SleeptimeComputeTool.name, params=tom_params))

# Create agent with Tom capabilities
agent = Agent(llm=llm, tools=tools)

# Start conversation
conversation = Conversation(
    agent=agent,
    workspace=cwd,
    persistence_dir=CONVERSATIONS_DIR
)

conversation.send_message(
    "I need to debug some code but I'm not sure where to start. "
    "Can you help me figure out the best approach?"
)
conversation.run()
```

## Tom Tools

### TomConsultTool

The consultation tool provides personalized guidance when the agent encounters vague or unclear user requests:

```python
# The agent can automatically call this tool when needed
# Example: User says "I need to debug something"
# Tom analyzes the vague request and provides specific guidance
```

**Key features:**
- Analyzes conversation history for context
- Provides personalized suggestions based on user modeling
- Helps disambiguate vague instructions
- Adapts to user communication patterns

### SleeptimeComputeTool

The indexing tool processes conversation history to build user preference profiles:

```python
# Index conversations for future personalization
sleeptime_compute_tool = conversation.agent.tools_map.get("sleeptime_compute")
if sleeptime_compute_tool:
    result = sleeptime_compute_tool.executor(
        SleeptimeComputeAction(), conversation
    )
```

**Key features:**
- Processes conversation history into user models
- Stores preferences in `~/.openhands/` directory
- Builds understanding of user patterns over time
- Enables long-term personalization across sessions

## Configuration

### RAG Support

Enable retrieval-augmented generation for enhanced context awareness:

```python
tom_params = {
    "enable_rag": True,  # Enable RAG for better context retrieval
}
```

### Custom LLM for Tom

You can optionally use a different LLM for Tom's internal reasoning:

```python
# Use the same LLM as main agent
tom_params["llm_model"] = llm.model
tom_params["api_key"] = llm.api_key.get_secret_value()

# Or configure a separate LLM for Tom
tom_llm = LLM(model="gpt-4", api_key=SecretStr("different-key"))
tom_params["llm_model"] = tom_llm.model
tom_params["api_key"] = tom_llm.api_key.get_secret_value()
```

## Data Storage

Tom stores user modeling data persistently in `~/.openhands/`:

```
~/.openhands/
  user_models/
    {user_id}/
      user_model.json
      processed_sessions_timestamps.json
  conversations/
    {session_id}/
      events
```

- `user_models/` stores user preference profiles, with each user having their own subdirectory containing `user_model.json` (the current user model)
- `conversations/` contains indexed conversation data

This persistent storage enables Tom to:
- Remember user preferences across sessions
- Track which conversations have been indexed
- Build long-term understanding of user patterns

## Use Cases

### 1. Handling Vague Requests

When a user provides minimal information:

```python
conversation.send_message("Help me with that bug")
# Tom analyzes history to determine which bug and suggest approach
```

### 2. Personalized Recommendations

Tom adapts suggestions based on past interactions:

```python
# After multiple conversations, Tom learns:
# - User prefers minimal explanations
# - User typically works with Python
# - User values efficiency over verbosity
```

### 3. Intent Inference

Understanding what the user really wants:

```python
conversation.send_message("Make it better")
# Tom infers from context what "it" is and how to improve it
```

## Best Practices

1. **Enable RAG**: For better context awareness, always enable RAG:
   ```python
   tom_params = {"enable_rag": True}
   ```

2. **Index Regularly**: Run sleeptime compute after important conversations to build better user models

3. **Provide Context**: Even with Tom, providing more context leads to better results

4. **Monitor Data**: Check `~/.openhands/` periodically to understand what's being learned

5. **Privacy Considerations**: Be aware that conversation data is stored locally for user modeling

## Running the Example

You can run the example code as-is.

**Note:** The model name should follow the [LiteLLM convention](https://models.litellm.ai/): `provider/model_name` (e.g., `anthropic/claude-sonnet-4-5-20250929`, `openai/gpt-4o`). The `LLM_API_KEY` should be the API key for your chosen provider.

### Bring-your-own provider key

```bash
export LLM_API_KEY="your-api-key"
export LLM_MODEL="anthropic/claude-sonnet-4-5-20250929"  # or openai/gpt-4o, etc.
cd software-agent-sdk
uv run python examples/01_standalone_sdk/30_tom_agent.py
```

### OpenHands Cloud

```bash
cd software-agent-sdk
uv run python examples/01_standalone_sdk/30_tom_agent.py
```

**Tip:** ChatGPT Plus/Pro subscribers: You can use `LLM.subscription_login()` to authenticate with your ChatGPT account and access Codex models without consuming API credits. See the [LLM Subscriptions guide](/sdk/guides/llm-subscriptions) for details.

## Next Steps

- [Agent Delegation](/sdk/guides/agent-delegation) - Combine Tom with sub-agents for complex workflows
- [Context Condenser](/sdk/guides/context-condenser) - Manage long conversation histories effectively
- [Custom Tools](/sdk/guides/custom-tools) - Create tools that work with Tom's insights
