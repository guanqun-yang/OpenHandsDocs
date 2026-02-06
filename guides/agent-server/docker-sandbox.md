<!-- Source: https://docs.openhands.dev/sdk/guides/agent-server/docker-sandbox -->

# Docker Sandbox

The docker sandboxed agent server demonstrates how to run agents in isolated Docker containers using `DockerWorkspace`. This provides complete isolation from the host system, making it ideal for production deployments, testing, and executing untrusted code safely.

Use `DockerWorkspace` with a pre-built agent server image for the fastest startup. When you need to build your own image from a base image, switch to `DockerDevWorkspace`.

The Docker sandbox image ships with features configured in the Dockerfile (e.g., secure defaults and services like VSCode and VNC exposed behind well-defined ports), which are not available in the local (non-Docker) agent server.

## 1) Basic Docker Sandbox

A ready-to-run example is available here!

### Key Concepts

#### DockerWorkspace Context Manager

The `DockerWorkspace` uses a context manager to automatically handle container lifecycle:

```python
with DockerWorkspace(
    # use pre-built image for faster startup (recommended)
    server_image="ghcr.io/openhands/agent-server:latest-python",
    host_port=8010,
    platform=detect_platform(),
) as workspace:
    # Container is running here
    # Work with the workspace
    pass
# Container is automatically stopped and cleaned up here
```

The workspace automatically:

- Pulls or builds the Docker image
- Starts the container with an agent server
- Waits for the server to be ready
- Cleans up the container when done

#### Platform Detection

The example includes platform detection to ensure the correct Docker image is built and used:

```python
def detect_platform():
    """Detects the correct Docker platform string."""
    machine = platform.machine().lower()
    if "arm" in machine or "aarch64" in machine:
        return "linux/arm64"
    return "linux/amd64"
```

This ensures compatibility across different CPU architectures (Intel/AMD vs ARM/Apple Silicon).

#### Testing the Workspace

Before creating a conversation, the example tests the workspace connection:

```python
result = workspace.execute_command(
    "echo 'Hello from sandboxed environment!' && pwd"
)
logger.info(
    f"Command '{result.command}' completed "
    f"with exit code {result.exit_code}"
)
logger.info(f"Output: {result.stdout}")
```

This verifies the workspace is properly initialized and can execute commands.

#### Automatic RemoteConversation

When you use a `DockerWorkspace`, the `Conversation` automatically becomes a `RemoteConversation`:

```python
conversation = Conversation(
    agent=agent,
    workspace=workspace,
    callbacks=[event_callback],
    visualize=True,
)

assert isinstance(conversation, RemoteConversation)
```

The SDK detects the remote workspace and uses WebSocket communication for real-time event streaming.

#### DockerWorkspace vs DockerDevWorkspace

Use `DockerWorkspace` when you can rely on the official pre-built images for the agent server. Switch to `DockerDevWorkspace` when you need to build or customize the image on-demand (slower startup, requires the SDK source tree and Docker build support).

```python
# ✅ Fast: Use pre-built image (recommended)
DockerWorkspace(
    server_image="ghcr.io/openhands/agent-server:latest-python",
    host_port=8010,
)

# 🛠️ Custom: Build on the fly (requires SDK tooling)
DockerDevWorkspace(
    base_image="nikolaik/python-nodejs:python3.12-nodejs22",
    host_port=8010,
    target="source",
)
```

### Ready-to-run Example: Docker Sandbox

This example is available on GitHub: `examples/02_remote_agent_server/02_convo_with_docker_sandboxed_server.py`

This example shows how to create a `DockerWorkspace` that automatically manages Docker containers for agent execution:

```python
import os
import platform
import time
from pydantic import SecretStr
from openhands.sdk import (
    LLM,
    Conversation,
    RemoteConversation,
    get_logger,
)
from openhands.tools.preset.default import get_default_agent
from openhands.workspace import DockerWorkspace

logger = get_logger(__name__)

# 1) Ensure we have LLM API key
api_key = os.getenv("LLM_API_KEY")
assert api_key is not None, "LLM_API_KEY environment variable is not set."

llm = LLM(
    usage_id="agent",
    model=os.getenv("LLM_MODEL", "anthropic/claude-sonnet-4-5-20250929"),
    base_url=os.getenv("LLM_BASE_URL"),
    api_key=SecretStr(api_key),
)

def detect_platform():
    """Detects the correct Docker platform string."""
    machine = platform.machine().lower()
    if "arm" in machine or "aarch64" in machine:
        return "linux/arm64"
    return "linux/amd64"

# 2) Create a Docker-based remote workspace that will set up and manage
# the Docker container automatically. Use `DockerWorkspace` with a pre-built
# image or `DockerDevWorkspace` to automatically build the image on-demand.
# with DockerDevWorkspace(
#     # dynamically build agent-server image
#     base_image="nikolaik/python-nodejs:python3.12-nodejs22",
#     host_port=8010,
#     platform=detect_platform(),
# ) as workspace:
with DockerWorkspace(
    # use pre-built image for faster startup
    server_image="ghcr.io/openhands/agent-server:latest-python",
    host_port=8010,
    platform=detect_platform(),
) as workspace:
    # 3) Create agent
    agent = get_default_agent(
        llm=llm,
        cli_mode=True,
    )

    # 4) Set up callback collection
    received_events: list = []
    last_event_time = {"ts": time.time()}

    def event_callback(event) -> None:
        event_type = type(event).__name__
        logger.info(f"🔔 Callback received event: {event_type}\n{event}")
        received_events.append(event)
        last_event_time["ts"] = time.time()

    # 5) Test the workspace with a simple command
    result = workspace.execute_command(
        "echo 'Hello from sandboxed environment!' && pwd"
    )
    logger.info(
        f"Command '{result.command}' completed with exit code {result.exit_code}"
    )
    logger.info(f"Output: {result.stdout}")

    conversation = Conversation(
        agent=agent,
        workspace=workspace,
        callbacks=[event_callback],
    )

    assert isinstance(conversation, RemoteConversation)

    try:
        logger.info(f"\n📋 Conversation ID: {conversation.state.id}")

        logger.info("📝 Sending first message...")
        conversation.send_message(
            "Read the current repo and write 3 facts about the project into FACTS.txt."
        )

        logger.info("🚀 Running conversation...")
        conversation.run()
        logger.info("✅ First task completed!")
        logger.info(f"Agent status: {conversation.state.execution_status}")

        # Wait for events to settle (no events for 2 seconds)
        logger.info("⏳ Waiting for events to stop...")
        while time.time() - last_event_time["ts"] < 2.0:
            time.sleep(0.1)
        logger.info("✅ Events have stopped")

        logger.info("🚀 Running conversation again...")
        conversation.send_message("Great! Now delete that file.")
        conversation.run()
        logger.info("✅ Second task completed!")

        cost = conversation.conversation_stats.get_combined_metrics().accumulated_cost
        print(f"EXAMPLE_COST: {cost}")

    finally:
        print("\n🧹 Cleaning up conversation...")
        conversation.close()
```

You can run the example code as-is. The model name should follow the LiteLLM convention: `provider/model_name` (e.g., `anthropic/claude-sonnet-4-5-20250929`, `openai/gpt-4o`). The `LLM_API_KEY` should be the API key for your chosen provider.

```bash
export LLM_API_KEY="your-api-key"
export LLM_MODEL="anthropic/claude-sonnet-4-5-20250929" # or openai/gpt-4o, etc.

cd software-agent-sdk
uv run python examples/02_remote_agent_server/02_convo_with_docker_sandboxed_server.py
```

ChatGPT Plus/Pro subscribers: You can use `LLM.subscription_login()` to authenticate with your ChatGPT account and access Codex models without consuming API credits. See the LLM Subscriptions guide for details.

## 2) VS Code in Docker Sandbox

A ready-to-run example is available here!

VS Code with Docker demonstrates how to enable VS Code Web integration in a Docker-sandboxed environment. This allows you to access a full VS Code editor running in the container, making it easy to inspect, edit, and manage files that the agent is working with.

### Key Concepts

#### VS Code-Enabled DockerWorkspace

The workspace is configured with extra ports for VS Code access:

```python
with DockerWorkspace(
    server_image="ghcr.io/openhands/agent-server:latest-python",
    host_port=18010,
    platform="linux/arm64",  # or "linux/amd64" depending on your architecture
    extra_ports=True,  # Expose extra ports for VSCode and VNC
) as workspace:
    """Extra ports allows you to access VSCode at localhost:18011"""
```

The `extra_ports=True` setting exposes:

- Port `host_port+1`: VS Code Web interface (host_port + 1)
- Port `host_port+2`: VNC viewer for visual access

If you need to customize the agent-server image, swap in `DockerDevWorkspace` with the same parameters and provide `base_image`/`target` to build on demand.

#### VS Code URL Generation

The example retrieves the VS Code URL with authentication token:

```python
# Get VSCode URL with token
vscode_port = (workspace.host_port or 8010) + 1
try:
    response = httpx.get(
        f"{workspace.host}/api/vscode/url",
        params={"workspace_dir": workspace.working_dir},
    )
    vscode_data = response.json()
    vscode_url = vscode_data.get("url", "").replace(
        "localhost:8001", f"localhost:{vscode_port}"
    )
except Exception:
    # Fallback if server route not available
    folder = (
        f"/{workspace.working_dir}"
        if not str(workspace.working_dir).startswith("/")
        else str(workspace.working_dir)
    )
    vscode_url = f"http://localhost:{vscode_port}/?folder={folder}"
```

This generates a properly authenticated URL with the workspace directory pre-opened.

#### VS Code URL Format

```
http://localhost:{vscode_port}/?tkn={token}&folder={workspace_dir}
```

where:

- `vscode_port`: Usually host_port + 1 (e.g., 8011)
- `token`: Authentication token for security
- `workspace_dir`: Workspace directory to open

### Ready-to-run Example: VS Code

This example is available on GitHub: `examples/02_remote_agent_server/05_vscode_with_docker_sandboxed_server.py`

This example shows how to configure DockerWorkspace with VS Code capabilities:

```python
import os
import time
import httpx
from pydantic import SecretStr
from openhands.sdk import LLM, Conversation, get_logger
from openhands.sdk.conversation.impl.remote_conversation import RemoteConversation
from openhands.tools.preset.default import get_default_agent
from openhands.workspace import DockerWorkspace

logger = get_logger(__name__)

api_key = os.getenv("LLM_API_KEY")
assert api_key is not None, "LLM_API_KEY environment variable is not set."

llm = LLM(
    usage_id="agent",
    model=os.getenv("LLM_MODEL", "anthropic/claude-sonnet-4-5-20250929"),
    base_url=os.getenv("LLM_BASE_URL"),
    api_key=SecretStr(api_key),
)

# Create a Docker-based remote workspace with extra ports for VSCode access
def detect_platform():
    """Detects the correct Docker platform string."""
    import platform
    machine = platform.machine().lower()
    if "arm" in machine or "aarch64" in machine:
        return "linux/arm64"
    return "linux/amd64"

with DockerWorkspace(
    server_image="ghcr.io/openhands/agent-server:latest-python",
    host_port=18010,
    platform=detect_platform(),
    extra_ports=True,  # Expose extra ports for VSCode and VNC
) as workspace:
    """Extra ports allows you to access VSCode at localhost:18011"""

    # Create agent
    agent = get_default_agent(
        llm=llm,
        cli_mode=True,
    )

    # Set up callback collection
    received_events: list = []
    last_event_time = {"ts": time.time()}

    def event_callback(event) -> None:
        event_type = type(event).__name__
        logger.info(f"🔔 Callback received event: {event_type}\n{event}")
        received_events.append(event)
        last_event_time["ts"] = time.time()

    # Create RemoteConversation using the workspace
    conversation = Conversation(
        agent=agent,
        workspace=workspace,
        callbacks=[event_callback],
    )

    assert isinstance(conversation, RemoteConversation)

    logger.info(f"\n📋 Conversation ID: {conversation.state.id}")
    logger.info("📝 Sending first message...")
    conversation.send_message("Create a simple Python script that prints Hello World")
    conversation.run()

    # Get VSCode URL with token
    vscode_port = (workspace.host_port or 8010) + 1
    try:
        response = httpx.get(
            f"{workspace.host}/api/vscode/url",
            params={"workspace_dir": workspace.working_dir},
        )
        vscode_data = response.json()
        vscode_url = vscode_data.get("url", "").replace(
            "localhost:8001", f"localhost:{vscode_port}"
        )
    except Exception:
        # Fallback if server route not available
        folder = (
            f"/{workspace.working_dir}"
            if not str(workspace.working_dir).startswith("/")
            else str(workspace.working_dir)
        )
        vscode_url = f"http://localhost:{vscode_port}/?folder={folder}"
```

## 3) Browser in Docker Sandbox

A ready-to-run example is available here!

Browser with Docker demonstrates how to enable browser automation capabilities in a Docker-sandboxed environment. This allows agents to browse websites, interact with web content, and perform web automation tasks while maintaining complete isolation from your host system.

### Key Concepts

#### Browser-Enabled DockerWorkspace

The workspace is configured with extra ports for browser access:

```python
with DockerWorkspace(
    server_image="ghcr.io/openhands/agent-server:latest-python",
    host_port=8010,
    platform=detect_platform(),
    extra_ports=True,  # Expose extra ports for VSCode and VNC
) as workspace:
    """Extra ports allows you to check localhost:8012 for VNC"""
```

The `extra_ports=True` setting exposes additional ports for:

- Port `host_port+1`: VS Code Web interface
- Port `host_port+2`: VNC viewer for browser visualization

If you need to pre-build a custom browser image, replace `DockerWorkspace` with `DockerDevWorkspace` and provide `base_image`/`target` to build before launch.

#### Enabling Browser Tools

Browser tools are enabled by setting `cli_mode=False`:

```python
# Create agent with browser tools enabled
agent = get_default_agent(
    llm=llm,
    cli_mode=False,  # CLI mode = False will enable browser tools
)
```

When `cli_mode=False`, the agent gains access to browser automation tools for web interaction. When VNC is available and `extra_ports=True`, the browser will be opened in the VNC desktop to visualize agent's work. You can watch the browser in real-time via VNC.

#### VNC Access

The VNC interface provides real-time visual access to the browser:

```
http://localhost:8012/vnc.html?autoconnect=1&resize=remote
```

- `autoconnect=1`: Automatically connect to VNC server
- `resize=remote`: Automatically adjust resolution

### Ready-to-run Example: Browser

This example is available on GitHub: `examples/02_remote_agent_server/03_browser_use_with_docker_sandboxed_server.py`

This example shows how to configure DockerWorkspace with browser capabilities and VNC access:

```python
import os
import platform
import time
from pydantic import SecretStr
from openhands.sdk import LLM, Conversation, get_logger
from openhands.sdk.conversation.impl.remote_conversation import RemoteConversation
from openhands.tools.preset.default import get_default_agent
from openhands.workspace import DockerWorkspace

logger = get_logger(__name__)

api_key = os.getenv("LLM_API_KEY")
assert api_key is not None, "LLM_API_KEY environment variable is not set."

llm = LLM(
    usage_id="agent",
    model=os.getenv("LLM_MODEL", "anthropic/claude-sonnet-4-5-20250929"),
    base_url=os.getenv("LLM_BASE_URL"),
    api_key=SecretStr(api_key),
)

def detect_platform():
    """Detects the correct Docker platform string."""
    machine = platform.machine().lower()
    if "arm" in machine or "aarch64" in machine:
        return "linux/arm64"
    return "linux/amd64"

# Create a Docker-based remote workspace with extra ports for browser access.
# Use `DockerWorkspace` with a pre-built image or `DockerDevWorkspace` to
# automatically build the image on-demand.
with DockerWorkspace(
    server_image="ghcr.io/openhands/agent-server:latest-python",
    host_port=8011,
    platform=detect_platform(),
    extra_ports=True,  # Expose extra ports for VSCode and VNC
) as workspace:
    """Extra ports allows you to check localhost:8012 for VNC"""

    # Create agent with browser tools enabled
    agent = get_default_agent(
        llm=llm,
        cli_mode=False,  # CLI mode = False will enable browser tools
    )

    # Set up callback collection
    received_events: list = []
    last_event_time = {"ts": time.time()}

    def event_callback(event) -> None:
        event_type = type(event).__name__
        logger.info(f"🔔 Callback received event: {event_type}\n{event}")
        received_events.append(event)
        last_event_time["ts"] = time.time()

    # Create RemoteConversation using the workspace
    conversation = Conversation(
        agent=agent,
        workspace=workspace,
        callbacks=[event_callback],
    )

    assert isinstance(conversation, RemoteConversation)

    logger.info(f"\n📋 Conversation ID: {conversation.state.id}")
    logger.info("📝 Sending first message...")
    conversation.send_message(
        "Could you go to https://openhands.dev/ blog page and summarize main "
        "points of the latest blog?"
    )
    conversation.run()

    cost = conversation.conversation_stats.get_combined_metrics().accumulated_cost
    print(f"EXAMPLE_COST: {cost}")

    if os.getenv("CI"):
        logger.info(
            "CI environment detected; skipping interactive prompt and closing workspace."
        )
    else:
        # Wait for user confirm to exit when running locally
        y = None
        while y != "y":
            y = input(
                "Because you've enabled extra_ports=True in DockerDevWorkspace, "
                "you can open a browser tab to see the *actual* browser OpenHands "
                "is interacting with via VNC.\n\n"
                "Link: http://localhost:8012/vnc.html?autoconnect=1&resize=remote\n\n"
                "Press 'y' and Enter to exit and terminate the workspace.\n"
                ">> "
            )
```

## Next Steps

- [Local Agent Server](local-server.md)
- [Agent Server Overview](overview.md) - Architecture and implementation details
- [API Sandboxed Server](api-sandbox.md) - Connect to hosted API service
- Agent Server Package Architecture - Remote execution architecture
