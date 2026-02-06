<!-- Source: https://docs.openhands.dev/sdk -->

# Software Agent SDK - OpenHands Docs

The OpenHands Software Agent SDK is a set of Python and REST APIs for building agents that work with code. You can use the OpenHands Software Agent SDK for:

- One-off tasks, like building a README for your repo
- Routine maintenance tasks, like updating dependencies
- Major tasks that involve multiple agents, like refactors and rewrites

You can even use the SDK to build new developer experiences—it's the engine behind the OpenHands CLI and OpenHands Cloud. Get started with some examples or keep reading to learn more.

## Features

### Single Python API

A unified Python API that enables you to run agents locally or in the cloud, define custom agent behaviors, and create custom tools.

### Pre-defined Tools

Ready-to-use tools for executing Bash commands, editing files, browsing the web, integrating with MCP, and more.

### REST-based Agent Server

A production-ready server that runs agents anywhere, including Docker and Kubernetes, while connecting seamlessly to the Python API.

## Why OpenHands Software Agent SDK?

### Emphasis on coding

While other agent SDKs (e.g. LangChain) are focused on more general use cases, like delivering chat-based support or automating back-office tasks, OpenHands is purpose-built for software engineering. While some folks do use OpenHands to solve more general tasks (code is a powerful tool!), most of us use OpenHands to work with code.

### State-of-the-Art Performance

OpenHands is a top performer across a wide variety of benchmarks, including SWE-bench, SWT-bench, and multi-SWE-bench. The SDK includes a number of state-of-the-art agentic features developed by our research team, including:

- Task planning and decomposition
- Automatic context compression
- Security analysis
- Strong agent-computer interfaces

OpenHands has attracted researchers from a wide variety of academic institutions, and is becoming the preferred harness for evaluating LLMs on coding tasks.

### Free and Open Source

OpenHands is also the leading open source framework for coding agents. It's MIT-licensed, and can work with any LLM—including big proprietary LLMs like Claude and OpenAI, as well as open source LLMs like Qwen and Devstral. Other SDKs (e.g. Claude Code) are proprietary and lock you into a particular model. Given how quickly models are evolving, it's best to stay model-agnostic!

## Get Started

### Getting Started Guide

Install the SDK, run your first agent, and explore the guides.

### Learn the SDK

**Core Concepts** - Understand the SDK's architecture: agents, tools, workspaces, and more.

**API Reference** - Explore the complete SDK API and source code.

## Build with Examples

### Standalone SDK

Build local agents with custom tools and capabilities.

### Remote Execution

Run agents on remote servers with Docker sandboxing.

### GitHub Workflows

Automate repository tasks with agent-powered workflows.

## Community

### Join Slack

Connect with the OpenHands community on Slack.

### GitHub Repository

Contribute to the SDK or report issues on GitHub.
