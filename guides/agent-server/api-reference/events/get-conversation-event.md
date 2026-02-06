<!-- Source: https://docs.openhands.dev/sdk/guides/agent-server/api-reference/events/get-conversation-event -->

# Get Conversation Event

Retrieve a specific event from a conversation

## Endpoint

```
GET /api/conversations/{conversation_id}/events/{event_id}
```

## Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `conversation_id` | string<uuid> | Yes | The conversation ID |
| `event_id` | string<uuid> | Yes | The event ID |

## Response

**200** - Successful Response

