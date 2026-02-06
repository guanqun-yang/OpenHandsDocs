<!-- Source: https://docs.openhands.dev/sdk/arch/workspace -->

# Workspace

The Workspace component abstracts execution environments for agent operations. It provides a unified interface for command execution and file operations across local processes, containers, and remote servers.

**Source:** openhands/sdk/workspace/

## Core Responsibilities

The Workspace system has four primary responsibilities:

1. **Execution Abstraction** - Unified interface for command execution across environments
2. **File Operations** - Upload, download, and manipulate files in workspace
3. **Resource Management** - Context manager protocol for setup/teardown
4. **Environment Isolation** - Separate agent execution from host system

## Architecture

The workspace system provides a pluggable interface for different execution environments.

## Key Components

| Component | Purpose | Design |
|-----------|---------|--------|
| BaseWorkspace | Abstract interface | Defines execution and file operation contracts |
| LocalWorkspace | Local execution | Subprocess-based command execution |
| RemoteWorkspace | Remote execution | HTTP API-based execution via agent-server |
| CommandResult | Execution output | Structured result with stdout, stderr, exit_code |
| FileOperationResult | File op outcome | Success status and metadata |

## Workspace Types

### Local vs Remote Execution

| Aspect | LocalWorkspace | RemoteWorkspace |
|--------|----------------|-----------------|
| Execution | Direct subprocess | HTTP → agent-server |
| Isolation | Process-level | Container/VM-level |
| Performance | Fast (no network) | Network overhead |
| Security | Host system access | Sandboxed |
| Use Case | Development, CLI | Production, web apps |

## Core Operations

### Command Execution

**Command Result Structure:**

| Field | Type | Description |
|-------|------|-------------|
| stdout | str | Standard output stream |
| stderr | str | Standard error stream |
| exit_code | int | Process exit code (0 = success) |
| timeout | bool | Whether command timed out |
| duration | float | Execution time in seconds |

### File Operations

| Operation | Local Implementation | Remote Implementation |
|-----------|---------------------|---------------------|
| Upload | shutil.copy() | POST /file/upload with multipart |
| Download | shutil.copy() | GET /file/download stream |
| Result | FileOperationResult | FileOperationResult |

## Resource Management

Workspaces use context manager for safe resource handling:

**Lifecycle Hooks:**

| Phase | LocalWorkspace | RemoteWorkspace |
|-------|----------------|-----------------|
| Enter | Create working directory | Connect to agent-server, verify |
| Use | Execute commands | Proxy commands via HTTP |
| Exit | No cleanup (persistent) | Disconnect, optionally stop container |

## Remote Workspace Extensions

The SDK provides remote workspace implementations in openhands-workspace package:

### Implementation Comparison

| Type | Setup | Isolation | Use Case |
|------|-------|-----------|----------|
| LocalWorkspace | Immediate | Process | Development, trusted code |
| DockerWorkspace | Spawn container | Container | Multi-user, untrusted code |
| RemoteAPIWorkspace | Connect to URL | Remote server | Distributed systems, cloud |

**Source:**
- DockerWorkspace: openhands-workspace/openhands/workspace/docker
- RemoteAPIWorkspace: openhands-workspace/openhands/workspace/remote_api

## Component Relationships

### How Workspace Integrates

**Relationship Characteristics:**
- **Conversation → Workspace:** Conversation factory uses workspace type to select LocalConversation or RemoteConversation
- **Workspace → Agent Server:** RemoteWorkspace delegates operations to agent-server API
- **Tools Independence:** Tools run in the same environment as workspace

## See Also

- **Conversation Architecture** - How workspace type determines conversation implementation
- **Agent Server** - Remote execution API
- **Tool System** - Tools that use workspace for execution
