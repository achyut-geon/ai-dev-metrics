# ai-dev-metrics

Central repository for collecting AI-assisted developer metrics across projects. Pre-commit hook data is pushed here automatically by each developer's local hook — no manual uploads needed.

## Repository structure

```
<PROJECT>/
  pre-commit-reviews/
    YYYY-MM/
      reviews.jsonl     ← one JSON line per commit attempt
```

Each top-level folder represents a project or team. Current projects:

| Folder | Description |
|--------|-------------|
| `SVP/` | SVP project — Claude Code pre-commit review metrics |

## How metrics get here

A Claude Code pre-commit hook runs on each developer's machine at commit time. It:

1. Sends the staged diff to Claude for review
2. Prints the review result in the terminal
3. Blocks the commit if Claude flags a critical issue (`BLOCK:` prefix)
4. Appends a JSON entry to `~/.claude/metrics/pre-commit-reviews.jsonl` locally
5. Pushes the same entry to this repo in the background (non-blocking)

The push uses a shared GitHub fine-grained PAT stored in the developer's system keyring (`secret-tool`). Developers never interact with this repo directly.

## JSON entry format

Each line in a `reviews.jsonl` file is a JSON object:

```json
{
  "git_user":      "Jane Smith",
  "timestamp":     "2026-06-22T10:45:00Z",
  "repo":          "my-service",
  "branch":        "feature/auth-refactor",
  "blocked":       false,
  "block_comment": "",
  "review_excerpt": "No critical issues found. Minor: variable `x` could be renamed for clarity."
}
```

| Field | Description |
|-------|-------------|
| `git_user` | Developer name from `git config user.name` |
| `timestamp` | UTC commit time (ISO 8601) |
| `repo` | Repository where the commit was attempted |
| `branch` | Branch name at commit time |
| `blocked` | `true` if Claude blocked the commit |
| `block_comment` | Claude's reason if blocked, empty otherwise |
| `review_excerpt` | First 500 characters of Claude's review |

## Adding a new project

1. Copy `install-claude-hook.sh` and change the top-level folder name in the `api_path` line:
   ```python
   api_path = 'NEW_PROJECT/pre-commit-reviews/{}/reviews.jsonl'.format(month)
   ```
2. Replace `GH_METRICS_PAT` with a valid PAT (Contents: Read & Write on this repo).
3. Distribute the updated script to the project's developers.
4. Add the new folder to the table above.

## Installing the hook (developers)

```bash
# Run from inside any git repository
bash install-claude-hook.sh
```

The script installs Claude Code if not present, stores the PAT in the system keyring, and activates the pre-commit hook. No further configuration is needed.

To bypass the hook for a specific commit:

```bash
git commit --no-verify
```

## Requirements

- Linux / macOS
- `python3` on PATH
- `libsecret-tools` for keyring storage (`sudo apt install libsecret-tools` on Ubuntu)
- Claude Code CLI (auto-installed by the hook script if missing)
