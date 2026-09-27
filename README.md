# claude-code-skills

Three focused Claude Code skills for safer automation and operational debugging.

## Included skills

| Skill | Purpose |
|---|---|
| `claude-p-agent-spawn-timeout` | Diagnose scheduled `claude -p` jobs that time out after spawning an internal agent |
| `git-prepush-secret-remote-check` | Check the Git remote, staged secrets, environment files, and basic PII before publishing |
| `hook-latency-wrap` | Instrument Claude Code hooks and summarize latency and failures |

## Why it exists

These are small runbooks for recurring failure modes that benefit from a repeatable checklist. They contain instructions only; they do not install hooks, alter Git remotes, or run commands automatically.

## Install

Copy one or all skill directories into your Claude Code skills directory:

```sh
mkdir -p "$HOME/.claude/skills"
cp -R skills/claude-p-agent-spawn-timeout "$HOME/.claude/skills/"
cp -R skills/git-prepush-secret-remote-check "$HOME/.claude/skills/"
cp -R skills/hook-latency-wrap "$HOME/.claude/skills/"
```

Restart Claude Code if the skills are not discovered in the current session.

## Usage

Ask Claude Code to use a skill by name when its situation applies, for example:

```text
Use git-prepush-secret-remote-check before I publish this repository.
```

## Configuration

No repository-wide environment variables are required. Commands and paths shown inside each skill are examples and should be adapted to the target machine.

## Requirements

- Claude Code with local skill discovery enabled
- macOS or Linux shell tools used by the selected runbook
- Git for the pre-push runbook
- Bash 5 or newer for an `EPOCHREALTIME`-based latency wrapper

## Limitations

- The repository contains procedural guidance, not the wrapper scripts or scheduled jobs themselves.
- CLI flags and tool names can change; verify them against the installed version before use.
- Regex-based secret checks are a safety net, not a replacement for a dedicated secret scanner.
