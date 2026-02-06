<!-- Source: https://docs.openhands.dev/sdk/guides/agent-server/api-reference/tools/list-available-tools -->

# List Available Tools

Retrieve a list of all available tools on the agent server.

## Endpoint

```
GET /api/tools/list
```

## Query Parameters

None required for this endpoint.

## Response

### 200 - Successful Response

Returns a list of all available tools configured on the agent server.

**Response Schema:**

```json
{
  "tools": [
    {
      "name": "string",
      "description": "string",
      "type": "string",
      "enabled": true
    }
  ]
}
```

**Response Fields:**

- **tools** (array) - List of available tools
  - **name** (string) - The tool name/identifier
  - **description** (string) - Human-readable description of what the tool does
  - **type** (string) - Tool type (e.g., "bash", "file", "browser", "vscode", etc.)
  - **enabled** (boolean) - Whether the tool is currently enabled

## cURL Example

```bash
curl --request GET \
  --url https://api.example.com/api/tools/list \
  --header 'Content-Type: application/json'
```

## Example Response

```json
{
  "tools": [
    {
      "name": "BashTool",
      "description": "Execute bash commands",
      "type": "bash",
      "enabled": true
    },
    {
      "name": "FileEditorTool",
      "description": "Read and write files",
      "type": "file",
      "enabled": true
    },
    {
      "name": "BrowserTool",
      "description": "Web browsing and interaction",
      "type": "browser",
      "enabled": false
    },
    {
      "name": "LogDataTool",
      "description": "Log structured data to JSON",
      "type": "custom",
      "enabled": true
    }
  ]
}
```

## Usage Notes

- The list includes all tools compiled into the server, both built-in and custom
- Use the `enabled` flag to determine which tools are active
- Tool availability depends on the agent server configuration and base image
- Custom tools are available if they were included in the base image with proper registration

## Common Tool Types

- **bash** - Execute shell commands
- **file** - Read and write files
- **browser** - Web browsing automation
- **vscode** - VS Code integration
- **custom** - User-defined tools

## Related Endpoints

- [Get Server Info](../server-details/get-server-info.md) - Get detailed server information
- [Health](../server-details/health.md) - Check server health status
- [Alive](../server-details/alive.md) - Check if server is running
