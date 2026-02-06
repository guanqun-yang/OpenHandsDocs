<!-- Source: https://docs.openhands.dev/sdk/guides/agent-server/api-reference/events/send-message -->

# Send Message

Send a message to the conversation

## Endpoint

```
POST /api/conversations/{conversation_id}/messages
```

## Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `conversation_id` | string<uuid> | Yes | The conversation ID |

## Request Body

**Content-Type:** `application/json`

Message to send

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `message` | string | Yes | The message content |

## Response

**200** - Successful Response

