<!-- Source: https://docs.openhands.dev/sdk/guides/agent-server/api-reference/conversations/condense-conversation -->

# Condense Conversation

Condense conversation history to reduce context size

## Endpoint

```
POST /api/conversations/{conversation_id}/condense
```

## Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `conversation_id` | string<uuid> | Yes | The conversation ID |

## Response

**200** - Successful Response

