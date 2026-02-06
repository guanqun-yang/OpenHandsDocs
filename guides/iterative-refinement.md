<!-- Source: https://docs.openhands.dev/sdk/guides/iterative-refinement -->

# Iterative Refinement

The ready-to-run example is available here!

## Overview

Iterative refinement is a powerful pattern where multiple agents work together in a feedback loop:

- A refactoring agent performs the main task (e.g., code conversion)
- A critique agent evaluates the quality and provides detailed feedback
- If quality is below threshold, the refactoring agent tries again with the feedback

This pattern is useful for:

- Code refactoring and modernization (e.g., COBOL to Java)
- Document translation and localization
- Content generation with quality requirements
- Any task requiring iterative improvement

## How It Works

### The Iteration Loop

The core workflow runs in a loop until quality threshold is met:

```python
QUALITY_THRESHOLD = 90.0
MAX_ITERATIONS = 5

while current_score < QUALITY_THRESHOLD and iteration < MAX_ITERATIONS:
    # Phase 1: Refactoring agent converts COBOL to Java
    refactoring_agent = get_default_agent(llm=llm, cli_mode=True)
    refactoring_conversation = Conversation(
        agent=refactoring_agent,
        workspace=str(workspace_dir)
    )
    refactoring_conversation.send_message(refactoring_prompt)
    refactoring_conversation.run()

    # Phase 2: Critique agent evaluates the conversion
    critique_agent = get_default_agent(llm=llm, cli_mode=True)
    critique_conversation = Conversation(
        agent=critique_agent,
        workspace=str(workspace_dir)
    )
    critique_conversation.send_message(critique_prompt)
    critique_conversation.run()

    # Parse score and decide whether to continue
    current_score = parse_critique_score(critique_file)
    iteration += 1
```

### Critique Scoring

The critique agent evaluates each file on four dimensions (0-25 pts each):

- **Correctness**: Does the Java code preserve the original business logic?
- **Code Quality**: Is the code clean and following Java conventions?
- **Completeness**: Are all COBOL features properly converted?
- **Best Practices**: Does it use proper OOP, error handling, and documentation?

### Feedback Loop

When the score is below threshold, the refactoring agent receives the critique file location:

```python
if critique_file and critique_file.exists():
    base_prompt += f"""
IMPORTANT: A previous refactoring attempt was evaluated and needs improvement.
Please review the critique at: {critique_file}
Address all issues mentioned in the critique to improve the conversion quality.
"""
```

## Customization

### Adjusting Thresholds

```python
QUALITY_THRESHOLD = 95.0  # Require higher quality
MAX_ITERATIONS = 10       # Allow more iterations
```

### Using Real COBOL Files

The example uses sample files, but you can use real files from the AWS CardDemo project.

## Ready-to-run Example

This example is available on GitHub: `examples/01_standalone_sdk/31_iterative_refinement.py`

```python
#!/usr/bin/env python3
"""
Iterative Refinement Example: COBOL to Java Refactoring

This example demonstrates an iterative refinement workflow where:
1. A refactoring agent converts COBOL files to Java files
2. A critique agent evaluates the quality of each conversion and provides scores
3. If the average score is below 90%, the process repeats with feedback

The workflow continues until the refactoring meets the quality threshold.

Source COBOL files can be obtained from:
https://github.com/aws-samples/aws-mainframe-modernization-carddemo/tree/main/app/cbl
"""

import os
import re
import tempfile
from pathlib import Path
from pydantic import SecretStr
from openhands.sdk import LLM, Conversation
from openhands.tools.preset.default import get_default_agent

QUALITY_THRESHOLD = float(os.getenv("QUALITY_THRESHOLD", "90.0"))
MAX_ITERATIONS = int(os.getenv("MAX_ITERATIONS", "5"))

def setup_workspace() -> tuple[Path, Path, Path]:
    """Create workspace directories for the refactoring workflow."""
    workspace_dir = Path(tempfile.mkdtemp())
    cobol_dir = workspace_dir / "cobol"
    java_dir = workspace_dir / "java"
    critique_dir = workspace_dir / "critiques"

    cobol_dir.mkdir(parents=True, exist_ok=True)
    java_dir.mkdir(parents=True, exist_ok=True)
    critique_dir.mkdir(parents=True, exist_ok=True)

    return workspace_dir, cobol_dir, java_dir

def create_sample_cobol_files(cobol_dir: Path) -> list[str]:
    """Create sample COBOL files for demonstration.

    In a real scenario, you would clone files from:
    https://github.com/aws-samples/aws-mainframe-modernization-carddemo/tree/main/app/cbl
    """
    sample_files = {
        "CBACT01C.cbl": """IDENTIFICATION DIVISION.
PROGRAM-ID. CBACT01C.
*****************************************************************
* Program: CBACT01C - Account Display Program
* Purpose: Display account information for a given account number
*****************************************************************
ENVIRONMENT DIVISION.
DATA DIVISION.
WORKING-STORAGE SECTION.
01 WS-ACCOUNT-ID PIC 9(11).
01 WS-ACCOUNT-STATUS PIC X(1).
01 WS-ACCOUNT-BALANCE PIC S9(13)V99.
01 WS-CUSTOMER-NAME PIC X(50).
01 WS-ERROR-MSG PIC X(80).
PROCEDURE DIVISION.
PERFORM 1000-INIT.
PERFORM 2000-PROCESS.
PERFORM 3000-TERMINATE.
STOP RUN.
1000-INIT.
INITIALIZE WS-ACCOUNT-ID
INITIALIZE WS-ACCOUNT-STATUS
INITIALIZE WS-ACCOUNT-BALANCE
INITIALIZE WS-CUSTOMER-NAME.
2000-PROCESS.
DISPLAY "ENTER ACCOUNT NUMBER: "
ACCEPT WS-ACCOUNT-ID
IF WS-ACCOUNT-ID = ZEROS
    MOVE "INVALID ACCOUNT NUMBER" TO WS-ERROR-MSG
    DISPLAY WS-ERROR-MSG
ELSE
    DISPLAY "ACCOUNT: " WS-ACCOUNT-ID
    DISPLAY "STATUS: " WS-ACCOUNT-STATUS
    DISPLAY "BALANCE: " WS-ACCOUNT-BALANCE
END-IF.
3000-TERMINATE.
DISPLAY "PROGRAM COMPLETE".""",
        "CBCUS01C.cbl": """IDENTIFICATION DIVISION.
PROGRAM-ID. CBCUS01C.
*****************************************************************
* Program: CBCUS01C - Customer Information Program
* Purpose: Manage customer data operations
*****************************************************************
ENVIRONMENT DIVISION.
DATA DIVISION.
WORKING-STORAGE SECTION.
01 WS-CUSTOMER-ID PIC 9(9).
01 WS-FIRST-NAME PIC X(25).
01 WS-LAST-NAME PIC X(25).
01 WS-ADDRESS PIC X(100).
01 WS-PHONE PIC X(15).
01 WS-EMAIL PIC X(50).
01 WS-OPERATION PIC X(1).
88 OP-ADD VALUE 'A'.
88 OP-UPDATE VALUE 'U'.
88 OP-DELETE VALUE 'D'.
88 OP-DISPLAY VALUE 'V'.
PROCEDURE DIVISION.
PERFORM 1000-MAIN-PROCESS.
STOP RUN.
1000-MAIN-PROCESS.
DISPLAY "CUSTOMER MANAGEMENT SYSTEM"
DISPLAY "A-ADD U-UPDATE D-DELETE V-VIEW"
ACCEPT WS-OPERATION
EVALUATE TRUE
    WHEN OP-ADD PERFORM 2000-ADD-CUSTOMER
    WHEN OP-UPDATE PERFORM 3000-UPDATE-CUSTOMER
    WHEN OP-DELETE PERFORM 4000-DELETE-CUSTOMER
    WHEN OP-DISPLAY PERFORM 5000-DISPLAY-CUSTOMER
    WHEN OTHER DISPLAY "INVALID OPERATION"
END-EVALUATE.""",
    }

    created_files = []
    for filename, content in sample_files.items():
        file_path = cobol_dir / filename
        file_path.write_text(content)
        created_files.append(filename)

    return created_files

def run_iterative_refinement() -> None:
    """Run the iterative refinement workflow."""
    # Setup
    api_key = os.getenv("LLM_API_KEY")
    assert api_key is not None, "LLM_API_KEY environment variable is not set."
    model = os.getenv("LLM_MODEL", "anthropic/claude-sonnet-4-5-20250929")
    base_url = os.getenv("LLM_BASE_URL")

    llm = LLM(
        model=model,
        base_url=base_url,
        api_key=SecretStr(api_key),
        usage_id="iterative_refinement",
    )

    workspace_dir, cobol_dir, java_dir = setup_workspace()
    critique_dir = workspace_dir / "critiques"

    print(f"Workspace: {workspace_dir}")
    print(f"COBOL Directory: {cobol_dir}")
    print(f"Java Directory: {java_dir}")
    print(f"Critique Directory: {critique_dir}")
    print()

    # Create sample COBOL files
    cobol_files = create_sample_cobol_files(cobol_dir)
    print(f"Created {len(cobol_files)} sample COBOL files:")
    for f in cobol_files:
        print(f" - {f}")
    print()

    critique_file = critique_dir / "critique_report.md"
    current_score = 0.0
    iteration = 0

    while current_score < QUALITY_THRESHOLD and iteration < MAX_ITERATIONS:
        iteration += 1
        print("=" * 80)
        print(f"ITERATION {iteration}")
        print("=" * 80)

        # Phase 1: Refactoring
        print("\n--- Phase 1: Refactoring Agent ---")
        refactoring_agent = get_default_agent(llm=llm, cli_mode=True)
        refactoring_conversation = Conversation(
            agent=refactoring_agent,
            workspace=str(workspace_dir),
        )

        refactoring_conversation.send_message("Convert COBOL to Java")
        refactoring_conversation.run()
        print("Refactoring phase complete.")

        # Phase 2: Critique
        print("\n--- Phase 2: Critique Agent ---")
        critique_agent = get_default_agent(llm=llm, cli_mode=True)
        critique_conversation = Conversation(
            agent=critique_agent,
            workspace=str(workspace_dir),
        )

        critique_conversation.send_message("Evaluate the Java conversion")
        critique_conversation.run()
        print("Critique phase complete.")

    # Final summary
    print("\n" + "=" * 80)
    print("ITERATIVE REFINEMENT COMPLETE")
    print("=" * 80)
    print(f"Total iterations: {iteration}")
    print(f"Workspace: {workspace_dir}")

if __name__ == "__main__":
    run_iterative_refinement()
```

### Running the Example

You can run the example code as-is. The model name should follow the LiteLLM convention: `provider/model_name` (e.g., `anthropic/claude-sonnet-4-5-20250929`, `openai/gpt-4o`).

The `LLM_API_KEY` should be the API key for your chosen provider.

ChatGPT Plus/Pro subscribers: You can use `LLM.subscription_login()` to authenticate with your ChatGPT account and access Codex models without consuming API credits. See the LLM Subscriptions guide for details.

## Next Steps

- Agent Delegation - Parallel task execution with sub-agents
- Custom Tools - Create specialized tools for your workflow
