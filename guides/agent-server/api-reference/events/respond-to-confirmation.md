<!-- Source: https://docs.openhands.dev/sdk/guides/agent-server/api-reference/events/respond-to-confirmation -->

# Respond to Confirmation

Respond to a confirmation request event

## Endpoint

```
POST /api/conversations/{conversation_id}/events/{event_id}/confirm
```

## Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `conversation_id` | string<uuid> | Yes | The conversation ID |
| `event_id` | string<uuid> | Yes | The event ID |

## Request Body

**Content-Type:** `application/json`

Confirmation response

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `confirmed` | boolean | Yes | Whether the action is confirmed |

## Response

**200** - Successful Response

