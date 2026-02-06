<!-- Source: https://docs.openhands.dev/sdk/guides/agent-server/api-reference/conversations/search-conversations -->

# Search Conversations

Search conversations by keyword with optional filters

## Endpoint

```
GET /api/conversations
```

## Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `query` | string | No | Search query text |
| `limit` | integer | No | Maximum number of results |
| `offset` | integer | No | Number of results to skip |

## Response

**200** - Successful Response

