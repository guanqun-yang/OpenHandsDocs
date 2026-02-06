<!-- Source: https://docs.openhands.dev/sdk/guides/agent-server/api-reference/bash/start-bash-command -->

# Start Bash Command

Start a bash command without waiting for it to complete.

## Endpoint

```
POST /api/bash/start_bash_command
```

## Request Body

### Parameters

**command** (string, required)
- The bash command to execute

**cwd** (string | null, optional)
- The current working directory

**timeout** (integer, optional, default: 300)
- The max number of seconds a command may be permitted to run.

### Example Request

```json
{
  "command": "long-running-process",
  "cwd": "/home/user",
  "timeout": 600
}
```

## Response

### 200 - Successful Response

Returns the command ID for the started bash command.

**Response Schema:**

```json
{
  "command_id": "string"
}
```

**Response Fields:**

- **command_id** (string, required) - The unique identifier for the started bash command. Use this to retrieve events with `get_bash_event` or `batch_get_bash_events`.

### 422 - Unprocessable Entity

Validation error response.

## cURL Example

```bash
curl --request POST \
  --url https://api.example.com/api/bash/start_bash_command \
  --header 'Content-Type: application/json' \
  --data '{
    "command": "long-running-process",
    "cwd": "/home/user",
    "timeout": 600
  }'
```

## Usage Pattern

1. Call `start_bash_command` to start a long-running command
2. Use the returned `command_id` with `get_bash_event` or `batch_get_bash_events` to monitor progress
3. Poll for events until the command completes (when `exit_code` is not null)

## Related Endpoints

- [Execute Bash Command](execute-bash-command.md) - Execute a command and wait for completion
- [Get Bash Event](get-bash-event.md) - Get a specific bash event
- [Batch Get Bash Events](batch-get-bash-events.md) - Get multiple bash events
- [Search Bash Events](search-bash-events.md) - Search bash events
- [Clear All Bash Events](clear-all-bash-events.md) - Clear all bash event history
