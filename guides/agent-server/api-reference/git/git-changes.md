<!-- Source: https://docs.openhands.dev/sdk/guides/agent-server/api-reference/git/git-changes -->

# Git Changes

Retrieve the current git changes in the workspace (staged and unstaged).

## Endpoint

```
GET /api/git/changes
```

## Query Parameters

**include_untracked** (boolean, optional, default: false)
- Whether to include untracked files in the response

**include_ignored** (boolean, optional, default: false)
- Whether to include ignored files in the response

## Response

### 200 - Successful Response

Returns git status information with staged and unstaged changes.

**Response Schema:**

```json
{
  "staged": [
    {
      "path": "src/main.py",
      "status": "M",
      "type": "file"
    }
  ],
  "unstaged": [
    {
      "path": "src/utils.py",
      "status": "M",
      "type": "file"
    },
    {
      "path": "docs/readme.md",
      "status": "??",
      "type": "file"
    }
  ],
  "untracked": [
    {
      "path": "build/",
      "status": "??",
      "type": "directory"
    }
  ]
}
```

**Response Fields:**

- **staged** (array) - Files with staged changes (added to git index)
  - **path** (string) - File path
  - **status** (string) - Git status code (M=modified, A=added, D=deleted, R=renamed, etc.)
  - **type** (string) - "file" or "directory"
- **unstaged** (array) - Files with unstaged changes in working directory
- **untracked** (array) - Untracked files (only if include_untracked=true)

## cURL Example

```bash
curl --request GET \
  --url 'https://api.example.com/api/git/changes?include_untracked=true' \
  --header 'Content-Type: application/json'
```

## Example Response

```json
{
  "staged": [
    {
      "path": "src/main.py",
      "status": "M",
      "type": "file"
    },
    {
      "path": "requirements.txt",
      "status": "A",
      "type": "file"
    }
  ],
  "unstaged": [
    {
      "path": "src/utils.py",
      "status": "M",
      "type": "file"
    }
  ],
  "untracked": []
}
```

## Git Status Codes

- **M** - Modified
- **A** - Added
- **D** - Deleted
- **R** - Renamed
- **C** - Copied
- **T** - Type changed
- **U** - Updated but unmerged
- **??** - Untracked

## Usage Notes

- Only shows files with actual changes
- Requires a git repository to be present
- Returns empty arrays if no changes exist
- Status codes follow standard git conventions

## Common Use Cases

1. **Change inspection** - View what files have changed
2. **Commit preparation** - Check what's staged before committing
3. **Workflow monitoring** - Track agent modifications to files
4. **Cleanup** - Identify untracked files to clean up

## Example Usage

```python
import httpx

async def get_git_status():
    response = httpx.get(
        f"{server_url}/api/git/changes",
        params={"include_untracked": True}
    )
    changes = response.json()

    print("Staged changes:")
    for file in changes['staged']:
        print(f"  {file['path']} ({file['status']})")

    print("\nUnstaged changes:")
    for file in changes['unstaged']:
        print(f"  {file['path']} ({file['status']})")
```

## Related Endpoints

- [Git Diff](git-diff.md) - Get detailed diff of changes
