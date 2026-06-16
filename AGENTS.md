# Repository Instructions

## Repo Shape
- This repo currently contains standalone OpenCode skills; there is no root package manifest, lockfile, CI, or test runner.
- The only tracked skill content is `dev-launcher/SKILL.md`; treat that file as the executable source of truth for behavior.

## Working On Skills
- Skill directories are self-contained and currently use a single `SKILL.md` with YAML frontmatter (`name`, `description`) followed by the instructions.
- Keep guidance compact and task-oriented; this repo is for reusable agent instructions, not application code.

## `dev-launcher` Specifics
- The skill must be used for requests to create a `dev` script, workspace launcher, tmux session setup, "open my workspace", or "set up dev environment".
- It should generate a project-specific `dev` bash script at the workspace root, not inside a subproject, and make it executable with `chmod +x dev`.
- Before generating a launcher, ask the user which tmux windows to include and in what order.
- `tdl c` must run as `bash -i -c 'tdl c'` because it is an omarchy shell function.
- Use absolute `$PROJECT_ROOT/...` paths for resources such as Posting/Postman collections so `./dev` works from any caller directory.
- Do not hardcode DB credentials; derive them from project config such as `application-local.properties` or Docker Compose env, or ask the user.

## Verification
- There are no repo-wide automated checks to run today; verify Markdown changes by rereading the changed file for accuracy and stale claims.
