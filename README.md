# Conductor Worktree Tracker 3D

A 3D visualization tool that monitors [Conductor](https://conductor.app)'s SQLite database and displays worktrees with PR hierarchy, CI/CD status, and commit history in an interactive Three.js interface.

## Features

- **3D Visualization**: Interactive Three.js scene with repos positioned in 3D space
- **PR Integration**: Shows PR titles, numbers, and stacked PR relationships
- **CI/CD Status**: Live status indicators for GitHub Actions
- **Commit History**: Shows commits between branches with author avatars
- **Session Status**: Shows which worktrees have active Claude sessions (working/idle)
- **Smart Branch Detection**: Detects actual git branches even when Conductor's database is stale

## Quick Start

```bash
# Start the server (serves 3D visualization at http://localhost:8765)
python3 tracker_server.py

# Use a different port
python3 tracker_server.py --port 9000
```

Then open http://localhost:8765 in your browser.

## Configuration

All settings can be configured via `config.json` in the same directory as `tracker_server.py`.

### config.json

```json
{
  "stale_minutes": 30,
  "fetch_prs": true,
  "since": "today",
  "pr_refresh_minutes": 15,
  "ci_cache_pending_seconds": 300,
  "ci_cache_stable_seconds": 900,
  "ci_cache_unknown_seconds": 600,
  "ci_max_requests_per_cycle": 2,
  "fetch_commits": true
}
```

### Configuration Variables

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `stale_minutes` | int | `30` | Minutes before a worktree is marked as stale. |
| `fetch_prs` | bool | `true` | Query GitHub for open PRs to determine hierarchy. |
| `since` | string | `"today"` | Activity lookback: `"today"` for since midnight, or hours (e.g., `"2"`). |
| `pr_refresh_minutes` | int | `15` | How often to refresh PR data from GitHub. |
| `ci_cache_pending_seconds` | int | `300` | Cache duration for pending CI status. |
| `ci_cache_stable_seconds` | int | `900` | Cache duration for pass/fail CI status. |
| `ci_max_requests_per_cycle` | int | `2` | Max CI API calls per refresh cycle. |
| `fetch_commits` | bool | `true` | Fetch commit history between branches. |

## API Endpoints

### GET /api/tracker

Returns JSON with:
- `trees`: Hierarchy of repos, PRs, and worktrees
- `session_statuses`: Current Claude session status per workspace
- `session_start`: When the tracker was started
- `worktree_count`: Number of tracked worktrees

Query parameters:
- `?refresh_ci=true`: Force refresh all CI statuses (bypasses cache)

## Visual Indicators

### Session Status Icons

| Icon | Status | Description |
|------|--------|-------------|
| Spinning | Working | Claude is actively processing |
| Static | Idle | Session is waiting for user input |
| Red | Error | Session encountered an error |

### CI/CD Status

| Status | Description |
|--------|-------------|
| Green | All checks passed |
| Red | One or more checks failed |
| Yellow | Checks are running |

## How It Works

1. **On startup**: Loads workspaces from Conductor's SQLite database, detects actual git branches, queries GitHub for open PRs
2. **Branch detection**: Runs `git branch --show-current` in each worktree to detect the real branch
3. **PR Hierarchy**: Uses GitHub PR base/head relationships to build parent-child tree
4. **CI Status**: Queries `gh pr checks` for each PR with rate-limited caching
5. **Commits**: Runs `git log base..head` to get commits between branches

## Dependencies

- Python 3.8+
- `gh` CLI (for GitHub PR queries and CI status): `brew install gh`

## Data Sources

- **Conductor DB**: `~/Library/Application Support/com.conductor.app/conductor.db`
- **GitHub PRs**: via `gh pr list` CLI command
- **CI Status**: via `gh pr checks` CLI command
- **Git branches**: via `git branch --show-current` in each worktree
- **Commits**: via `git log` in each repo
