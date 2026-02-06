<!-- Source: https://docs.openhands.dev/sdk/arch/tool-system -->

# Tool System & MCP

The Tool System provides a type-safe, extensible framework for defining agent capabilities. It standardizes how agents interact with external systems through a structured Action-Observation pattern with automatic validation and schema generation.

**Source:** openhands-sdk/openhands/sdk/tool/

## Core Responsibilities

The Tool System has four primary responsibilities:

1. **Type Safety** - Enforce action/observation schemas via Pydantic models
2. **Schema Generation** - Auto-generate LLM-compatible tool descriptions from Pydantic schemas
3. **Execution Lifecycle** - Validate inputs, execute logic, wrap outputs
4. **Tool Registry** - Discover and resolve tools by name or pattern

## Tool System

### Architecture Overview

### Key Components

| Component | Purpose | Design |
|-----------|---------|--------|
| ToolBase | Abstract base class | Generic over Action and Observation types, defines abstract create() |
| ToolDefinition | Concrete tool class | Can be instantiated directly or subclassed for factory pattern |
| Action | Input model | Pydantic model with visualize property |
| Observation | Output model | Pydantic model with to_llm_content property |
| ToolExecutor | Execution interface | ABC with __call__() method, optional close() |
| ToolAnnotations | Behavioral hints | MCP-spec hints (readOnly, destructive, idempotent, openWorld) |
| Tool (spec) | Tool specification | Configuration object with name and params |
| ToolRegistry | Tool discovery | Resolves Tool specs to ToolDefinition instances |

### Action-Observation Pattern

The tool system follows a strict input-output contract: Action → Observation. The Agent layer wraps these in events for conversation management.

**Tool System Boundary:**
- **Input:** dict[str, Any] (JSON arguments) → validated Action instance
- **Output:** Observation instance with structured result
- **No knowledge of:** Events, LLM messages, conversation state

## Tool Definition

Tools are defined using two patterns depending on complexity:

### Pattern 1: Direct Instantiation (Simple Tools)

For stateless tools that don't need runtime configuration (e.g., finish, think):

**Components:**
- **Action** - Pydantic model with visualize property for display
- **Observation** - Pydantic model with to_llm_content property for LLM
- **ToolExecutor** - Stateless executor with __call__(action) → observation
- **ToolDefinition** - Direct instantiation with executor instance

### Pattern 2: Subclass with Factory (Stateful Tools)

For tools requiring runtime configuration or persistent state (e.g., execute_bash, file_editor, glob):

**Components:**
- **Action/Observation** - Same as Pattern 1
- **ToolExecutor** - Stateful executor with __init__() for configuration and optional close() for cleanup
- **MyTool(ToolDefinition)** - Subclass with @classmethod create(conv_state, ...) factory method
- **Factory Method** - Returns sequence of configured tool instances

### Key Design Elements

| Component | Purpose | Requirements |
|-----------|---------|--------------|
| Action | Defines LLM-provided parameters | Extends Action, includes visualize property returning Rich Text |
| Observation | Defines structured output | Extends Observation, includes to_llm_content property returning content list |
| ToolExecutor | Implements business logic | Extends ToolExecutor[ActionT, ObservationT], implements __call__() method |
| ToolDefinition | Ties everything together | Either instantiate directly (Pattern 1) or subclass with create() method (Pattern 2) |

### When to Use Each Pattern

| Pattern | Use Case | Examples |
|---------|----------|----------|
| Direct Instantiation | Stateless tools with no configuration needs | finish, think, simple utilities |
| Subclass with Factory | Tools requiring runtime state or configuration | execute_bash, file_editor, glob, grep |

## Tool Annotations

Tools include optional ToolAnnotations based on the Model Context Protocol (MCP) spec that provide behavioral hints to LLMs:

| Field | Meaning | Examples |
|-------|---------|----------|
| readOnlyHint | Tool doesn't modify state | glob (True), execute_bash (False) |
| destructiveHint | May delete/overwrite data | file_editor (True), task_tracker (False) |
| idempotentHint | Repeated calls are safe | glob (True), execute_bash (False) |
| openWorldHint | Interacts beyond closed domain | execute_bash (True), task_tracker (False) |

**Key Behaviors:**
- LLM-based Security risk prediction automatically added for tools with readOnlyHint=False
- Annotations help LLMs reason about tool safety and side effects

## Tool Registry

The registry enables dynamic tool discovery and instantiation from tool specifications:

### Resolution Workflow

1. **Tool (Spec)** - Configuration object with name (e.g., "BashTool") and params (e.g., {"working_dir": "/workspace"})
2. **Resolver Lookup** - Registry finds the registered resolver for the tool name
3. **Factory Invocation** - Resolver calls the tool's .create() method with params and conversation state
4. **Instance Creation** - Tool instance(s) are created with configured executors
5. **Agent Usage** - Instances are added to the agent's tools_map for execution

### Registration Types

| Type | Registration | Resolver Behavior |
|------|--------------|------------------|
| Tool Instance | register_tool(name, instance) | Returns the fixed instance (params not allowed) |
| Tool Subclass | register_tool(name, ToolClass) | Calls ToolClass.create(**params, conv_state=state) |
| Factory Function | register_tool(name, factory) | Calls factory(**params, conv_state=state) |

### File Organization

Tools follow a consistent file structure for maintainability:

```
openhands-tools/openhands/tools/my_tool/
├── __init__.py                 # Export MyTool
├── definition.py               # Action, Observation, MyTool(ToolDefinition)
├── impl.py                     # MyExecutor(ToolExecutor)
└── [other modules]             # Tool-specific utilities
```

**File Responsibilities:**

| File | Contains | Purpose |
|------|----------|---------|
| definition.py | Action, Observation, ToolDefinition subclass | Public API, schema definitions, factory method |
| impl.py | ToolExecutor implementation | Business logic, state management, execution |
| __init__.py | Tool exports | Package interface |

**Benefits:**
- Separation of Concerns - Public API separate from implementation
- Avoid Circular Imports - Import impl only inside create() method
- Consistency - All tools follow same structure for discoverability

**Example Reference:** See execute_bash/ for complete implementation

## MCP Integration

The tool system supports external tools via the Model Context Protocol (MCP). MCP tools are configured separately from the tool registry via the mcp_config field in Agent class and are automatically discovered from MCP servers during agent initialization.

**Source:** openhands-sdk/openhands/sdk/mcp/

### Architecture Overview

### Key Components

| Component | Purpose | Design |
|-----------|---------|--------|
| MCPClient | MCP server connection | Extends FastMCP with sync/async bridge |
| MCPToolDefinition | Tool wrapper | Wraps MCP tools as SDK ToolDefinition with dynamic validation |
| MCPToolExecutor | Execution handler | Bridges agent actions to MCP tool calls via MCPClient |
| MCPToolAction | Generic action wrapper | Simple dict[str, Any] wrapper for MCP tool arguments |
| MCPToolObservation | Result wrapper | Wraps MCP tool results as observations with content blocks |
| _create_mcp_action_type() | Dynamic schema | Runtime Pydantic model generated from MCP input |

### Sync/Async Bridge

MCP protocol is asynchronous, but SDK tools execute synchronously. The bridge pattern in client.py solves this:

**Bridge Features:**
- Background Event Loop - Executes async code from sync contexts
- Timeout Support - Configurable timeouts for MCP operations
- Error Handling - Wraps MCP errors in observations
- Connection Pooling - Reuses connections across tool calls

### Tool Discovery Flow

**Source:** create_mcp_tools() | agent._initialize()

**Discovery Steps:**

1. **Spawn Server** - Launch MCP server via stdio protocol (using MCPClient)
2. **List Tools** - Call MCP tools/list endpoint to retrieve available tools
3. **Parse Schemas** - Extract tool names, descriptions, and inputSchema from MCP response
4. **Create Definitions** - For each tool, call MCPToolDefinition.create() which:
   - Creates an MCPToolExecutor instance bound to the tool name and client
   - Wraps the MCP tool metadata in MCPToolDefinition
   - Uses generic MCPToolAction as the action type (NOT dynamic models yet)
5. **Add to Agent** - All MCPToolDefinition instances are added to agent's tools_map during initialize() (bypasses ToolRegistry)
6. **Lazy Validation** - Dynamic Pydantic models are generated lazily when:
   - action_from_arguments() is called (argument validation)
   - to_openai_tool() is called (schema export to LLM)

**Schema Handling:**

| MCP Schema | SDK Integration | When Used |
|-----------|-----------------|-----------|
| name | Tool name (stored in MCPToolDefinition) | Discovery, execution |
| description | Tool description for LLM | Discovery, LLM prompting |
| inputSchema | Stored in mcp_tool.inputSchema | Lazy model generation |
| inputSchema fields | Converted to Pydantic fields via Schema.from_mcp_schema() | Validation, schema export |
| annotations | Mapped to ToolAnnotations | Security analysis, LLM hints |

### MCP Server Configuration

MCP servers are configured via the mcp_config field on the Agent class. Configuration follows FastMCP config format:

```python
from openhands.sdk import Agent

agent = Agent(
    mcp_config={
        "mcpServers": {
            "fetch": {
                "command": "uvx",
                "args": ["mcp-server-fetch"]
            },
            "filesystem": {
                "command": "npx",
                "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path"]
            }
        }
    }
)
```

## Component Relationships

**Relationship Characteristics:**
- **Native → Registry → tools_map:** Native tools resolved via ToolRegistry
- **MCP → tools_map:** MCP tools bypass registry, added directly during initialize()
- **tools_map → LLM:** Generate schemas describing all available capabilities
- **Agent → tools_map:** Execute actions, receive observations
- **tools_map → Conversation:** Read state for context-aware execution
- **tools_map → Security:** Tool annotations inform risk assessment

## See Also

- **Agent Architecture** - How agents select and execute tools
- **Events** - ActionEvent and ObservationEvent structures
- **Security Analyzer** - Action risk assessment
- **Skill Architecture** - Embedding MCP configs in repository skills
- **Custom Tools Guide** - Building your own tools
- **FastMCP Documentation** - Underlying MCP client library
