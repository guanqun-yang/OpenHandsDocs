<!-- Source: https://docs.openhands.dev/sdk/guides/agent-server/api-reference/server-details/health -->

# Health

Get detailed health status of the agent server and its services.

## Endpoint

```
GET /health
```

## Query Parameters

None required for this endpoint.

## Response

### 200 - OK

The server and its services are healthy.

**Response Schema:**

```json
{
  "status": "healthy",
  "services": {
    "bash": {
      "status": "healthy",
      "latency_ms": 5
    },
    "files": {
      "status": "healthy",
      "latency_ms": 3
    },
    "vscode": {
      "status": "healthy",
      "latency_ms": 10
    },
    "desktop": {
      "status": "healthy",
      "latency_ms": 8
    }
  },
  "uptime_seconds": 3600
}
```

**Response Fields:**

- **status** (string) - Overall health status: "healthy", "degraded", "unhealthy"
- **services** (object) - Health status of individual services
  - Service name (e.g., "bash", "files", "vscode", "desktop")
    - **status** (string) - Service status
    - **latency_ms** (integer) - Response latency in milliseconds
- **uptime_seconds** (integer) - Server uptime in seconds

### 503 - Service Unavailable

The server or its services are unhealthy.

## cURL Example

```bash
curl --request GET \
  --url https://api.example.com/health \
  --header 'Content-Type: application/json'
```

## Example Response

```json
{
  "status": "healthy",
  "services": {
    "bash": {
      "status": "healthy",
      "latency_ms": 4
    },
    "files": {
      "status": "healthy",
      "latency_ms": 2
    },
    "vscode": {
      "status": "healthy",
      "latency_ms": 12
    }
  },
  "uptime_seconds": 7200
}
```

## Health Status Values

- **healthy** - Service is fully operational
- **degraded** - Service is operational but experiencing issues
- **unhealthy** - Service is not operational

## Usage Notes

- This endpoint performs more comprehensive checks than `/alive`
- Includes latency measurements for performance monitoring
- Useful for detailed diagnostics and monitoring systems
- Provides service-specific status information

## Common Use Cases

1. **Readiness checks** - Verify all services are ready before use
2. **Detailed monitoring** - Track service performance over time
3. **Diagnostics** - Identify which services have issues
4. **SLA compliance** - Monitor uptime and availability

## Example Usage

```python
import httpx

async def check_server_health():
    response = httpx.get(f"{server_url}/health", timeout=5.0)
    health = response.json()

    if health["status"] == "healthy":
        print("Server is healthy")
        for service, info in health["services"].items():
            print(f"  {service}: {info['status']} ({info['latency_ms']}ms)")
    else:
        print(f"Server health: {health['status']}")
        return False

    return True
```

## Related Endpoints

- [Alive](alive.md) - Simple connectivity check
- [Get Server Info](get-server-info.md) - Detailed server information
