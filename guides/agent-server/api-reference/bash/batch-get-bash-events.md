<!-- Source: https://docs.openhands.dev/sdk/guides/agent-server/api-reference/bash/batch-get-bash-events -->

# Batch Get Bash Events

Retrieve multiple bash events by their IDs.

## Endpoint

```
POST /api/bash/batch_get_bash_events
```

## Request Body

### Parameters

**event_ids** (array of strings, required)
- List of event IDs to retrieve

### Example Request

```json
{
  "event_ids": [
    "event-id-1",
    "event-id-2",
    "event-id-3"
  ]
}
```

## Response

### 200 - Successful Response

Returns an array of bash events.

**Response Schema:**

```json
[
  {
    "command_id": "string",
    "kind": "BashOutput",
    "id": "string",
    "timestamp": "2023-11-07T05:31:56Z",
    "order": 0,
    "exit_code": 0,
    "stdout": "string",
    "stderr": "string"
  }
]
```

**Response Fields (per event):**

- **command_id** (string, required) - The ID of the bash command
- **kind** (string, required) - Type: "BashOutput"
- **id** (string) - Event ID
- **timestamp** (string, date-time) - Timestamp of the event
- **order** (integer) - The order for this output, sequentially starting with 0
- **exit_code** (integer | null) - Exit code. None implies the command is still running.
- **stdout** (string | null) - The standard output from the command
- **stderr** (string | null) - The error output from the command

### 422 - Unprocessable Entity

Validation error response.

## cURL Example

```bash
curl --request POST \
  --url https://api.example.com/api/bash/batch_get_bash_events \
  --header 'Content-Type: application/json' \
  --data '{
    "event_ids": [
      "event-id-1",
      "event-id-2",
      "event-id-3"
    ]
  }'
```

## Usage Notes

- Only retrieves events that exist; missing events are silently omitted from the response
- Useful for retrieving multiple events in a single request instead of making multiple individual requests
- Events are returned in the order requested

## Related Endpoints

- [Start Bash Command](start-bash-command.md) - Start a bash command without waiting
- [Execute Bash Command](execute-bash-command.md) - Execute a command and wait for completion
- [Get Bash Event](get-bash-event.md) - Get a specific bash event
- [Search Bash Events](search-bash-events.md) - Search bash events
- [Clear All Bash Events](clear-all-bash-events.md) - Clear all bash event history
