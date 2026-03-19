---
mode: agent
description: Safety guardrails — warns before destructive operations (rm -rf, DROP TABLE, force-push, kubectl delete). Use when touching production or working in a shared environment. Say "be careful" to activate.
---

# /careful — Destructive Command Guardrails

Safety mode is now **active**. Before every terminal command, check it against the patterns below. If a destructive command is detected, warn the user and ask for confirmation before proceeding.

---

## Protected Operations

| Pattern | Example | Risk |
|---------|---------|------|
| `rm -rf` / `rm -r` / `rm --recursive` | `rm -rf /var/data` | Recursive delete |
| `DROP TABLE` / `DROP DATABASE` | `DROP TABLE users;` | Permanent data loss |
| `TRUNCATE` | `TRUNCATE orders;` | Permanent data loss |
| `DELETE FROM` without `WHERE` | `DELETE FROM logs;` | Permanent data loss |
| `git push --force` / `-f` | `git push -f origin main` | History rewrite |
| `git reset --hard` | `git reset --hard HEAD~3` | Uncommitted work loss |
| `git checkout .` / `git restore .` | `git checkout .` | Uncommitted work loss |
| `git clean -fd` / `git clean -fxd` | `git clean -fxd` | Untracked file deletion |
| `kubectl delete` | `kubectl delete pod --all` | Production impact |
| `kubectl exec` + destructive flag | `kubectl exec ... -- rm` | Container impact |
| `docker rm -f` / `docker system prune` | `docker system prune -a` | Image/container loss |
| `terraform destroy` | `terraform destroy -auto-approve` | Infrastructure deletion |
| Production database migrations | `rails db:drop`, `flask db downgrade` | Schema loss |
| `chmod -R 777` | `chmod -R 777 /etc` | Security risk |
| `> file` (redirect to overwrite) | `echo "" > important.txt` | File overwrite |
| `mv` to overwrite important files | `mv new.conf /etc/nginx.conf` | Config overwrite |

---

## Safe Exceptions (No Warning Needed)

These patterns are safe and allowed without warning:
- `rm -rf node_modules` — build artifact
- `rm -rf .next` / `dist` / `build` / `out` — build artifacts
- `rm -rf __pycache__` / `.cache` / `.turbo` / `coverage` / `.pytest_cache` — temp files
- `git reset --hard HEAD` (un-modified files only) — safe soft reset
- `git clean -fd` in a fresh clone with no important untracked files

---

## Warning Format

When a destructive command is detected:

```
⚠️  DESTRUCTIVE OPERATION DETECTED

Command: [the command]
Risk: [what could be lost or broken]

Before proceeding, confirm:
A) YES, proceed — I understand the risk
B) NO, cancel — let me reconsider
C) Show me a safe alternative

[If a safe alternative exists, propose it]
```

ALWAYS WAIT for user confirmation before executing any flagged command.

---

## Production Signals

Extra caution for commands that touch production:
- Commands with `prod`, `production`, `live` in the context
- Commands targeting production database connections
- Commands with `--force` or `-f` flags on any git operation
- `kubectl` commands targeting non-dev namespaces
- Terraform commands outside plan/local workspaces

For these: require explicit "YES, I understand this affects production" confirmation.

---

## Scope

This safety mode applies to:
- All `bash` / `shell` commands you would run
- All file deletion or overwrite operations
- All database operations that modify or delete data
- All git operations that rewrite history

It does NOT apply to:
- Read-only operations (grep, cat, ls, git diff, git log, git status)
- Safe writes (creating new files, appending to logs)
- Test commands (npm test, pytest, go test)

---

## Deactivation

This safety mode is active for this session. To deactivate, start a new chat session.

To override a specific warning: respond with "proceed" or "yes, I understand the risk."
