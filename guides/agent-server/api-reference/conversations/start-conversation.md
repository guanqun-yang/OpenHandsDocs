<!-- Source: https://docs.openhands.dev/sdk/guides/agent-server/api-reference/conversations/start-conversation -->

# Start Conversation

Start a new conversation

## Endpoint

```
POST /api/conversations
```

## Request Body

**Content-Type:** `application/json`

Configuration for starting a new conversation

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `agent` | object | Yes | Agent configuration |
| `llm` | object | Yes | LLM configuration |

## Response

**200** - Successful Response

