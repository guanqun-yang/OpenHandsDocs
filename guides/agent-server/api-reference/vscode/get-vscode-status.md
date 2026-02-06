<!-- Source: https://docs.openhands.dev/sdk/guides/agent-server/api-reference/vscode/get-vscode-status -->

# Get VSCode Status

Check the status and availability of the VS Code service.

## Endpoint

```
GET /api/vscode/status
```

## Query Parameters

None required for this endpoint.

## Response

### 200 - Successful Response

Returns the status of the VS Code service.

**Response Schema:**

```json
{
  "status": "ready",
  "running": true,
  "port": 8011,
  "message": "VS Code server is running and ready"
}
```

**Response Fields:**

- **status** (string) - Current status ("ready", "starting", "stopping", "error")
- **running** (boolean) - Whether the VS Code service is running
- **port** (integer) - The port where VS Code is accessible
- **message** (string) - Human-readable status message

### 503 - Service Unavailable

VS Code service is not available or not enabled.

## cURL Example

```bash
curl --request GET \
  --url https://api.example.com/api/vscode/status \
  --header 'Content-Type: application/json'
```

## Example Response

```json
{
  "status": "ready",
  "running": true,
  "port": 8011,
  "message": "VS Code server is running and ready"
}
```

## Status Values

- **ready** - VS Code is fully initialized and accessible
- **starting** - VS Code service is starting up
- **stopping** - VS Code service is shutting down
- **error** - VS Code encountered an error during startup
- **disabled** - VS Code service is not enabled

## Prerequisites

- The agent server must be configured with `extra_ports=True` to enable VS Code
- VS Code service must be running in the container
- The host port+1 must be accessible from your machine

## Configuration

When using `DockerWorkspace` or similar workspace types, enable VS Code with:

```python
with DockerWorkspace(
    server_image="ghcr.io/openhands/agent-server:latest-python",
    host_port=8010,
    extra_ports=True,  # Enable VS Code access
) as workspace:
    # Check VS Code status
    response = httpx.get(f"{workspace.host}/api/vscode/status")
    status = response.json()
    print(f"VS Code status: {status['status']}")
```

## Related Endpoints

- [Get VSCode URL](get-vscode-url.md) - Get the VS Code Web interface URL
- [Get Desktop URL](../desktop/get-desktop-url.md) - Get the desktop/VNC interface URL
