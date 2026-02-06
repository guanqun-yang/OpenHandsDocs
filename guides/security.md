<!-- Source: https://docs.openhands.dev/sdk/guides/security -->

# Security & Action Confirmation

Agent actions can be controlled through two complementary mechanisms: confirmation policy that determine when user approval is required, and security analyzer that evaluates action risk levels. Together, they provide flexible control over agent behavior while maintaining safety.

## Confirmation Policy

A ready-to-run example is available here!

Confirmation policy controls whether actions require user approval before execution. They provide a simple way to ensure safe agent operation by requiring explicit permission for actions.

### Setting Confirmation Policy

Set the confirmation policy on your conversation:

```python
from openhands.sdk.security.confirmation_policy import AlwaysConfirm

conversation = Conversation(agent=agent, workspace=".")
conversation.set_confirmation_policy(AlwaysConfirm())
```

Available policies:

- `AlwaysConfirm()` - Require approval for all actions
- `NeverConfirm()` - Execute all actions without approval
- `ConfirmRisky()` - Only require approval for risky actions (requires security analyzer)

### Custom Confirmation Handler

Implement your approval logic by checking conversation status:

```python
while conversation.state.agent_status != AgentExecutionStatus.FINISHED:
    if conversation.state.agent_status == AgentExecutionStatus.WAITING_FOR_CONFIRMATION:
        pending = ConversationState.get_unmatched_actions(conversation.state.events)
        if not confirm_in_console(pending):
            conversation.reject_pending_actions("User rejected")
            continue
    conversation.run()
```

### Rejecting Actions

Provide feedback when rejecting to help the agent try a different approach:

```python
if not user_approved:
    conversation.reject_pending_actions(
        "User rejected because actions seem too risky. "
        "Please try a safer approach."
    )
```

### Ready-to-run Example - Confirmation

Full confirmation example: `examples/01_standalone_sdk/04_confirmation_mode_example.py`

Require user approval before executing agent actions:

```python
"""OpenHands Agent SDK — Confirmation Mode Example"""

import os
import signal
from collections.abc import Callable
from pydantic import SecretStr
from openhands.sdk import LLM, BaseConversation, Conversation
from openhands.sdk.conversation.state import (
    ConversationExecutionStatus,
    ConversationState,
)
from openhands.sdk.security.confirmation_policy import AlwaysConfirm, NeverConfirm
from openhands.sdk.security.llm_analyzer import LLMSecurityAnalyzer
from openhands.tools.preset.default import get_default_agent

signal.signal(signal.SIGINT, lambda *_: (_ for _ in ()).throw(KeyboardInterrupt()))

def _print_action_preview(pending_actions) -> None:
    print(f"\n🔍 Agent created {len(pending_actions)} action(s) awaiting confirmation:")
    for i, action in enumerate(pending_actions, start=1):
        snippet = str(action.action)[:100].replace("\n", " ")
        print(f" {i}. {action.tool_name}: {snippet}...")

def confirm_in_console(pending_actions) -> bool:
    """Return True to approve, False to reject."""
    _print_action_preview(pending_actions)
    while True:
        try:
            ans = (
                input("\nDo you want to execute these actions? (yes/no): ")
                .strip()
                .lower()
            )
        except (EOFError, KeyboardInterrupt):
            print("\n❌ No input received; rejecting by default.")
            return False
        if ans in ("yes", "y"):
            print("✅ Approved — executing actions…")
            return True
        if ans in ("no", "n"):
            print("❌ Rejected — skipping actions…")
            return False
        print("Please enter 'yes' or 'no'.")

def run_until_finished(conversation: BaseConversation, confirmer: Callable) -> None:
    """Drive the conversation until FINISHED."""
    while conversation.state.execution_status != ConversationExecutionStatus.FINISHED:
        if (
            conversation.state.execution_status == ConversationExecutionStatus.WAITING_FOR_CONFIRMATION
        ):
            pending = ConversationState.get_unmatched_actions(conversation.state.events)
            if not pending:
                raise RuntimeError(
                    "⚠️ Agent is waiting for confirmation but no pending actions "
                    "were found. This should not happen."
                )
            if not confirmer(pending):
                conversation.reject_pending_actions("User rejected the actions")
                continue
        print("▶️ Running conversation.run()…")
        conversation.run()

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

agent = get_default_agent(llm=llm)
conversation = Conversation(agent=agent, workspace=os.getcwd())

# Confirmation mode ON
conversation.set_confirmation_policy(AlwaysConfirm())
print("\n1) Command that will likely create actions…")
conversation.send_message("Please list the files in the current directory using ls -la")
run_until_finished(conversation, confirm_in_console)

# Disable confirmation mode
print("\n4) Disable confirmation mode and run a command…")
conversation.set_confirmation_policy(NeverConfirm())
conversation.send_message("Please echo 'Hello from confirmation mode example!'")
conversation.run()

print("\n=== Example Complete ===")
```

#### Running the Example

You can run the example code as-is. The model name should follow the LiteLLM convention: `provider/model_name` (e.g., `anthropic/claude-sonnet-4-5-20250929`, `openai/gpt-4o`).

The `LLM_API_KEY` should be the API key for your chosen provider.

##### Bring-your-own provider key

```bash
export LLM_API_KEY="your-api-key"
export LLM_MODEL="anthropic/claude-sonnet-4-5-20250929" # or openai/gpt-4o, etc.
cd software-agent-sdk
uv run python examples/01_standalone_sdk/04_confirmation_mode_example.py
```

##### OpenHands Cloud

ChatGPT Plus/Pro subscribers: You can use `LLM.subscription_login()` to authenticate with your ChatGPT account and access Codex models without consuming API credits. See the LLM Subscriptions guide for details.

## Security Analyzer

Security analyzer evaluates the risk of agent actions before execution, helping protect against potentially dangerous operations. They analyze each action and assign a security risk level:

- **LOW** - Safe operations with minimal security impact
- **MEDIUM** - Moderate security impact, review recommended
- **HIGH** - Significant security impact, requires confirmation
- **UNKNOWN** - Risk level could not be determined

Security analyzer work in conjunction with confirmation policy (like `ConfirmRisky()`) to determine whether user approval is needed before executing an action. This provides an additional layer of safety for autonomous agent operations.

### LLM Security Analyzer

A ready-to-run example is available here!

The `LLMSecurityAnalyzer` is the default implementation provided in the agent-sdk. It leverages the LLM's understanding of action context to provide lightweight security analysis. The LLM can annotate actions with security risk levels during generation, which the analyzer then uses to make security decisions.

#### Security Analyzer Configuration

Create an LLM-based security analyzer to review actions before execution:

```python
from openhands.sdk import LLM
from openhands.sdk.security.llm_analyzer import LLMSecurityAnalyzer

llm = LLM(
    usage_id="security-analyzer",
    model=model,
    base_url=base_url,
    api_key=SecretStr(api_key),
)

security_analyzer = LLMSecurityAnalyzer(llm=security_llm)
agent = Agent(llm=llm, tools=tools, security_analyzer=security_analyzer)
```

The security analyzer:

- Reviews each action before execution
- Flags potentially dangerous operations
- Can be configured with custom security policy
- Uses a separate LLM to avoid conflicts with the main agent

### Custom Security Analyzer Implementation

You can extend the security analyzer functionality by creating your own implementation that inherits from the `SecurityAnalyzerBase` class. This allows you to implement custom security logic tailored to your specific requirements.

#### Creating a Custom Analyzer

To create a custom security analyzer, inherit from `SecurityAnalyzerBase` and implement the `security_risk()` method:

```python
from openhands.sdk.security.analyzer import SecurityAnalyzerBase
from openhands.sdk.security.risk import SecurityRisk
from openhands.sdk.event.llm_convertible import ActionEvent

class CustomSecurityAnalyzer(SecurityAnalyzerBase):
    """Custom security analyzer with domain-specific rules."""

    def security_risk(self, action: ActionEvent) -> SecurityRisk:
        """Evaluate security risk based on custom rules."""
        action_str = str(action.action.model_dump()).lower() if action.action else ""

        # High-risk patterns
        if any(pattern in action_str for pattern in ['rm -rf', 'sudo', 'chmod 777']):
            return SecurityRisk.HIGH

        # Medium-risk patterns
        if any(pattern in action_str for pattern in ['curl', 'wget', 'git clone']):
            return SecurityRisk.MEDIUM

        # Default to low risk
        return SecurityRisk.LOW

# Use your custom analyzer
security_analyzer = CustomSecurityAnalyzer()
agent = Agent(llm=llm, tools=tools, security_analyzer=security_analyzer)
```

For more details on the base class implementation, see the source code.

### Configurable Security Policy

A ready-to-run example is available here!

Agents use security policies to guide their risk assessment of actions. The SDK provides a default security policy template, but you can customize it to match your specific security requirements and guidelines.

#### Using Custom Security Policies

You can provide a custom security policy template when creating an agent:

```python
from openhands.sdk import Agent, LLM

llm = LLM(
    usage_id="agent",
    model="anthropic/claude-sonnet-4-5-20250929",
    api_key=SecretStr(api_key),
)

# Provide a custom security policy template file
agent = Agent(
    llm=llm,
    tools=tools,
    security_policy_filename="my_security_policy.j2",
)
```

Custom security policies allow you to:

- Define organization-specific risk assessment guidelines
- Set custom thresholds for security risk levels
- Add domain-specific security rules
- Tailor risk evaluation to your use case

The security policy is provided as a Jinja2 template that gets rendered into the agent's system prompt, guiding how it evaluates the security risk of its actions.

## Next Steps

- Custom Tools - Build secure custom tools
- Custom Secrets - Secure credential management
