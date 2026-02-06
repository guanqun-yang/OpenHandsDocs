<!-- Source: https://docs.openhands.dev/sdk/guides/plugins -->

# Plugins

Plugins provide a way to package and distribute multiple agent components together. A single plugin can include:

- **Skills**: Specialized knowledge and workflows
- **Hooks**: Event handlers for tool lifecycle
- **MCP Config**: External tool server configurations
- **Agents**: Specialized agent definitions
- **Commands**: Slash commands

The plugin format is compatible with the Claude Code plugin structure.

## Plugin Structure

See the `example_plugins` directory for a complete working plugin structure.

A plugin follows this directory structure:

```
plugin-name.plugin/
├── .plugin/
│   └── plugin.json
├── skills/
│   └── skill-name/
├── hooks/
│   └── hooks.json
├── agents/
│   └── agent-name.md
├── commands/
│   └── command-name.md
├── .mcp.json
└── README.md
```

Note that the plugin metadata, i.e., `plugin-name/.plugin/plugin.json`, is required.

## Plugin Manifest

The manifest file `plugin-name/.plugin/plugin.json` defines plugin metadata:

```json
{
  "name": "code-quality",
  "version": "1.0.0",
  "description": "Code quality tools and workflows",
  "author": "openhands",
  "license": "MIT",
  "repository": "https://github.com/example/code-quality-plugin"
}
```

## Skills

Skills are defined in markdown files with YAML frontmatter:

```yaml
---
name: python-linting
description: Instructions for linting Python code
trigger:
  type: keyword
  keywords:
    - lint
    - linting
    - code quality
---

# Python Linting Skill

Run ruff to check for issues:

\`\`\`bash
ruff check .
\`\`\`
```

## Hooks

Hooks are defined in `hooks/hooks.json`:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "file_editor",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'File edited: $OPENHANDS_TOOL_NAME'",
            "timeout": 5
          }
        ]
      }
    ]
  }
}
```

## MCP Configuration

MCP servers are configured in `.mcp.json`:

```json
{
  "mcpServers": {
    "fetch": {
      "command": "uvx",
      "args": ["mcp-server-fetch"]
    }
  }
}
```

## Using Plugin Components

The ready-to-run example is available here!

Brief explanation on how to use a plugin with an agent.

### 1. Loading a Plugin

First, load the desired plugins.

```python
from openhands.sdk.plugin import Plugin

# Load a single plugin
plugin = Plugin.load("/path/to/plugin")

# Load all plugins from a directory
plugins = Plugin.load_all("/path/to/plugins")
```

### 2. Accessing Components

You can access the different plugin components to see which ones are available.

```python
# Skills
for skill in plugin.skills:
    print(f"Skill: {skill.name}")

# Hooks configuration
if plugin.hooks:
    print(f"Hooks configured: {plugin.hooks}")

# MCP servers
if plugin.mcp_config:
    servers = plugin.mcp_config.get("mcpServers", {})
    print(f"MCP servers: {list(servers.keys())}")
```

### 3. Using with an Agent

You can now feed your agent with your preferred plugin.

```python
# Create agent context with plugin skills
agent_context = AgentContext(
    skills=plugin.skills,
)

# Create agent with plugin MCP config
agent = Agent(
    llm=llm,
    tools=tools,
    mcp_config=plugin.mcp_config or {},
    agent_context=agent_context,
)

# Create conversation with plugin hooks
conversation = Conversation(
    agent=agent,
    hook_config=plugin.hooks,
)
```

## Ready-to-run Example

This example is available on GitHub: `examples/05_skills_and_plugins/02_loading_plugins/main.py`

```python
"""
Example: Loading Plugins

This example demonstrates how to load plugins that bundle multiple components:
- Skills (specialized knowledge and workflows)
- Hooks (event handlers for tool lifecycle)
- MCP configuration (external tool servers)
- Agents (specialized agent definitions)
- Commands (slash commands)

Plugins follow the Claude Code plugin structure for compatibility.
See the example_plugins/ directory for a complete plugin structure.
"""

import os
import sys
import tempfile
from pathlib import Path
from pydantic import SecretStr
from openhands.sdk import LLM, Agent, AgentContext, Conversation
from openhands.sdk.plugin import Plugin
from openhands.sdk.tool import Tool
from openhands.tools.file_editor import FileEditorTool
from openhands.tools.terminal import TerminalTool

# Get the directory containing this script
script_dir = Path(__file__).parent
example_plugins_dir = script_dir / "example_plugins"

# Load a single plugin
print("=" * 80)
print("Part 1: Loading a Single Plugin")
print("=" * 80)

plugin_path = example_plugins_dir / "code-quality"
print(f"Loading plugin from: {plugin_path}")
plugin = Plugin.load(plugin_path)

print("\nPlugin loaded successfully!")
print(f" Name: {plugin.name}")
print(f" Version: {plugin.version}")
print(f" Description: {plugin.description}")

# Exploring Plugin Components
print("\n" + "=" * 80)
print("Part 2: Exploring Plugin Components")
print("=" * 80)

# Skills
print(f"\nSkills ({len(plugin.skills)}):")
for skill in plugin.skills:
    desc = skill.description or ""
    print(f" - {skill.name}: {desc[:60]}...")
    if skill.trigger:
        print(f" Triggers: {skill.trigger}")

# Hooks
hook_config = plugin.hooks
has_hooks = hook_config is not None and not hook_config.is_empty()
print(f"\nHooks: {'Configured' if has_hooks else 'None'}")

# MCP Config
print(f"\nMCP Config: {'Configured' if plugin.mcp_config else 'None'}")
if plugin.mcp_config is not None:
    servers = plugin.mcp_config.get("mcpServers", {})
    for server_name in servers:
        print(f" - {server_name}")

# Loading all plugins
print("\n" + "=" * 80)
print("Part 3: Loading All Plugins from a Directory")
print("=" * 80)

plugins = Plugin.load_all(example_plugins_dir)
print(f"\nLoaded {len(plugins)} plugin(s) from {example_plugins_dir}")
for p in plugins:
    print(f" - {p.name} v{p.version}")

# Using Plugin Components with an Agent
print("\n" + "=" * 80)
print("Part 4: Using Plugin Components with an Agent")
print("=" * 80)

api_key = os.getenv("LLM_API_KEY")
if not api_key:
    print("Skipping agent demo (LLM_API_KEY not set)")
    print("\nTo run the full demo, set the LLM_API_KEY environment variable:")
    print(" export LLM_API_KEY=your-api-key")
    print("EXAMPLE_COST: 0")
    sys.exit(0)

model = os.getenv("LLM_MODEL", "anthropic/claude-sonnet-4-5-20250929")
llm = LLM(
    usage_id="plugin-demo",
    model=model,
    api_key=SecretStr(api_key),
    base_url=os.getenv("LLM_BASE_URL"),
)

agent_context = AgentContext(
    skills=plugin.skills,
    load_public_skills=False,
)

tools = [
    Tool(name=TerminalTool.name),
    Tool(name=FileEditorTool.name),
]

agent = Agent(
    llm=llm,
    tools=tools,
    agent_context=agent_context,
    mcp_config=plugin.mcp_config or {},
)

with tempfile.TemporaryDirectory() as tmpdir:
    conversation = Conversation(
        agent=agent,
        workspace=tmpdir,
        hook_config=plugin.hooks,
    )

    print("\n--- Demo 1: Skill Triggering ---")
    conversation.send_message(
        "How do I lint Python code? Just give a brief explanation."
    )
    conversation.run()

    print("\n--- Demo 2: Hook Execution ---")
    conversation.send_message(
        "Create a file called hello.py with a simple print statement."
    )
    conversation.run()

    print("\n--- Demo 3: MCP Tool Usage ---")
    conversation.send_message(
        "Use the fetch tool to get the content from https://httpbin.org/get "
        "and tell me what the 'origin' field contains."
    )
    conversation.run()

print(f"\nTotal cost: ${llm.metrics.accumulated_cost:.4f}")
print(f"EXAMPLE_COST: {llm.metrics.accumulated_cost:.4f}")
```

### Running the Example

You can run the example code as-is. The model name should follow the LiteLLM convention: `provider/model_name` (e.g., `anthropic/claude-sonnet-4-5-20250929`, `openai/gpt-4o`).

The `LLM_API_KEY` should be the API key for your chosen provider.

#### Bring-your-own provider key

```bash
export LLM_API_KEY="your-api-key"
export LLM_MODEL="anthropic/claude-sonnet-4-5-20250929" # or openai/gpt-4o, etc.
cd software-agent-sdk
uv run python examples/05_skills_and_plugins/02_loading_plugins/main.py
```

#### OpenHands Cloud

ChatGPT Plus/Pro subscribers: You can use `LLM.subscription_login()` to authenticate with your ChatGPT account and access Codex models without consuming API credits. See the LLM Subscriptions guide for details.

## Next Steps

- Skills - Learn more about skills and triggers
- Hooks - Understand hook event types
- MCP Integration - Configure external tool servers
