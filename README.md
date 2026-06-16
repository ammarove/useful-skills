# Useful Skills

This repository collects reusable OpenCode skills that help agents perform common workflows consistently and avoid repeating project-specific setup from scratch.

## Current Skills

- [`dev-launcher`](dev-launcher/SKILL.md): creates a project-specific `dev` bash script that opens or attaches to a named tmux workspace with one window per selected tool or service.

## `dev-launcher`

The `dev-launcher` skill is for requests such as creating a `dev` script, setting up a tmux workspace, opening a project workspace, or setting up a development environment.

It scans a target project for signals such as Docker Compose files, app manifests, database configuration, Postman collections, and runtime tooling. Then it asks which tmux windows to include and in what order before generating the script.

The generated `dev` script should live at the workspace root, be executable with `chmod +x dev`, and be safe to rerun by attaching to the existing tmux session when one already exists.

## Prerequisites

Required:

- `tmux`, because the launcher creates and attaches to tmux sessions.

Optional, depending on which windows the user chooses:

- `tdl`, for a `work` window. The skill runs it as `bash -i -c 'tdl c'` because it is an omarchy shell function.
- `hermes`, for a Hermes TUI window.
- `posting`, for opening Postman collections from the terminal.
- `lazysql`, for database windows.
- Project runtime tools such as `npm`, Gradle, Cargo, Python, or Django commands when an app window is included.

## Skill Authoring Notes

Each skill is stored in its own directory with a `SKILL.md` file. Keep skill instructions compact, task-oriented, and focused on behavior future agents would otherwise get wrong.
