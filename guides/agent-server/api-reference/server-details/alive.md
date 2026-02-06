<!-- Source: https://docs.openhands.dev/sdk/guides/agent-server/api-reference/server-details/alive -->

# Alive

Check if the agent server is running and responding.

## Endpoint

```
GET /alive
```

## Query Parameters

None required for this endpoint.

## Response

### 200 - OK

The server is alive and responding.

**Response Schema:**

```json
{
  "status": "alive"
}
```

**Response Fields:**

- **status** (string) - Status indicator: "alive"

### 503 - Service Unavailable

The server is not responding or is temporarily unavailable.

## cURL Example

```bash
curl --request GET \
  --url https://api.example.com/alive \
  --header 'Content-Type: application/json'
```

## Example Response

```json
{
  "status": "alive"
}
```

## Usage Notes

- This is a lightweight endpoint for simple connectivity checks
- Returns immediately without performing system checks
- Useful for health monitoring and load balancer heartbeats
- No authentication typically required for this endpoint

## Common Use Cases

1. **Startup verification** - Check if server has started
2. **Connection testing** - Verify network connectivity
3. **Load balancer health checks** - Monitor server availability
4. **Liveness probes** - Kubernetes/container orchestration checks

## Example Usage

```python
import httpx

async def check_server_alive():
    try:
        response = httpx.get(f"{server_url}/alive", timeout=2.0)
        if response.status_code == 200:
            print("Server is alive")
            return True
    except httpx.RequestError:
        print("Server is not responding")
        return False
```

## Related Endpoints

- [Health](health.md) - Detailed health status
- [Get Server Info](get-server-info.md) - Detailed server information
