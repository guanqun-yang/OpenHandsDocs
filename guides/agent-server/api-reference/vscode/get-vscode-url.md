<!-- Source: https://docs.openhands.dev/sdk/guides/agent-server/api-reference/vscode/get-vscode-url -->

# Get VSCode URL

Retrieve the authenticated URL to access the VS Code Web interface.

## Endpoint

```
GET /api/vscode/url
```

## Query Parameters

**workspace_dir** (string, optional)
- The workspace directory to open in VS Code (will be pre-opened in the editor)

## Response

### 200 - Successful Response

Returns the VS Code Web interface URL with authentication token.

**Response Schema:**

```json
{
  "url": "http://localhost:8011/?tkn=token123&folder=/home/user/workspace"
}
```

**Response Fields:**

- **url** (string) - The complete authenticated URL to access VS Code

### 503 - Service Unavailable

VS Code service is not available or not enabled.

## cURL Example

```bash
curl --request GET \
  --url 'https://api.example.com/api/vscode/url?workspace_dir=/home/user/workspace' \
  --header 'Content-Type: application/json'
```

## Example Response

```json
{
  "url": "http://localhost:8011/?tkn=abc123def456&folder=/home/user/workspace"
}
```

## URL Format

The returned URL has the following format:

```
http://localhost:{vscode_port}/?tkn={token}&folder={workspace_dir}
```

Where:

- `vscode_port` - Usually host_port + 1 (e.g., 8011 if host_port is 8010)
- `token` - Authentication token for secure access
- `workspace_dir` - The workspace directory to open (optional)

## Usage Example

```python
import httpx

# Get VS Code URL
response = httpx.get(
    f"{workspace.host}/api/vscode/url",
    params={"workspace_dir": "/home/user/project"}
)
vscode_url = response.json()["url"]

# Open in browser
print(f"Open VS Code at: {vscode_url}")
```

## Prerequisites

- The agent server must be configured with `extra_ports=True`
- VS Code service must be running in the container
- The host port+1 must be accessible from your machine

## Features

When you access VS Code via this URL, you get:

- Full VS Code editor interface
- File explorer and editing capabilities
- Integrated terminal
- Syntax highlighting for multiple languages
- Git integration (if git is available)
- Debugging tools
- Extensions (if installed in the image)

## Configuration

Enable VS Code access with:

```python
with DockerWorkspace(
    server_image="ghcr.io/openhands/agent-server:latest-python",
    host_port=8010,
    extra_ports=True,  # Enable VS Code access
) as workspace:
    # Get authenticated VS Code URL
    response = httpx.get(
        f"{workspace.host}/api/vscode/url",
        params={"workspace_dir": workspace.working_dir}
    )
    vscode_url = response.json()["url"]
```

## Related Endpoints

- [Get VSCode Status](get-vscode-status.md) - Check VS Code service status
- [Get Desktop URL](../desktop/get-desktop-url.md) - Get the desktop/VNC interface URL
