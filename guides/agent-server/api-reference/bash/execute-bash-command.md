<!-- Source: https://docs.openhands.dev/sdk/guides/agent-server/api-reference/bash/execute-bash-command -->

# Execute Bash Command

Execute a bash command and wait for a result.

## Endpoint

```
POST /api/bash/execute_bash_command
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
  "command": "ls -la",
  "cwd": "/home/user",
  "timeout": 300
}
```

## Response

### 200 - Successful Response

Returns the output of the bash command. A single command may have multiple pieces of output depending on how large.

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
- **timestamp** (string, date-time) - Timestamp of the output
- **order** (integer) - The order for this output, sequentially starting with 0
- **exit_code** (integer | null) - Exit code. None implies the command is still running.
- **stdout** (string | null) - The standard output from the command
- **stderr** (string | null) - The error output from the command

### 422 - Unprocessable Entity

Validation error response.

## cURL Example

```bash
curl --request POST \
  --url https://api.example.com/api/bash/execute_bash_command \
  --header 'Content-Type: application/json' \
  --data '{
    "command": "ls -la",
    "cwd": "/home/user",
    "timeout": 300
  }'
```

## Related Endpoints

- [Start Bash Command](start-bash-command.md) - Start a bash command without waiting
- [Get Bash Event](get-bash-event.md) - Get a specific bash event
- [Batch Get Bash Events](batch-get-bash-events.md) - Get multiple bash events
- [Search Bash Events](search-bash-events.md) - Search bash events
- [Clear All Bash Events](clear-all-bash-events.md) - Clear all bash event history
