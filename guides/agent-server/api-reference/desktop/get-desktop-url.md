<!-- Source: https://docs.openhands.dev/sdk/guides/agent-server/api-reference/desktop/get-desktop-url -->

# Get Desktop URL

Retrieve the URL to access the desktop/VNC interface of the remote agent server.

## Endpoint

```
GET /api/desktop/url
```

## Query Parameters

None required for this endpoint.

## Response

### 200 - Successful Response

Returns the desktop/VNC URL for accessing the remote agent server's visual desktop.

**Response Schema:**

```json
{
  "url": "http://localhost:8012/vnc.html?autoconnect=1&resize=remote"
}
```

**Response Fields:**

- **url** (string) - The complete URL to access the desktop/VNC interface

## cURL Example

```bash
curl --request GET \
  --url https://api.example.com/api/desktop/url \
  --header 'Content-Type: application/json'
```

## Usage

The returned URL can be opened in a web browser to access:

- Visual desktop environment (when VNC is enabled)
- Graphical applications running in the agent server
- Browser visualization (if browser tools are enabled)
- Other GUI-based tools

### Example Response

```json
{
  "url": "http://localhost:8012/vnc.html?autoconnect=1&resize=remote"
}
```

## Prerequisites

- The agent server must be configured with `extra_ports=True` to enable VNC access
- The VNC service must be running in the container
- The host port+2 must be accessible from your machine

## Configuration

When using `DockerWorkspace` or similar workspace types, enable desktop access with:

```python
with DockerWorkspace(
    server_image="ghcr.io/openhands/agent-server:latest-python",
    host_port=8010,
    extra_ports=True,  # Enable desktop/VNC access
) as workspace:
    # Get desktop URL
    response = httpx.get(f"{workspace.host}/api/desktop/url")
    desktop_url = response.json()["url"]
    print(f"Open desktop at: {desktop_url}")
```

## Related Endpoints

- [Get VSCode URL](../vscode/get-vscode-url.md) - Get the VS Code Web interface URL
- [Get VSCode Status](../vscode/get-vscode-status.md) - Check VS Code service status
