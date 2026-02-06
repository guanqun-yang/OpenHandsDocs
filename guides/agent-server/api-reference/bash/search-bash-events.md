<!-- Source: https://docs.openhands.dev/sdk/guides/agent-server/api-reference/bash/search-bash-events -->

# Search Bash Events

Search for bash events using various filter criteria.

## Endpoint

```
POST /api/bash/search_bash_events
```

## Request Body

### Parameters

**command_id** (string, optional)
- Filter events by command ID

**limit** (integer, optional, default: 100)
- Maximum number of events to return

**offset** (integer, optional, default: 0)
- Number of events to skip from the beginning

**sort_by** (string, optional, default: "timestamp")
- Field to sort results by (e.g., "timestamp", "order")

**sort_order** (string, optional, default: "desc")
- Sort order: "asc" for ascending or "desc" for descending

### Example Request

```json
{
  "command_id": "cmd-123",
  "limit": 50,
  "offset": 0,
  "sort_by": "timestamp",
  "sort_order": "desc"
}
```

## Response

### 200 - Successful Response

Returns an array of bash events matching the search criteria.

**Response Schema:**

```json
{
  "events": [
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
  ],
  "total": 100
}
```

**Response Fields:**

- **events** (array) - List of matching bash events
- **total** (integer) - Total number of events matching the search criteria

**Event Fields:**

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
  --url https://api.example.com/api/bash/search_bash_events \
  --header 'Content-Type: application/json' \
  --data '{
    "command_id": "cmd-123",
    "limit": 50,
    "offset": 0,
    "sort_by": "timestamp",
    "sort_order": "desc"
  }'
```

## Pagination Example

To paginate through results:

```bash
# First page
curl --request POST \
  --url https://api.example.com/api/bash/search_bash_events \
  --header 'Content-Type: application/json' \
  --data '{
    "command_id": "cmd-123",
    "limit": 20,
    "offset": 0
  }'

# Second page
curl --request POST \
  --url https://api.example.com/api/bash/search_bash_events \
  --header 'Content-Type: application/json' \
  --data '{
    "command_id": "cmd-123",
    "limit": 20,
    "offset": 20
  }'
```

## Related Endpoints

- [Start Bash Command](start-bash-command.md) - Start a bash command without waiting
- [Execute Bash Command](execute-bash-command.md) - Execute a command and wait for completion
- [Get Bash Event](get-bash-event.md) - Get a specific bash event
- [Batch Get Bash Events](batch-get-bash-events.md) - Get multiple bash events
- [Clear All Bash Events](clear-all-bash-events.md) - Clear all bash event history
