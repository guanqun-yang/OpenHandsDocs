<!-- Source: https://docs.openhands.dev/sdk/arch/condenser -->

# Condenser

The Condenser system manages conversation history compression to keep agent context within LLM token limits. It reduces long event histories into condensed summaries while preserving critical information for reasoning. For more details, read the blog here.

**Source:** openhands-sdk/openhands/sdk/context/condenser/

## Core Responsibilities

The Condenser system has four primary responsibilities:

1. **History Compression** - Reduce event lists to fit within context windows
2. **Threshold Detection** - Determine when condensation should trigger
3. **Summary Generation** - Create meaningful summaries via LLM or heuristics
4. **View Management** - Transform event history into LLM-ready views

## Architecture

The condenser provides pluggable implementations for different condensation strategies.

## Key Components

| Component | Purpose | Design |
|-----------|---------|--------|
| CondenserBase | Abstract interface | Defines condense() contract |
| RollingCondenser | Rolling window base | Implements threshold-based triggering |
| LLMSummarizingCondenser | LLM summarization | Uses LLM to generate summaries |
| NoOpCondenser | No-op implementation | Returns view unchanged |
| PipelineCondenser | Multi-stage pipeline | Chains multiple condensers |
| View | Event view | Represents history for LLM |
| Condensation | Condensation event | Metadata about compression |

## Condenser Types

### NoOpCondenser

Pass-through condenser that performs no compression.

### LLMSummarizingCondenser

Uses an LLM to generate summaries of conversation history:

**Process:**
1. **Check Threshold:** Compare view size to configured limit (e.g., event count > max_size)
2. **Select Events:** Identify events to keep (first N + last M) and events to summarize (middle)
3. **LLM Call:** Generate summary of middle events using dedicated LLM
4. **Create Event:** Wrap summary in Condensation event with forgotten_event_ids
5. **Add to History:** Agent adds Condensation to event log and returns early
6. **Next Step:** View.from_events() filters forgotten events and inserts summary

**Configuration:**
- **max_size:** Event count threshold before condensation triggers (default: 120)
- **keep_first:** Number of initial events to preserve verbatim (default: 4)
- **llm:** LLM instance for summarization (often cheaper model than reasoning LLM)

### PipelineCondenser

Chains multiple condensers in sequence.

**Use Case:** Multi-stage compression (e.g., remove old events, then summarize, then truncate)

## Condensation Flow

### Trigger Mechanisms

Condensers can be triggered in two ways:

**Automatic Trigger:**
- **When:** Threshold exceeded (e.g., event count > max_size)
- **Who:** Agent calls condenser.condense() each step
- **Purpose:** Proactively keep context within limits

**Manual Trigger:**
- **When:** CondensationRequest event added to history (via view.unhandled_condensation_request)
- **Who:** Agent (on LLM context window error) or application code
- **Purpose:** Force compression when context limit exceeded

### Condensation Workflow

**Key Steps:**

1. **Threshold Check:** should_condense() determines if condensation needed
2. **Event Selection:** Identify events to keep (head + tail) vs forget (middle)
3. **Summary Generation:** LLM creates compressed representation of forgotten events
4. **Condensation Creation:** Create Condensation event with forgotten_event_ids and summary
5. **Return to Agent:** Condenser returns Condensation (not View)
6. **History Update:** Agent adds Condensation to event log and exits step
7. **Next Step:** View.from_events() processes Condensation to filter events and insert summary

## View and Condensation

### View Structure

A View represents the conversation history as it will be sent to the LLM:

**View Components:**
- **events:** List of LLMConvertibleEvent objects (filtered by Condensation)
- **unhandled_condensation_request:** Flag for pending manual condensation
- **condensations:** List of all Condensation events processed

**Methods:**
- from_events() creates view from raw events, handling Condensation semantics

### Condensation Event

When condensation occurs, a Condensation event is created:

**Condensation Fields:**
- **forgotten_event_ids:** List of event IDs to filter out
- **summary:** Compressed text representation of forgotten events
- **summary_offset:** Index where summary event should be inserted
- Inherits from Event: id, timestamp, source

## Rolling Window Pattern

RollingCondenser implements a common pattern for threshold-based condensation:

**Rolling Window Strategy:**
- **Keep Head:** Preserve first keep_first events (default: 4) - usually system prompts
- **Keep Tail:** Preserve last target_size - keep_first - 1 events - recent context
- **Summarize Middle:** Compress events between head and tail into summary
- **Target Size:** After condensation, view has max_size // 2 events (default: 60)

## Component Relationships

### How Condenser Integrates

- **Agent → State:** Calls View.from_events() to get current view
- **Agent → Condenser:** Calls condense(view) each step if condenser registered
- **Condenser → Agent:** Returns View (proceed) or Condensation (defer)
- **Agent → Events:** Adds Condensation event to log when returned

## See Also

- **Agent Architecture** - How agents use condensers during reasoning
- **Conversation Architecture** - View generation and event management
- **Events** - Condensation event type and append-only log
- **Context Condenser Guide** - Configuring and using condensers
