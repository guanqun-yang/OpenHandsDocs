<!-- Source: https://docs.openhands.dev/sdk/guides/agent-server/api-reference/bash/get-bash-event -->

# Get Bash Event

Retrieve a specific bash event by its ID.

## Endpoint

```
GET /api/bash/get_bash_event/{event_id}
```

## Path Parameters

**event_id** (string, required)
- The ID of the bash event to retrieve

## Response

### 200 - Successful Response

Returns the requested bash event.

**Response Schema:**

```json
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
```

**Response Fields:**

- **command_id** (string, required) - The ID of the bash command
- **kind** (string, required) - Type: "BashOutput"
- **id** (string) - Event ID
- **timestamp** (string, date-time) - Timestamp of the event
- **order** (integer) - The order for this output, sequentially starting with 0
- **exit_code** (integer | null) - Exit code. None implies the command is still running.
- **stdout** (string | null) - The standard output from the command
- **stderr** (string | null) - The error output from the command

### 404 - Not Found

The specified event ID does not exist.

### 422 - Unprocessable Entity

Validation error response.

## cURL Example

```bash
curl --request GET \
  --url https://api.example.com/api/bash/get_bash_event/event-id-123 \
  --header 'Content-Type: application/json'
```

## Related Endpoints

- [Start Bash Command](start-bash-command.md) - Start a bash command without waiting
- [Execute Bash Command](execute-bash-command.md) - Execute a command and wait for completion
- [Batch Get Bash Events](batch-get-bash-events.md) - Get multiple bash events
- [Search Bash Events](search-bash-events.md) - Search bash events
- [Clear All Bash Events](clear-all-bash-events.md) - Clear all bash event history
