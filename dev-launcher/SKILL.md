---
name: dev-launcher
description: Use when the user asks to create a dev script, workspace launcher, or tmux session setup for a project. Also use when user says "open my workspace", "set up dev environment", or wants a script that opens their project tools in tmux windows.
---

# dev-launcher

## Overview

Generate a project-specific `dev` bash script that creates (or attaches to) a named tmux session with one window per tool/service. The script is idempotent — re-running it attaches to the existing session.

## Process

### 1. Detect project context

Scan the target directory for signals:

| Signal | Suggests |
|--------|----------|
| `docker-compose.yml` / `compose.yaml` | Services (MySQL, Redis, Postgres, etc.) |
| `application*.properties` / `application*.yml` | DB URL, port, credentials |
| `gradlew` / `build.gradle` | Spring Boot app → `./gradlew bootRun` |
| `package.json` with `"start"` script | Node app → `npm start` |
| `Cargo.toml` | Rust app → `cargo run` |
| `pyproject.toml` / `manage.py` | Python/Django app |
| `postman/` directory | API collection → `posting` |
| `*.postman_collection.json` | API collection → `posting` |
| `.env` / `.env.local` | DB credentials override |

### 2. Suggest windows

Always offer these as the base set, then ask the user which to include and in what order:

| Window | Command | When to suggest |
|--------|---------|-----------------|
| `work` | `bash -i -c 'tdl c'` | Always on omarchy setups |
| `hermes` | `hermes --tui` | If `hermes` is installed |
| `app` | `./gradlew bootRun` / `npm start` / etc. | If app detected |
| `db` | `lazysql <url>` | If DB detected |
| `api` | `posting --collection <path>` | If postman collection found |
| `logs` | `tail -f <logfile>` | If log path known |

**Always ask the user** before generating — they may want fewer windows, different order, or custom commands.

### 3. Determine script location

- Place `dev` at the **workspace root** (the folder the user considers "the project"), not inside a subproject
- All windows open in that same root by default
- Paths to subproject resources (postman collections, gradlew) use `$PROJECT_ROOT/relative/path`

### 4. Generate the script

Use this template:

```bash
#!/usr/bin/env bash
set -euo pipefail

PROJECT_ROOT="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
SESSION="<project-name>"

if tmux has-session -t "$SESSION" 2>/dev/null; then
  tmux attach-session -t "$SESSION"
  exit 0
fi

# Window 1: work
tmux new-session -d -s "$SESSION" -c "$PROJECT_ROOT" -n work
tmux send-keys -t "$SESSION:work" "bash -i -c 'tdl c'" C-m

# Window 2: <name>
tmux new-window -t "$SESSION" -c "$PROJECT_ROOT" -n <name>
tmux send-keys -t "$SESSION:<name>" "<command>" C-m

# ... more windows ...

tmux select-window -t "$SESSION:work"
tmux attach-session -t "$SESSION"
```

### 5. Key rules

- `tdl c` needs `bash -i -c 'tdl c'` — it's an omarchy shell function, requires interactive shell
- DB URL for lazysql: `mysql://user:pass@host:port/` — get credentials from `application-local.properties` or docker-compose env
- `posting` collection path must be absolute using `$PROJECT_ROOT/...` if the script lives above the postman folder
- Make the script executable: `chmod +x dev`
- Session name = lowercase project folder name (no spaces)

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| `tdl c` without `bash -i` | Function not found. Use `bash -i -c 'tdl c'` |
| Relative path for posting collection | Breaks if user runs `./dev` from different dir. Use `$PROJECT_ROOT/...` |
| Script placed inside subproject | Should be at workspace root |
| Hardcoded DB credentials | Read from `application-local.properties` or ask user |
| Forgetting `chmod +x` | Script won't run |
