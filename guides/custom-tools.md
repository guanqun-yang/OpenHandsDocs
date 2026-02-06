<!-- Source: https://docs.openhands.dev/sdk/guides/custom-tools -->

# Custom Tools

The ready-to-run example is available [here](#ready-to-run-example)!

## Understanding the Tool System

The SDK's tool system is built around three core components:

1. **Action** - Defines input parameters (what the tool accepts)
2. **Observation** - Defines output data (what the tool returns)
3. **Executor** - Implements the tool's logic (what the tool does)

These components are tied together by a `ToolDefinition` that registers the tool with the agent.

## Built-in Tools

The [tools package](https://github.com/OpenHands/software-agent-sdk/tree/main/openhands-tools/openhands/tools) provides a bunch of built-in tools that follow these patterns:

```python
from openhands.tools import BashTool, FileEditorTool
from openhands.tools.preset import get_default_tools

# Use specific tools
agent = Agent(llm=llm, tools=[BashTool.create(), FileEditorTool.create()])

# Or use preset tools
tools = get_default_tools()
agent = Agent(llm=llm, tools=tools)
```

See [source code](https://github.com/OpenHands/software-agent-sdk/tree/main/openhands-tools/openhands/tools) for the complete list of available tools and design philosophy.

## Creating a Custom Tool

Here's a minimal example of creating a custom grep tool:

### 1. Define the Action

Defines input parameters (what the tool accepts):

```python
class GrepAction(Action):
    pattern: str = Field(description="Regex to search for")
    path: str = Field(
        default=".",
        description="Directory to search (absolute or relative)"
    )
    include: str | None = Field(
        default=None,
        description="Optional glob to filter files (e.g. '*.py')"
    )
```

### 2. Define the Observation

Defines output data (what the tool returns):

```python
class GrepObservation(Observation):
    matches: list[str] = Field(default_factory=list)
    files: list[str] = Field(default_factory=list)
    count: int = 0

    @property
    def to_llm_content(self) -> Sequence[TextContent | ImageContent]:
        if not self.count:
            return [TextContent(text="No matches found.")]

        files_list = "\n".join(f"- {f}" for f in self.files[:20])
        sample = "\n".join(self.matches[:10])
        more = "\n..." if self.count > 10 else ""

        ret = (
            f"Found {self.count} matching lines.\n"
            f"Files:\n{files_list}\n"
            f"Sample:\n{sample}{more}"
        )
        return [TextContent(text=ret)]
```

The `to_llm_content()` property formats observations for the LLM.

### 3. Define the Executor

Implements the tool's logic (what the tool does):

```python
class GrepExecutor(ToolExecutor[GrepAction, GrepObservation]):
    def __init__(self, terminal: TerminalExecutor):
        self.terminal: TerminalExecutor = terminal

    def __call__(
        self, action: GrepAction, conversation=None,
    ) -> GrepObservation:
        root = os.path.abspath(action.path)
        pat = shlex.quote(action.pattern)
        root_q = shlex.quote(root)

        # Use grep -r; add --include when provided
        if action.include:
            inc = shlex.quote(action.include)
            cmd = f"grep -rHnE --include {inc} {pat} {root_q}"
        else:
            cmd = f"grep -rHnE {pat} {root_q}"

        cmd += " 2>/dev/null | head -100"
        result = self.terminal(TerminalAction(command=cmd))

        matches: list[str] = []
        files: set[str] = set()

        output_text = result.text
        if output_text.strip():
            for line in output_text.strip().splitlines():
                matches.append(line)
                # Expect "path:line:content"
                # take the file part before first ":"
                file_path = line.split(":", 1)[0]
                if file_path:
                    files.add(os.path.abspath(file_path))

        return GrepObservation(
            matches=matches,
            files=sorted(files),
            count=len(matches),
        )
```

### 4. Finally, define the tool

```python
class GrepTool(ToolDefinition[GrepAction, GrepObservation]):
    """Custom grep tool that searches file contents using regular expressions."""

    @classmethod
    def create(
        cls, conv_state, terminal_executor: TerminalExecutor | None = None
    ) -> Sequence[ToolDefinition]:
        """Create GrepTool instance with a GrepExecutor.

        Args:
            conv_state: Conversation state to get working directory from.
            terminal_executor: Optional terminal executor to reuse.
                If not provided, a new one will be created.

        Returns:
            A sequence containing a single GrepTool instance.
        """
        if terminal_executor is None:
            terminal_executor = TerminalExecutor(
                working_dir=conv_state.workspace.working_dir
            )

        grep_executor = GrepExecutor(terminal_executor)
        return [
            cls(
                description=_GREP_DESCRIPTION,
                action_type=GrepAction,
                observation_type=GrepObservation,
                executor=grep_executor,
            )
        ]
```

## Good to know

### Tool Registration

Tools are registered using `register_tool()` and referenced by name:

```python
# Register a simple tool class
register_tool("FileEditorTool", FileEditorTool)

# Register a factory function that creates multiple tools
register_tool("BashAndGrepToolSet", _make_bash_and_grep_tools)

# Use registered tools by name
tools = [
    Tool(name="FileEditorTool"),
    Tool(name="BashAndGrepToolSet"),
]
```

### Factory Functions

Tool factory functions receive `conv_state` as a parameter, allowing access to workspace information:

```python
def _make_bash_and_grep_tools(conv_state) -> list[ToolDefinition]:
    """Create execute_bash and custom grep tools sharing one executor."""
    bash_executor = BashExecutor(
        working_dir=conv_state.workspace.working_dir
    )

    # Create and configure tools...
    return [bash_tool, grep_tool]
```

### Shared Executors

Multiple tools can share executors for efficiency and state consistency:

```python
bash_executor = BashExecutor(working_dir=conv_state.workspace.working_dir)
bash_tool = execute_bash_tool.set_executor(executor=bash_executor)

grep_executor = GrepExecutor(bash_executor)
grep_tool = ToolDefinition(
    name="grep",
    description=_GREP_DESCRIPTION,
    action_type=GrepAction,
    observation_type=GrepObservation,
    executor=grep_executor,
)
```

## When to Create Custom Tools

Create custom tools when you need to:

- Combine multiple operations into a single, structured interface
- Add typed parameters with validation
- Format complex outputs for LLM consumption
- Integrate with external APIs or services

## Ready-to-run Example

This example is available on GitHub: [examples/01_standalone_sdk/02_custom_tools.py](https://github.com/OpenHands/software-agent-sdk/blob/main/examples/01_standalone_sdk/02_custom_tools.py)

You can run the example code as-is.

**Note:** The model name should follow the [LiteLLM convention](https://models.litellm.ai/): `provider/model_name` (e.g., `anthropic/claude-sonnet-4-5-20250929`, `openai/gpt-4o`). The `LLM_API_KEY` should be the API key for your chosen provider.

### Running the Example

#### Bring-your-own provider key

```bash
export LLM_API_KEY="your-api-key"
export LLM_MODEL="anthropic/claude-sonnet-4-5-20250929"  # or openai/gpt-4o, etc.
cd software-agent-sdk
uv run python examples/01_standalone_sdk/02_custom_tools.py
```

#### OpenHands Cloud

```bash
cd software-agent-sdk
uv run python examples/01_standalone_sdk/02_custom_tools.py
```

**Tip:** ChatGPT Plus/Pro subscribers: You can use `LLM.subscription_login()` to authenticate with your ChatGPT account and access Codex models without consuming API credits. See the [LLM Subscriptions guide](/sdk/guides/llm-subscriptions) for details.

## Next Steps

- [Model Context Protocol (MCP) Integration](/sdk/guides/mcp) - Use Model Context Protocol servers
- [Tools Package Source Code](https://github.com/OpenHands/software-agent-sdk/tree/main/openhands-tools/openhands/tools) - Built-in tools implementation
