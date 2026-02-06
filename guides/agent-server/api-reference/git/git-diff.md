<!-- Source: https://docs.openhands.dev/sdk/guides/agent-server/api-reference/git/git-diff -->

# Git Diff

Retrieve detailed git diff information for changed files.

## Endpoint

```
POST /api/git/diff
```

## Request Body

### Parameters

**file_path** (string, optional)
- Specific file to get diff for. If not provided, returns diff for all changed files.

**staged** (boolean, optional, default: false)
- If true, get diff of staged changes. If false, get diff of unstaged changes.

**context_lines** (integer, optional, default: 3)
- Number of context lines to show around changes

### Example Request

```json
{
  "file_path": "src/main.py",
  "staged": false,
  "context_lines": 3
}
```

## Response

### 200 - Successful Response

Returns detailed git diff for the specified files.

**Response Schema:**

```json
{
  "diffs": [
    {
      "file_path": "src/main.py",
      "status": "M",
      "additions": 5,
      "deletions": 2,
      "patch": "diff --git a/src/main.py b/src/main.py\n..."
    }
  ]
}
```

**Response Fields:**

- **diffs** (array) - List of diffs for changed files
  - **file_path** (string) - Path to the file
  - **status** (string) - Git status code (M, A, D, R, etc.)
  - **additions** (integer) - Number of lines added
  - **deletions** (integer) - Number of lines deleted
  - **patch** (string) - Unified diff format patch

### 422 - Unprocessable Entity

Validation error or file not found.

## cURL Example

```bash
curl --request POST \
  --url https://api.example.com/api/git/diff \
  --header 'Content-Type: application/json' \
  --data '{
    "file_path": "src/main.py",
    "staged": false,
    "context_lines": 3
  }'
```

## Example Response

```json
{
  "diffs": [
    {
      "file_path": "src/main.py",
      "status": "M",
      "additions": 5,
      "deletions": 2,
      "patch": "diff --git a/src/main.py b/src/main.py\nindex 1234567..abcdefg 100644\n--- a/src/main.py\n+++ b/src/main.py\n@@ -10,6 +10,8 @@\n def main():\n+    print('Hello')\n     pass\n"
    }
  ]
}
```

## Patch Format

The patch field contains unified diff format with:

- File header showing old/new versions
- Hunk headers showing line numbers
- Context lines (unchanged)
- Added lines (prefixed with +)
- Deleted lines (prefixed with -)

## Usage Notes

- Returns unified diff format (standard for most tools)
- Context lines help understand changes in context
- Works for both staged and unstaged changes
- Binary files show minimal diff information

## Common Use Cases

1. **Code review** - Inspect specific changes before committing
2. **Change analysis** - Understand what the agent modified
3. **Conflict detection** - Identify potential merge conflicts
4. **Audit trail** - Track exactly what changed in files

## Example Usage

```python
import httpx

async def get_file_diff(file_path):
    response = httpx.post(
        f"{server_url}/api/git/diff",
        json={
            "file_path": file_path,
            "staged": False,
            "context_lines": 5
        }
    )
    diff_data = response.json()

    for diff in diff_data['diffs']:
        print(f"File: {diff['file_path']}")
        print(f"Changes: +{diff['additions']} -{diff['deletions']}")
        print("\nDiff:")
        print(diff['patch'])
```

## Related Endpoints

- [Git Changes](git-changes.md) - Get summary of changed files
