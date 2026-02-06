<!-- Source: https://docs.openhands.dev/sdk/guides/github-workflows/assign-reviews -->

# Assign Reviews

The reference workflow is available [here](https://github.com/OpenHands/agent-sdk/tree/main/examples/03_github_workflows/01_basic_action)!

Automate pull request triage by intelligently assigning reviewers based on git blame analysis, notifying reviewers of pending PRs, and prompting authors on stale pull requests. The agent performs three sequential checks: pinging reviewers on clean PRs awaiting review (3+ days), reminding authors on stale PRs (5+ days), and auto-assigning reviewers based on code ownership for unassigned PRs.

## How it works

It relies on the basic action workflow (`01_basic_action`) which provides a flexible template for running arbitrary agent tasks in GitHub Actions.

**Core Components:**

- `agent_script.py` - Python script that initializes the OpenHands agent with configurable LLM settings and executes tasks based on provided prompts
- `workflow.yml` - GitHub Actions workflow that sets up the environment, installs dependencies, and runs the agent

**Prompt Options:**

- `PROMPT_STRING` - Direct inline text for simple prompts (used in this example)
- `PROMPT_LOCATION` - URL or file path for external prompts

The workflow downloads the agent script, validates configuration, runs the task, and uploads execution logs as artifacts.

## Assign Reviews Use Case

This specific implementation uses the basic action template to handle three PR management scenarios:

### 1. Need Reviewer Action

- Identifies PRs waiting for review
- Notifies reviewers to take action

### 2. Need Author Action

- Finds stale PRs with no activity for 5+ days
- Prompts authors to update, request review, or close

### 3. Need Reviewers

- Detects non-draft PRs without assigned reviewers (created 1+ day ago, CI passing)
- Uses git blame analysis to identify relevant contributors
- Automatically assigns reviewers based on file ownership and contribution history
- Balances reviewer workload across team members

## Quick Start

1. **Copy workflow to your repository**

```bash
cp examples/03_github_workflows/01_basic_action/assign-reviews.yml .github/workflows/assign-reviews.yml
```

2. **Configure secrets in GitHub Settings**

Go to GitHub Settings → Secrets → Actions, and add `LLM_API_KEY` (get from https://docs.openhands.dev/openhands/usage/llms/openhands-llms).

3. **Configure GitHub Actions permissions**

Go to GitHub Settings → Actions → General → Workflow permissions and enable "Read and write permissions".

4. **(Optional) Customize the schedule in the workflow file**

The default is: Daily at 12 PM UTC.

## Features

- **Intelligent Assignment** - Uses git blame to identify relevant reviewers based on code ownership
- **Automated Notifications** - Sends contextual reminders to reviewers and authors
- **Workload Balancing** - Distributes review requests evenly across team members
- **Scheduled & Manual** - Runs daily automatically or on-demand via workflow dispatch

## Reference Workflow

This example is available on GitHub: `examples/03_github_workflows/01_basic_action/assign-reviews.yml`

```yaml
---
# To set this up:
# 1. Change the name below to something relevant to your task
# 2. Modify the "env" section below with your prompt
# 3. Add your LLM_API_KEY to the repository secrets
# 4. Commit this file to your repository
# 5. Trigger the workflow manually or set up a schedule

name: Assign Reviews

on:
  # Manual trigger
  workflow_dispatch:
  # Scheduled trigger (disabled by default, uncomment and customize as needed)
  schedule:
    # Run at 12 PM UTC every day
    - cron: 0 12 * * *

permissions:
  contents: write
  pull-requests: write
  issues: write

jobs:
  run-task:
    runs-on: blacksmith-4vcpu-ubuntu-2404
    env:
      AGENT_SCRIPT_URL: https://raw.githubusercontent.com/OpenHands/agent-sdk/main/examples/03_github_workflows/01_basic_action/agent_script.py
      PROMPT_LOCATION: ''
      PROMPT_STRING: >
        Use GITHUB_TOKEN and the github API to organize open pull requests and issues in the repo.
        ...
      LLM_MODEL: <YOUR_LLM_MODEL>
      LLM_BASE_URL: <YOUR_LLM_BASE_URL>
    steps:
      - name: Checkout repository
        uses: actions/checkout@v5
      - name: Set up Python
        uses: actions/setup-python@v6
        with:
          python-version: '3.13'
      - name: Install uv
        uses: astral-sh/setup-uv@v7
        with:
          enable-cache: true
      - name: Install OpenHands dependencies
        run: |
          uv pip install --system "openhands-sdk @ git+https://github.com/OpenHands/agent-sdk.git@main#subdirectory=openhands-sdk"
          uv pip install --system "openhands-tools @ git+https://github.com/OpenHands/agent-sdk.git@main#subdirectory=openhands-tools"
      - name: Check required configuration
        env:
          LLM_API_KEY: ${{ secrets.LLM_API_KEY }}
        run: |
          # Validates configuration...
      - name: Run task
        env:
          LLM_API_KEY: ${{ secrets.LLM_API_KEY }}
          PYTHONPATH: ''
        run: |
          # Downloads and runs the agent script...
      - name: Upload logs as artifact
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: openhands-task-logs
          path: |
            *.log
            output/
          retention-days: 7
```

## Related Files

- [Agent Script](https://github.com/OpenHands/agent-sdk/tree/main/examples/03_github_workflows/01_basic_action/agent_script.py)
- [Workflow File](https://github.com/OpenHands/agent-sdk/tree/main/examples/03_github_workflows/01_basic_action/workflow.yml)
- [Basic Action README](https://github.com/OpenHands/agent-sdk/tree/main/examples/03_github_workflows/01_basic_action)
