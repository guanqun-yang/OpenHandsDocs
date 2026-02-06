<!-- Source: https://docs.openhands.dev/sdk/guides/agent-server/api-reference/events/batch-get-conversation-events -->

# Batch Get Conversation Events

Get a batch of conversation events by their IDs

## Endpoint

```
GET /api/conversations/{conversation_id}/events
```

## Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `conversation_id` | string<uuid> | Yes | The conversation ID |

## Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `event_ids` | string<uuid>[] | Yes | List of event IDs to retrieve |

## Response

**200** - Successful Response

