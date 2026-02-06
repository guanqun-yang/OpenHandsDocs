<!-- Source: https://docs.openhands.dev/sdk/guides/github-workflows/todo-management -->

# TODO Management

The reference workflow is available [here](https://github.com/OpenHands/agent-sdk/tree/main/examples/03_github_workflows/03_todo_management)!

Scan your codebase for TODO comments and let the OpenHands Agent implement them, creating a pull request for each TODO and picking relevant reviewers based on code changes and file ownership.

## Quick Start

1. **Copy workflow to your repository**

```bash
cp examples/03_github_workflows/03_todo_management/workflow.yml .github/workflows/todo-management.yml
```

2. **Configure secrets in GitHub Settings → Secrets**

Go to GitHub Settings → Secrets and add `LLM_API_KEY` (get from https://docs.openhands.dev/openhands/usage/llms/openhands-llms).

3. **Configure GitHub Actions permissions**

Go to Settings → Actions → General → Workflow permissions and enable:
- Read and write permissions
- Allow GitHub Actions to create and approve pull requests

4. **Add TODO comments to your code**

Trigger the agent by adding TODO comments into your code.

Example:
```python
# TODO(openhands): Add input validation for user email
```

The workflow is configurable and any identifier can be used in place of `TODO(openhands)`.

## Features

- **Scanning** - Finds matching TODO comments with configurable identifiers and extracts the TODO description.
- **Implementation** - Sends the TODO description to the OpenHands Agent that automatically implements it.
- **PR Management** - Creates feature branches, pull requests and picks most relevant reviewers.

## Best Practices

- **Start Small** - Begin with `MAX_TODOS: 1` to test the workflow
- **Clear Descriptions** - Write descriptive TODO comments
- **Review PRs** - Always review the generated PRs before merging

## Reference Workflow

This example is available on GitHub: `examples/03_github_workflows/03_todo_management/`

```yaml
---
# Automated TODO Management Workflow
# Make sure to replace <YOUR_LLM_MODEL> and <YOUR_LLM_BASE_URL> with
# appropriate values for your LLM setup.

name: Automated TODO Management

on:
  workflow_dispatch:
    inputs:
      max_todos:
        description: Maximum number of TODOs to process in this run
        required: false
        default: '3'
        type: string
      todo_identifier:
        description: TODO identifier to search for (e.g., TODO(openhands))
        required: false
        default: TODO(openhands)
        type: string
  pull_request:
    types: [labeled]

permissions:
  contents: write
  pull-requests: write
  issues: write

jobs:
  scan-todos:
    runs-on: ubuntu-latest
    if: >
      github.event_name == 'workflow_dispatch' ||
      (github.event_name == 'pull_request' &&
       github.event.label.name == 'automatic-todo')
    outputs:
      todos: ${{ steps.scan.outputs.todos }}
      todo-count: ${{ steps.scan.outputs.todo-count }}
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.13'
      - name: Copy TODO scanner
        run: |
          cp examples/03_github_workflows/03_todo_management/scanner.py /tmp/scanner.py
          chmod +x /tmp/scanner.py
      - name: Scan for TODOs
        id: scan
        run: |
          # Scans codebase for TODO comments, limits results, sets outputs...

  process-todos:
    needs: scan-todos
    if: needs.scan-todos.outputs.todo-count > 0
    runs-on: ubuntu-latest
    strategy:
      matrix:
        todo: ${{ fromJson(needs.scan-todos.outputs.todos) }}
      max-parallel: 1
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
          token: ${{ secrets.GITHUB_TOKEN }}
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.13'
      - name: Install uv
        uses: astral-sh/setup-uv@v6
        with:
          enable-cache: true
      - name: Install OpenHands dependencies
        run: |
          uv pip install --system "openhands-sdk @ git+https://github.com/OpenHands/agent-sdk.git@main#subdirectory=openhands-sdk"
          uv pip install --system "openhands-tools @ git+https://github.com/OpenHands/agent-sdk.git@main#subdirectory=openhands-tools"
      - name: Process TODO
        env:
          LLM_MODEL: <YOUR_LLM_MODEL>
          LLM_BASE_URL: <YOUR_LLM_BASE_URL>
          LLM_API_KEY: ${{ secrets.LLM_API_KEY }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          # Creates branch, runs agent, commits changes, creates PR...

  summary:
    needs: [scan-todos, process-todos]
    if: always()
    runs-on: ubuntu-latest
    steps:
      - name: Generate Summary
        run: |
          # Generates workflow summary...
```

## Related Documentation

- [Agent Script](https://github.com/OpenHands/agent-sdk/tree/main/examples/03_github_workflows/03_todo_management/agent_script.py)
- [Scanner Script](https://github.com/OpenHands/agent-sdk/tree/main/examples/03_github_workflows/03_todo_management/scanner.py)
- [Workflow File](https://github.com/OpenHands/agent-sdk/tree/main/examples/03_github_workflows/03_todo_management/workflow.yml)
- [Prompt Template](https://github.com/OpenHands/agent-sdk/tree/main/examples/03_github_workflows/03_todo_management/prompt.py)
