<!-- Source: https://docs.openhands.dev/sdk/guides/critic -->

# Critic (Experimental)

**This feature is highly experimental and subject to change.** The API, configuration, and behavior may evolve significantly based on feedback and testing.

A ready-to-run example is available [here](#ready-to-run-example)!

## What is a Critic?

A critic is an evaluator that analyzes agent actions and conversation history to predict the quality or success probability of agent decisions. The critic runs alongside the agent and provides:

- **Quality scores**: Probability scores between 0.0 and 1.0 indicating predicted success
- **Real-time feedback**: Scores computed during agent execution, not just at completion

You can use critic scores to build automated workflows, such as triggering the agent to reflect on and fix its previous solution when the critic indicates poor task performance.

This critic is a more advanced extension of the approach described in our blog post [SOTA on SWE-Bench Verified with Inference-Time Scaling and Critic Model](https://openhands.dev/blog).

A technical report with detailed evaluation metrics is forthcoming.

## Quick Start

When using the OpenHands LLM Provider (`llm-proxy.*.all-hands.dev`), the critic is automatically configured - no additional setup required.

## Understanding Critic Results

Critic evaluations produce scores and feedback:

- **score**: Float between 0.0 and 1.0 representing predicted success probability
- **message**: Optional feedback with detailed probabilities
- **success**: Boolean property (True if score >= 0.5)

Results are automatically displayed in the conversation visualizer.

## Accessing Results Programmatically

```python
from openhands.sdk import Event, ActionEvent, MessageEvent

def callback(event: Event):
    if isinstance(event, (ActionEvent, MessageEvent)):
        if event.critic_result is not None:
            print(f"Critic score: {event.critic_result.score:.3f}")
            print(f"Success: {event.critic_result.success}")

conversation = Conversation(agent=agent, callbacks=[callback])
```

## Troubleshooting

### Critic Evaluations Not Appearing

- Verify the critic is properly configured and passed to the Agent
- Ensure you're using the OpenHands LLM Provider (`llm-proxy.*.all-hands.dev`)

### API Authentication Errors

- Verify `LLM_API_KEY` is set correctly
- Check that the API key has not expired

## Ready-to-run Example

The critic model is hosted by the OpenHands LLM Provider and is currently free to use.

This example is available on GitHub: [examples/01_standalone_sdk/34_critic_example.py](https://github.com/OpenHands/software-agent-sdk/blob/main/examples/01_standalone_sdk/34_critic_example.py)

### Running the Example

You can run the example code as-is:

```bash
uv run python examples/01_standalone_sdk/34_critic_example.py
```

With environment variables:

```bash
export LLM_API_KEY="your-api-key"
export LLM_MODEL="anthropic/claude-sonnet-4-5-20250929"  # or openai/gpt-4o, etc.
cd software-agent-sdk
uv run python examples/01_standalone_sdk/34_critic_example.py
```

**Tip:** ChatGPT Plus/Pro subscribers: You can use `LLM.subscription_login()` to authenticate with your ChatGPT account and access Codex models without consuming API credits. See the [LLM Subscriptions guide](/sdk/guides/llm-subscriptions) for details.

## Next Steps

- [Observability](/sdk/guides/observability) - Monitor and log agent behavior
- [Metrics](/sdk/guides/llm-metrics) - Collect performance metrics
- [Stuck Detector](/sdk/guides/agent-stuck-detector) - Detect unproductive agent patterns
