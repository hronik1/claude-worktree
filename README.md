# claude-worktree

A CLI tool to streamline Claude-assisted development workflows by automating git worktree creation and Claude session management.

## Why?

When working on multiple tasks, context switching between branches can be disruptive. This tool:

1. Creates isolated git worktrees for each task (Jira ticket or custom branch)
2. Launches Claude in plan mode with relevant context
3. Automatically transitions Jira tickets to "In Progress" (if in Todo)
4. Maintains resumable Claude sessions per worktree
5. Provides cleanup utilities for merged worktrees

Worktrees are created in a sibling directory (`../<repo>_worktrees/<ticket-or-branch>/`) to keep your main repo clean.

## Dependencies

- **git** - For worktree management
- **Claude CLI** - `claude` command must be available in PATH
- **Jira MCP server** (optional) - Required for `-j` flag to fetch ticket details and transition status

## Install

```bash
# Clone or copy the script
cp claude-worktree ~/.local/bin/

# Make executable
chmod +x ~/.local/bin/claude-worktree

# Ensure ~/.local/bin is in your PATH
# Add to ~/.bashrc or ~/.zshrc if needed:
export PATH="$HOME/.local/bin:$PATH"
```

## Usage

```bash
# Create worktree for a Jira ticket (fetches ticket, transitions to In Progress if Todo)
claude-worktree -j REPO-611

# Create worktree with custom branch name
claude-worktree -b feature/my-feature

# Add extra context to the initial prompt
claude-worktree -j REPO-611 -p "Focus on the API layer"

# List active worktrees
claude-worktree --list

# Clean up worktrees with merged branches
claude-worktree --cleanup --dry-run  # preview
claude-worktree --cleanup            # execute
```

### Flags

| Flag | Description |
|------|-------------|
| `-j <TICKET>` | Create worktree for Jira ticket. Branch: `<username>/<ticket-id>` |
| `-b <BRANCH>` | Create worktree with custom branch name |
| `-p <PROMPT>` | Append additional context to Claude's initial prompt |
| `--list` | List active worktrees |
| `--cleanup` | Remove worktrees for branches merged into main |
| `--help` | Show help message |

Note: Exactly one of `-j` or `-b` must be specified when creating a worktree.

### Resuming Sessions

If you run `claude-worktree -j <ticket>` for an existing worktree, it will `cd` to that worktree and run `claude --continue` to resume your previous session.

### Local Claude Settings

Claude stores project-specific settings in `.claude/settings.json` (tracked in git) and `.claude/settings.local.json` (gitignored, for personal overrides like MCP servers or API keys).

Since worktrees are separate directories, gitignored files don't automatically appear. This tool symlinks `.claude/settings.local.json` from your main repo to each worktree, so your local Claude configuration (MCP servers, etc.) works consistently across all worktrees.

## How It Works

1. Detects the default branch from `origin/HEAD` (falls back to main/prod/master)
2. Creates a worktree at `../<repo>_worktrees/<TICKET-ID>/` or `../<repo>_worktrees/<branch-name>/`
3. Creates a new branch: `<username>/<ticket-id>` or uses your specified branch name
4. Symlinks `.claude/settings.local.json` from the main repo (if present)
5. Launches Claude with a prompt to:
   - Fetch the Jira ticket (if `-j` used)
   - Transition to In Progress if currently in Todo
   - Enter plan mode via `EnterPlanMode` tool

## Worktree Structure

```
<repo-parent>/
├── <repo>/                          # main repo
└── <repo>_worktrees/                # worktrees managed by this tool
    ├── REPO-611/                   # from -j REPO-611
    ├── REPO-612/                   # from -j REPO-612
    └── feature-my-feature/          # from -b feature/my-feature
```
