<!-- Source: https://docs.openhands.dev/sdk/guides/skill -->

# Agent Skills & Context

This guide shows how to implement skills in the SDK. For conceptual overview, see Skills Overview.

OpenHands supports an extended version of the AgentSkills standard with optional keyword triggers.

## Context Loading Methods

| Method | When Content Loads | Use Case |
|--------|-------------------|----------|
| Always-loaded | At conversation start | Repository rules, coding standards |
| Trigger-loaded | When keywords match | Specialized tasks, domain knowledge |
| Progressive disclosure | Agent reads on demand | Large reference docs (AgentSkills) |

## Always-Loaded Context

Content that's always in the system prompt.

### Option 1: AGENTS.md (Auto-loaded)

Place `AGENTS.md` at your repo root - it's loaded automatically. See Permanent Context.

```python
from openhands.sdk.context.skills import load_project_skills

# Automatically finds AGENTS.md, CLAUDE.md, GEMINI.md at workspace root
skills = load_project_skills(workspace_dir="/path/to/repo")
agent_context = AgentContext(skills=skills)
```

### Option 2: Inline Skill (Code-defined)

```python
from openhands.sdk import AgentContext
from openhands.sdk.context import Skill

agent_context = AgentContext(
    skills=[
        Skill(
            name="code-style",
            content="Always use type hints in Python.",
            trigger=None,  # No trigger = always loaded
        ),
    ]
)
```

## Trigger-Loaded Context

Content injected when keywords appear in user messages. See Keyword-Triggered Skills.

```python
from openhands.sdk.context import Skill, KeywordTrigger

Skill(
    name="encryption-helper",
    content="Use the encrypt.sh script to encrypt messages.",
    trigger=KeywordTrigger(keywords=["encrypt", "decrypt"]),
)
```

When user says "encrypt this", the content is injected into the message:

```
<EXTRA_INFO>
The following information has been included based on a keyword match for "encrypt".
Skill location: /path/to/encryption-helper

Use the encrypt.sh script to encrypt messages.
</EXTRA_INFO>
```

## Progressive Disclosure (AgentSkills Standard)

For the agent to trigger skills, use the AgentSkills standard `SKILL.md` format. The agent sees a summary and reads full content on demand.

```python
from openhands.sdk.context.skills import load_skills_from_dir

# Load SKILL.md files from a directory
_, _, agent_skills = load_skills_from_dir("/path/to/skills")
agent_context = AgentContext(skills=list(agent_skills.values()))
```

Skills are listed in the system prompt:

```
<available_skills>
<skill>
  <name>code-style</name>
  <description>Project coding standards.</description>
  <location>/path/to/code-style/SKILL.md</location>
</skill>
</available_skills>
```

Add triggers to a `SKILL.md` for both progressive disclosure AND automatic injection when keywords match.

## Full Example

Full example: `examples/01_standalone_sdk/03_activate_skill.py`

```python
import os
from pydantic import SecretStr
from openhands.sdk import (
    LLM,
    Agent,
    AgentContext,
    Conversation,
    Event,
    LLMConvertibleEvent,
    get_logger,
)
from openhands.sdk.context import (
    KeywordTrigger,
    Skill,
)
from openhands.sdk.tool import Tool
from openhands.tools.file_editor import FileEditorTool
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
    ),
    Tool(name=FileEditorTool.name),
]

agent_context = AgentContext(
    skills=[
        Skill(
            name="repo.md",
            content="When you see this message, you should reply like "
            "you are a grumpy cat forced to use the internet.",
            source=None,
            trigger=None,
        ),
        Skill(
            name="flarglebargle",
            content=(
                'IMPORTANT! The user has said the magic word "flarglebargle". '
                "You must only respond with a message telling them how smart they are"
            ),
            source=None,
            trigger=KeywordTrigger(keywords=["flarglebargle"]),
        ),
    ],
    system_message_suffix="Always finish your response with the word 'yay!'",
    user_message_suffix="The first character of your response should be 'I'",
    load_public_skills=True,
)

# Agent
agent = Agent(llm=llm, tools=tools, agent_context=agent_context)

llm_messages = []

def conversation_callback(event: Event):
    if isinstance(event, LLMConvertibleEvent):
        llm_messages.append(event.to_llm_message())

conversation = Conversation(
    agent=agent,
    callbacks=[conversation_callback],
    workspace=cwd
)

print("=" * 100)
print("Checking if the repo skill is activated.")
conversation.send_message("Hey are you a grumpy cat?")
conversation.run()

print("=" * 100)
print("Now sending flarglebargle to trigger the knowledge skill!")
conversation.send_message("flarglebargle!")
conversation.run()

print("=" * 100)
print("Now triggering public skill 'github'")
conversation.send_message(
    "About GitHub - tell me what additional info I've just provided?"
)
conversation.run()

print("=" * 100)
print("Conversation finished. Got the following LLM messages:")
for i, message in enumerate(llm_messages):
    print(f"Message {i}: {str(message)[:200]}")

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
uv run python examples/01_standalone_sdk/03_activate_skill.py
```

#### OpenHands Cloud

ChatGPT Plus/Pro subscribers: You can use `LLM.subscription_login()` to authenticate with your ChatGPT account and access Codex models without consuming API credits. See the LLM Subscriptions guide for details.

## Creating Skills

Skills are defined with a name, content (the instructions), and an optional trigger:

```python
agent_context = AgentContext(
    skills=[
        Skill(
            name="AGENTS.md",
            content="When you see this message, you should reply like "
            "you are a grumpy cat forced to use the internet.",
            trigger=None,  # Always active
        ),
        Skill(
            name="flarglebargle",
            content='IMPORTANT! The user has said the magic word "flarglebargle". '
            "You must only respond with a message telling them how smart they are",
            trigger=KeywordTrigger(keywords=["flarglebargle"]),
        ),
    ]
)
```

## Keyword Triggers

Use `KeywordTrigger` to activate skills only when specific words appear:

```python
Skill(
    name="magic-word",
    content="Special instructions when magic word is detected",
    trigger=KeywordTrigger(keywords=["flarglebargle", "sesame"]),
)
```

## File-Based Skills (SKILL.md)

For reusable skills, use the AgentSkills standard directory format.

Full example: `examples/05_skills_and_plugins/01_loading_agentskills/main.py`

### Directory Structure

Each skill is a directory containing:

```
my-skill/
├── SKILL.md
├── scripts/
│   └── helper.sh
├── references/
│   └── examples.md
└── assets/
    └── config.json
```

| Component | Required | Description |
|-----------|----------|-------------|
| SKILL.md | Yes | Skill definition with frontmatter |
| scripts/ | No | Executable scripts |
| references/ | No | Reference documentation |
| assets/ | No | Static assets |

### SKILL.md Format

The `SKILL.md` file defines the skill with YAML frontmatter:

```yaml
---
name: my-skill                    # Required (standard)
description: >                    # Required (standard)
  A brief description of what this skill does and when to use it.
license: MIT                      # Optional (standard)
compatibility: Requires bash      # Optional (standard)
metadata:                         # Optional (standard)
  author: your-name
  version: "1.0"
triggers:                         # Optional (OpenHands extension)
  - keyword1
  - keyword2
---

# Skill Content

Instructions and documentation for the agent...
```

### Frontmatter Fields

| Field | Required | Description |
|-------|----------|-------------|
| name | Yes | Skill identifier (lowercase + hyphens) |
| description | Yes | What the skill does (shown to agent) |
| triggers | No | Keywords that auto-activate this skill (OpenHands extension) |
| license | No | License name |
| compatibility | No | Environment requirements |
| metadata | No | Custom key-value pairs |

Add triggers to make your `SKILL.md` keyword-activated by matching a user prompt. Without triggers, the skill can only be triggered by the agent, not the user.

## Loading Skills

Use `load_skills_from_dir()` to load all skills from a directory:

```python
from openhands.sdk.context.skills import load_skills_from_dir

repo_skills, knowledge_skills, agent_skills = load_skills_from_dir(skills_dir)
```

- **repo_skills**: Skills from `repo.md` files (always active)
- **knowledge_skills**: Skills from `knowledge/` subdirectories
- **agent_skills**: Skills from `SKILL.md` files (AgentSkills standard)

## Key Functions

### discover_skill_resources()

Discovers resource files in a skill directory:

```python
from openhands.sdk.context.skills import discover_skill_resources

resources = discover_skill_resources(skill_dir)
print(resources.scripts)      # List of script files
print(resources.references)   # List of reference files
print(resources.assets)       # List of asset files
print(resources.skill_root)   # Path to skill directory
```

## Loading Public Skills

OpenHands maintains a public skills repository with community-contributed skills. You can automatically load these skills without waiting for SDK updates.

### Automatic Loading via AgentContext

Enable public skills loading in your `AgentContext`:

```python
agent_context = AgentContext(
    load_public_skills=True,  # Auto-load from public registry
    skills=[
        # Your custom skills here
    ]
)
```

When enabled, the SDK will:

- Clone or update the public skills repository to `~/.openhands/cache/skills/` on first run
- Load all available skills from the repository
- Merge them with your explicitly defined skills

Skill Precedence: If a skill name conflicts, your explicitly defined skills take precedence over public skills.

### Programmatic Loading

You can also load public skills manually and have more control:

```python
from openhands.sdk.context.skills import load_public_skills

# Load all public skills
public_skills = load_public_skills()

# Use with AgentContext
agent_context = AgentContext(skills=public_skills)

# Or combine with custom skills
my_skills = [
    Skill(name="custom", content="Custom instructions", trigger=None)
]
agent_context = AgentContext(skills=my_skills + public_skills)
```

### Custom Skills Repository

You can load skills from your own repository:

```python
from openhands.sdk.context.skills import load_public_skills

# Load from a custom repository
custom_skills = load_public_skills(
    repo_url="https://github.com/my-org/my-skills",
    branch="main"
)
```

## Customizing Agent Context

### Message Suffixes

Append custom instructions to the system prompt or user messages via `AgentContext`:

```python
agent_context = AgentContext(
    system_message_suffix="""
<REPOSITORY_INFO>
Repository: my-project
Branch: feature/new-api
</REPOSITORY_INFO>
""".strip(),
    user_message_suffix="Remember to explain your reasoning."
)
```

- **system_message_suffix**: Appended to system prompt (always active, combined with repo skills)
- **user_message_suffix**: Appended to each user message

### Replacing the Entire System Prompt

For complete control, provide a custom Jinja2 template via the Agent class:

```python
from openhands.sdk import Agent

agent = Agent(
    llm=llm,
    tools=tools,
    system_prompt_filename="/path/to/custom_system_prompt.j2",  # Absolute path
    system_prompt_kwargs={"cli_mode": True, "repo_name": "my-project"}
)
```

Custom template example (`custom_system_prompt.j2`):

```jinja
You are a helpful coding assistant for {{ repo_name }}.

{% if cli_mode %}
You are running in CLI mode. Keep responses concise.
{% endif %}

Follow these guidelines:
- Write clean, well-documented code
- Consider edge cases and error handling
- Suggest tests when appropriate
```

Key points:

- Use relative filenames (e.g., `"system_prompt.j2"`) to load from the agent's prompts directory
- Use absolute paths (e.g., `"/path/to/prompt.j2"`) to load from any location
- Pass variables to the template via `system_prompt_kwargs`
- The `system_message_suffix` from `AgentContext` is automatically appended after your custom prompt

## Next Steps

- Custom Tools - Create specialized tools
- MCP Integration - Connect external tool servers
- Confirmation Mode - Add execution approval
