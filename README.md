# Omni Git Workflow Manager (`g`)

# About

**`g`** is an interactive Git workflow manager for the [Omni Shell Framework](../omni/README.md). It combines `git`, `fzf`, and `vimdiff` into a cohesive command-line interface with full tab completion for rapid staging, structured conventional commits, non-destructive history editing, branch operations, semantic version tagging, interactive merge conflict resolution, and remote management.

## Key Features

- **Rapid Shortcuts (`g .`, `g ..`, `g ,`)**: Stage, unstage, check status, and switch/create branches with single-character subcommands.
- **Structured Conventional Commits**: Configure global commit types (`feat`, `fix`, `ref`, `ci`, `conf`, `doc`, etc.) and scopes/contexts with automatic tab completion (`<type>(<context>): <message>`).
- **Non-Destructive History Editing (`g history`)**: Squash (`join`), reword (`rename`), or drop (`rm`) historical commits automatically, or open an isolated temporary branch (`g-edit-<hash>`) session to safely experiment before rebasing your branch.
- **Interactive Conflict Resolution (`g conflict`)**: Preview merge conflicts before merging (`git merge-tree`), inspect conflict blocks with line numbers, open 2-way diffs in `vimdiff`, or step through conflicts hunks interactively choosing ours (`+`), theirs (`-`), or combined resolutions (`+-` / `-+`).
- **Automated SemVer Tagging (`g tag`)**: Automatically calculate and increment `major`, `minor`, or `patch` versions (`vX.Y.Z`).

## Prerequisites

- `git`
- `fzf` (used for interactive file, branch, commit, tag, and remote pickers)
- `vimdiff` (optional, used by `g commit diff` and `g conflict diff`)

---

# Install

## Via the Official `omni` Provider

```bash
omni providers install omni
omni tools use g --provider omni
```

## From a Local Directory

```bash
omni tools use -p /path/to/omni/g
```

## Directly from GitHub

```bash
omni tools use -u https://github.com/omnishell/g.git
```

---

# Usage

## Quick Shortcuts & Top-Level Commands

| Command | Description |
| :--- | :--- |
| `g .` | Show latest commit (`git log -1 --oneline`) and working tree status (`git status`) |
| `g . <files...>` | Stage specified file(s) (`git add`) |
| `g . -a\|--all` | Stage all changes (`git add --all`) |
| `g ..` | Interactively select staged files via `fzf` to unstage |
| `g .. <files...>` | Unstage specified file(s) (`git reset HEAD --`) |
| `g .. -a\|--all` | Unstage all staged changes (`git reset`) |
| `g ,` | List all local and remote branches (`git branch -a`) |
| `g , <branch>` | Switch to `<branch>` if it exists, or create and switch to it (`git checkout -b <branch>`) |
| `g info` | Show latest commit and status |
| `g send [<ref>]` | Push `<ref>` (defaults to `HEAD`) to `origin` |
| `g git <args...>` | Pass arbitrary arguments directly to raw `git` |

---

## Staging & Working Tree (`g stage`)

```bash
# Show latest commit and working tree status
g stage info

# List currently staged files
g stage ls

# Interactively pick modified/untracked files via fzf to stage
g stage add

# Unstage specific files, or pick interactively via fzf if no files are given
g stage rm [<files...>]

# Discard working tree changes in specified file path(s)
g stage undo <file1> [<file2> ...]

# Hard-reset current branch back <N> commits (prompts for confirmation)
g stage undo <N>

# Remove untracked files and directories (`git clean -fd`)
g stage clean
```

---

## Commits & Diffs (`g commit`)

```bash
# Create a commit (prompts interactively if message is omitted)
g commit new [<message>]

# Create a conventional commit with type and optional context: "<type>(<ctx>): <message>"
g commit new [-c|--context <ctx>] <type>: [<message>]

# View colorized commit graph for the last N commits (default: 10)
g commit ls [<N>]

# View working tree diff, or open vimdiff difftool when arguments/paths are passed
g commit diff [<args...>]

# Manage global conventional commit types (defaults: feat, fix, ref, ci, conf, doc)
g commit type ls
g commit type new|set <type>
g commit type rm <type>

# Manage global conventional commit contexts/scopes
g commit context ls
g commit context new|set <context>
g commit context rm <context>
```

---

## History Rewriting (`g history`)

```bash
# Squash multiple commits into a single commit (select interactively with fzf if omitted)
g history join [<commit1> <commit2> ...] [-m|--message <new_message>]

# Rename/reword HEAD or any historical commit (accepts commit hash or steps back from HEAD)
g history rename [<commit_or_steps>] [-m|--message <new_message>]

# Drop one or more commits from history (accepts hashes, step counts, or interactive fzf selection)
g history rm [<commits_or_steps...>]
```

### Safe History Edit Sessions (`g history session`)

Start an isolated editing session on a temporary branch (`g-edit-<hash>`) reset to an earlier commit, make changes, and then cleanly rebase the original branch onto your edited history:

```bash
# Start a session at a target commit or N steps back (or pick via fzf)
g history session new [<commit_or_steps>]

# Inspect active session status (original branch, temp branch, target commit)
g history session ls

# Rebase the original branch onto the edited session HEAD (pass -d/--delete to remove temp branch)
g history session apply [-d|--delete]

# Abort the session, return to the original branch, and delete the temporary branch
g history session rm
```

---

## Branch Management (`g branch`)

```bash
# List all local and remote branches
g branch ls

# Create and switch to a new branch (or switch to existing branch)
g branch new <branch>
g branch <branch>

# Delete branch(es) (uses fzf multi-select if omitted)
g branch rm [<branch>]

# Push branch to origin (defaults to HEAD)
g branch send [<branch>]

# Pull with fast-forward from remote (defaults to origin)
g branch sync [<remote>]

# Rename a branch (select via fzf if omitted)
g branch rename [<old_branch>] [--name <new_branch>]

# Rebase onto one or more branches (or remote branches with -r/--remote)
g branch rebase [-r|--remote <remote>] [<branches...>]

# Fast-forward merge (`git merge --ff-only`) one or more branches
g branch ff [-r|--remote <remote>] [<branches...>]

# Merge one or more branches with an optional commit message
g branch merge [<branches...>] [-m|--message <message>]
```

---

## Semantic Version Tags (`g tag`)

```bash
# List tags sorted by version descending
g tag ls

# Create a new semantic version tag (auto-increments latest vX.Y.Z, starting at v0.1.0)
g tag new patch [-m|--message <msg>]
g tag new minor [-m|--message <msg>]
g tag new major [-m|--message <msg>]

# Create a custom tag
g tag new custom <tag_name> [-m|--message <msg>]

# Push the latest tag to origin
g tag send

# Edit an existing tag's name and/or annotation message
g tag edit [<tag>] [--name <new_name>] [--desc|-m <new_message>]

# Delete a tag locally (or pick via fzf)
g tag rm [<tag>]
```

---

## Conflict Detection & Resolution (`g conflict`)

```bash
# List files currently in a conflicted state
g conflict
g conflict HEAD

# Dry-run merge check: test if merging <branch> into HEAD would cause conflicts
g conflict <branch>

# Display conflict markers and blocks with line numbers across conflicted files
g conflict ls [<files...>]

# Interactively resolve conflicts hunk-by-hunk on a safe temporary branch:
#   [+] Keep current branch (ours)
#   [-] Keep incoming branch (theirs)
#   [+-] Keep ours followed by theirs
#   [-+] Keep theirs followed by ours
#   [Enter] Skip hunk   [q] Quit
g conflict resolve [<file>]

# Open ours (stage 2) and theirs (stage 3) side-by-side in vimdiff
g conflict diff <file>
```

---

## Remote Management (`g remote`)

```bash
# List remotes with URLs
g remote ls

# Add a new remote
g remote new [--name] <name> [--url] <url>

# Update a remote's name and/or URL
g remote update <name> [--name <new_name>] [--url <new_url>]

# Remove remote(s) by name, URL, or interactive fzf selection
g remote rm [<name_or_url...>]
```
