<!-- Source: https://docs.openhands.dev/sdk/guides/agent-server/api-reference/get-server-info -->

# Get Server Info

Retrieve detailed information about the agent server configuration and capabilities.

## Endpoint

```
GET /api/server/info
```

## Query Parameters

None required for this endpoint.

## Response

### 200 - Successful Response

Returns detailed server information.

**Response Schema:**

```json
{
  "version": "0.1.0",
  "python_version": "3.12.0",
  "os": "Linux",
  "hostname": "agent-server-1",
  "workspace_dir": "/home/agent/workspace",
  "max_file_size_mb": 100,
  "tools": ["bash", "files", "vscode", "browser"],
  "vscode_enabled": true,
  "desktop_enabled": true,
  "api_port": 8000,
  "uptime_seconds": 3600,
  "timestamp": "2023-11-07T05:31:56Z"
}
```

**Response Fields:**

- **version** (string) - Agent server version
- **python_version** (string) - Python version running on the server
- **os** (string) - Operating system (Linux, macOS, Windows, etc.)
- **hostname** (string) - Server hostname
- **workspace_dir** (string) - Default workspace directory
- **max_file_size_mb** (integer) - Maximum file size supported for uploads (MB)
- **tools** (array) - List of available tools
- **vscode_enabled** (boolean) - Whether VS Code is enabled
- **desktop_enabled** (boolean) - Whether desktop/VNC is enabled
- **api_port** (integer) - Port where API is listening
- **uptime_seconds** (integer) - Server uptime in seconds
- **timestamp** (string, date-time) - Current server time

## cURL Example

```bash
curl --request GET \
  --url https://api.example.com/api/server/info \
  --header 'Content-Type: application/json'
```

## Example Response

```json
{
  "version": "0.1.0",
  "python_version": "3.12.5",
  "os": "Linux",
  "hostname": "agent-docker-1",
  "workspace_dir": "/app/workspace",
  "max_file_size_mb": 100,
  "tools": ["bash", "files", "vscode", "browser"],
  "vscode_enabled": true,
  "desktop_enabled": true,
  "api_port": 8000,
  "uptime_seconds": 7200,
  "timestamp": "2023-11-07T10:31:56Z"
}
```

## Usage Notes

- Provides comprehensive server configuration information
- Useful for verifying server capabilities before making requests
- Can be used to display server metadata in UI/dashboards
- Tool list helps determine what operations are available

## Common Use Cases

1. **Capability verification** - Check if required tools are available
2. **Version checking** - Verify server version compatibility
3. **Configuration inspection** - Review server settings
4. **Debugging** - Gather diagnostic information
5. **Monitoring** - Track server uptime and configuration

## Example Usage

```python
import httpx

async def get_server_capabilities():
    response = httpx.get(f"{server_url}/api/server/info")
    info = response.json()

    print(f"Server Version: {info['version']}")
    print(f"Python: {info['python_version']}")
    print(f"Available Tools: {', '.join(info['tools'])}")

    if "browser" in info['tools']:
        print("Browser automation is available")

    if info['vscode_enabled']:
        print("VS Code Web is enabled")

    return info
```

## Related Endpoints

- [Alive](server-details/alive.md) - Check if server is running
- [Health](server-details/health.md) - Get detailed health status
- [List Available Tools](tools/list-available-tools.md) - Get list of tools
