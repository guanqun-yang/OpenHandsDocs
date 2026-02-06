<!-- Source: https://docs.openhands.dev/sdk/guides/agent-server/api-reference/conversations/update-conversation-secrets -->

# Update Conversation Secrets

Update secrets for a conversation

## Endpoint

```
POST /api/conversations/{conversation_id}/secrets
```

## Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `conversation_id` | string<uuid> | Yes | The conversation ID |

## Request Body

**Content-Type:** `application/json`

Secrets to update

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `secrets` | object | Yes | Key-value pairs of secrets |

## Response

**200** - Successful Response

