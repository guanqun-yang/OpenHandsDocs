<!-- Source: https://docs.openhands.dev/sdk/guides/agent-server/api-reference/conversations/batch-get-conversations -->

# Batch Get Conversations

Get a batch of conversations given their IDs, returning null for any missing item

## Endpoint

```
GET /api/conversations
```

## Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `ids` | string<uuid>[] | Yes | List of conversation IDs |

## Response

**200** - Successful Response

