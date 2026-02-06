<!-- Source: https://docs.openhands.dev/sdk/arch/agent -->

# Agent

The Agent component implements the core reasoning-action loop that drives autonomous task execution. It orchestrates LLM queries, tool execution, and context management through a stateless, event-driven architecture.

**Source:** openhands-sdk/openhands/sdk/agent/

## Core Responsibilities

The Agent system has four primary responsibilities:

1. **Reasoning-Action Loop** - Query LLM to generate next actions based on conversation history
2. **Tool Orchestration** - Select and execute tools, handle results and errors
3. **Context Management** - Apply skills, manage conversation history via condensers
4. **Security Validation** - Analyze proposed actions for safety before execution via security analyzer

## Architecture

The agent orchestrates multiple subsystems:

- Agent Core
- Skills
- Feedback
- Event History
- Agent Context (Skills + Prompts)
- Condenser (History compression)
- LLM Query (Generate actions)
- Security Analyzer (Risk assessment)
- Tool Executor (Action → Observation)
- Observation Events

## Key Components

| Component | Purpose | Design |
|-----------|---------|--------|
| Agent | Main implementation | Stateless reasoning-action loop executor |
| AgentBase | Abstract base class | Defines agent interface and initialization |
| AgentContext | Context container | Manages skills, prompts, and metadata |
| Condenser | History compression | Reduces context when token limits approaches |
| SecurityAnalyzer | Safety validation | Evaluates action risk before execution |

## Reasoning-Action Loop

The agent operates through a single-step execution model where each `step()` call processes one reasoning cycle:

### Step Execution Flow

1. **Pending Actions:** If actions awaiting confirmation exist, execute them and return
2. **Condensation:** If condenser exists:
   - Call `condenser.condense()` with current event view
   - If returns View: use condensed events for LLM query (continue in same step)
   - If returns Condensation: emit event and return (will be processed next step)
3. **LLM Query:** Query LLM with messages from event history
   - If context window exceeded: emit CondensationRequest and return
4. **Response Parsing:** Parse LLM response into events
   - Tool calls → create ActionEvent(s)
   - Text message → create MessageEvent and return
5. **Confirmation Check:** If actions need user approval: Set conversation status to WAITING_FOR_CONFIRMATION and return
6. **Action Execution:** Execute tools and create ObservationEvent(s)

### Key Characteristics

- **Stateless:** Agent holds no mutable state between steps
- **Event-Driven:** Reads from event history, writes new events
- **Interruptible:** Each step is atomic and can be paused/resumed

## Agent Context

The agent applies AgentContext which includes skills and prompts to shape LLM behavior:

| Skill Type | When triggered | Use Case |
|-----------|----------------|----------|
| repo | Always included | Project-specific context, conventions |
| knowledge | Trigger-based | Domain knowledge, special behaviors |
| System prompt | Per-conversation | - |
| Prompt template | Per-conversation | - |

## Tool Execution

Tools follow a strict action-observation pattern:

### Execution Modes

| Mode | Behavior | Use Case |
|------|----------|----------|
| Direct | Execute immediately | Development, trusted environments |
| Confirmation | Store as pending, wait for user approval | High-risk actions, production |

### Security Integration

Before execution, the security analyzer evaluates each action:

- **Low Risk:** Execute immediately
- **Medium Risk:** Log warning, execute with monitoring
- **High Risk:** Block execution, request user confirmation

## Component Relationships

### How Agent Interacts

- **Conversation → Agent:** Orchestrates step execution, provides event history
- **Agent → LLM:** Queries for next actions, receives tool calls or messages
- **Agent → Tools:** Executes actions, receives observations
- **AgentContext → Agent:** Injects skills and prompts into LLM queries

## See Also

- **Conversation Architecture** - Agent orchestration and lifecycle
- **Tool System** - Tool definition and execution patterns
- **Events** - Event types and structures
- **Skills** - Prompt engineering and skill patterns
- **LLM** - Language model abstraction
