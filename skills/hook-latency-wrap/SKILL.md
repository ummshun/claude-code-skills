---
name: hook-latency-wrap
description: Use when measuring the duration and failure rate of any Claude Code hook, such as PostToolUse, Stop, or UserPromptSubmit, with a transparent Bash wrapper based on EPOCHREALTIME.
author: auto
created: 2026-05-29
version: 1.0.0
status: active
disallowed-tools: Agent
---

## Procedure

1. Create a wrapper command that accepts the real hook path as its first argument, preserves stdin, and records `$EPOCHREALTIME` immediately before and after invoking the hook.
2. Append one JSONL record per run with fields such as `ts`, `hook`, `duration_ms`, and `exit_code`. Store it under `$HOME/.claude/logs/` or another configurable log directory.
3. Replace the target hook's `command` in `settings.json` with the wrapper followed by the real hook path. The same wrapper can instrument multiple hooks.
4. Return the real hook's exit code. If metrics logging itself fails, do not mask or replace the hook result.
5. Aggregate a bounded time window, such as the last seven days, by hook name and report p50, p95, maximum duration, and failure rate.

## Pitfalls

- `$EPOCHREALTIME` requires Bash 5 or newer. macOS ships Bash 3.2, so use an installed Bash 5 path in the wrapper shebang.
- Spawning Python for each timing operation can add enough overhead to distort short hooks.
- Keep metrics on a separate file descriptor so hook stdout and stderr do not enter the JSONL log.
- Fire-and-forget hooks finish after the wrapper exits, so wrapper duration does not represent their actual completion time.
- Measure wrapper overhead separately if millisecond-level precision matters.

## Verification

- Run `bash -n <wrapper-path>`.
- Pipe a small JSON object into the wrapper and a harmless test hook.
- Confirm that stdin reaches the hook unchanged, the wrapper returns the same exit code, and exactly one valid JSONL record is added.
- Trigger the configured Claude Code hook and confirm the report includes it with plausible timing values.
