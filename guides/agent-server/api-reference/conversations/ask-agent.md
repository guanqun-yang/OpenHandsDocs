<!-- Source: https://docs.openhands.dev/sdk/guides/agent-server/api-reference/conversations/ask-agent -->

# Ask Agent

Ask the agent a simple question without affecting conversation state

## Endpoint

```
POST /api/conversations/{conversation_id}/ask_agent
```

## Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `conversation_id` | string<uuid> | Yes | The conversation ID |

## Request Body

**Content-Type:** `application/json`

The question to ask

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `question` | string | Yes | The question text |

## Response

**200** - Successful Response

