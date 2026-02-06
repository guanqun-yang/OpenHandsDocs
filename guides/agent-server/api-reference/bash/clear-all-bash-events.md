<!-- Source: https://docs.openhands.dev/sdk/guides/agent-server/api-reference/bash/clear-all-bash-events -->

# Clear All Bash Events

Clear all bash events from the event history.

## Endpoint

```
POST /api/bash/clear_all_bash_events
```

## Request Body

No request body required for this endpoint.

### Example Request

```bash
curl --request POST \
  --url https://api.example.com/api/bash/clear_all_bash_events \
  --header 'Content-Type: application/json'
```

## Response

### 200 - Successful Response

Returns a success message indicating all bash events have been cleared.

**Response Schema:**

```json
{
  "success": true,
  "message": "All bash events have been cleared"
}
```

**Response Fields:**

- **success** (boolean) - Indicates if the operation was successful
- **message** (string) - Human-readable status message

### 422 - Unprocessable Entity

Validation error response.

## cURL Example

```bash
curl --request POST \
  --url https://api.example.com/api/bash/clear_all_bash_events \
  --header 'Content-Type: application/json'
```

## Warning

This operation is destructive and cannot be undone. All bash event history will be permanently deleted. Use with caution in production environments.

## Usage Notes

- Clears all bash events for the entire agent server, not just for a specific command or session
- Useful for cleaning up event history when debugging or resetting state
- Can significantly reduce memory usage if many events have accumulated

## Related Endpoints

- [Start Bash Command](start-bash-command.md) - Start a bash command without waiting
- [Execute Bash Command](execute-bash-command.md) - Execute a command and wait for completion
- [Get Bash Event](get-bash-event.md) - Get a specific bash event
- [Batch Get Bash Events](batch-get-bash-events.md) - Get multiple bash events
- [Search Bash Events](search-bash-events.md) - Search bash events
