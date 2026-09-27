---
name: claude-p-agent-spawn-timeout
description: Use when a `claude -p` invocation running inside a time-boxed launchd/cron job produces an empty or truncated artifact / times out, even though the task looks small — check whether the prompt caused Claude to spawn an internal subagent (Task/Agent tool) to explore many files, which can silently blow past the wrapper's timeout budget.
author: auto
created: 2026-09-16
version: 1.0.0
status: active
---

## Procedure
1. Reproduce manually: run the exact `claude -p "..."` command by hand with `time`, without the launchd/timeout wrapper, and watch wall-clock duration.
2. Check the prompt text for phrasing that invites broad exploration (e.g. "read the raw N days of files", "look through all logs") — this is exactly the kind of instruction that makes Claude launch an internal subagent (Task tool) to search/read, which has unpredictable duration.
3. Fix by doing both:
   - Add `--disallowedTools Agent` (or the CLI's current equivalent flag/tool name) to the `claude -p` invocation to block internal subagent spawning.
   - Bound the read scope explicitly in the prompt itself (e.g. "read at most 20 files") so the direct (non-agent) read also stays fast.
4. Re-kick the job once by hand and confirm it finishes well inside the timeout with exit 0 and a non-empty, correctly sized artifact.

## Pitfalls
- The `timeout N` wrapper kills the whole process with no diagnostic — logs just show an empty/missing artifact, which looks like a quota or API failure but usually isn't. Rule out subagent spawning first before chasing quota/network theories.
- Confirm the exact disallow-tool flag name/spelling matches the installed CLI version; a stale flag name silently no-ops instead of erroring.
- Bounding file count in the prompt alone is not sufficient if the prompt still implies "explore" — Claude may still spawn a subagent for a small-looking task. Block the tool explicitly.

## Verification
- Run the job's exact command by hand; confirm completion time is well under the timeout, exit 0, and the output artifact is non-empty with plausible size/content.
- Re-trigger the scheduled job (or wait for its next slot) and confirm the WARN/empty-output no longer appears in its log.
