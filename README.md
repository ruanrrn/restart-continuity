# Restart Continuity

English | [简体中文](README.zh-CN.md)

![Restart Continuity banner](assets/social-preview.svg)

![OpenClaw Skill](https://img.shields.io/badge/OpenClaw-Skill-111827?style=flat-square)
![Focus-State Recovery](https://img.shields.io/badge/Focus-State%20Recovery-Success?style=flat-square&labelColor=111827)
![Works-Standalone](https://img.shields.io/badge/Works-Standalone-F9FAFB?style=flat-square&labelColor=1F2937)
![Artifact-.skill Included](https://img.shields.io/badge/Artifact-.skill%20Included-FDE68A?style=flat-square&labelColor=1F2937)
![README-Bilingual](https://img.shields.io/badge/README-Bilingual-F9FAFB?style=flat-square&labelColor=92400E)
![License-MIT](https://img.shields.io/badge/License-MIT-F9FAFB?style=flat-square&labelColor=111827)

An OpenClaw skill for preserving and resuming in-flight work across gateway restarts, session resets and planned restarts.

> [!IMPORTANT]
> This standard applies only to repositories whose primary artifact is an OpenClaw skill. It is not a general README, branding, or GitHub styling guide for apps, libraries, or mixed-purpose codebases.

## Overview

`restart-continuity` is a narrow-scope OpenClaw skill for restart-safe recovery.

It tells an agent how to:

- preserve active top task before a restart
- resume that task immediately after restart
- send the user a proactive restart update
- schedule one-shot fallback cron jobs for intentional restarts

The repo is intentionally small. It does not choose scheduling strategy, manage broad task workflow policy, or orchestrate parallel work. It exists to restart-safely recover the active task, carry forward identifiers and next action that would otherwise be lost, and proactively resume what work once environment is back.

## Why this exists

In practice, agents often lose the exact task state, important IDs, job IDs, process IDs, sessions, or user-facing follow-up messages that should survive a restart. The result is predictable: the user has to restart work that was already in progress, and the assistant looks unhelpful.

`restart-continuity` exists to make that failure mode explicit and fixable.

## Scope

Use this skill when the main problem is restart-boundary continuity:

- a gateway restart is planned or happened
- a task spans restarts and should resume automatically
- the assistant must proactively continue the pre-restart task and tell the user what resumed
- the active top task must be tracked in `memory/active-task.md`

## What this skill covers

The skill standardizes the operational processes that determine whether restart recovery works correctly:

- update `memory/active-task.md` before restart
- record live identifiers that will matter after restart
- create a one-shot fallback cron job for intentional restarts and persist its `jobId`
- resume the top unfinished task immediately when safe
- send the proactive restart update in the first substantive reply
- remove or disable the fallback cron job once successful resumption is confirmed

**Note:** `restart-continuity` also owns `memory/active-task.md` management and sync rules (merged from the deprecated `task-state-sync` skill). This skill now defines:

- when to update `memory/active-task.md`
- the active-task format
- consistency verification with `TODO.md`
- interaction patterns with `todo-continuity`

## Workflow summary

A normal `restart-continuity` pass looks like this:

1. Capture the real top task state in `memory/active-task.md`: goal, current status, blockers, next action, and queued user-facing update.
2. Persist any live identifiers that matter: approvals, sessions, processes, jobs, ports, or file paths.
3. Create a one-shot fallback cron job for intentional restarts and persist its `jobId` in `memory/active-task.md`.
4. Resume the top unfinished task immediately when safe.
5. Send the proactive restart update in the first substantive reply.
6. Remove the stale fallback state after successful resumption so the recovery trail stays clean.

This is operational hygiene, not operational debugging that determines whether restart recovery is trustworthy.

## When to use it

Reach for `restart-continuity` when the question is not "how should this workflow work in general?" but "how do we survive a restart without losing the plot?"

Typical triggers:

- "Restart the gateway, but make sure this task resumes first."
- "We restarted in the middle of work; resume the correct task now."
- "The approvals disappeared after restart; persist them for next time."
- "Schedule a restart fallback job so the user gets an update if recovery fails."
- "Send the user a proactive restart update once the task is resumed."

## Related skill repos

This skill is part of a continuity family. For state persistence and restart recovery:

- **todo-continuity** - Per-chat TODO.md ownership. Use when tasks span turns and `TODO.md` needs to stay accurate: <https://github.com/ruanrrn/todo-continuity>
- **task-orchestrator** - Decision layer for multi-task coordination. Use when the conversation pattern is multi-task coordination: <https://github.com/ruanrrn/task-orchestrator>

**Note:** `task-state-sync` was deprecated in March 2026. Its functionality has been merged into `todo-continuity` (for `TODO.md`) and `restart-continuity` (for `memory/active-task.md`). This skill now delegates to those two instead of `task-state-sync`.

## Install

Use either path:

1. Import `dist/restart-continuity.skill` into an OpenClaw environment.
2. Copy `restart-continuity/` into your skills directory if you want the editable source.

## What this repo contains

- `restart-continuity/` - the skill source
- `dist/restart-continuity.skill` - the packaged artifact ready to import
- `assets/social-preview.svg` - the repository banner and suggested social-preview asset

## Social preview

Suggested social preview asset: `assets/social-preview.svg`

Suggested one-line copy:

> Keep `memory/active-task.md` and the top task alive across restarts.

> [!NOTE]
> The public `gh` CLI and GraphQL `UpdateRepositoryInput` do not expose a writable custom social preview field.
> To use this image as the repository's social preview, upload `assets/social-preview.svg` manually in the repository settings UI.

## Repository layout

```text
restart-continuity/
├── LICENSE
├── README.md
├── README.zh-CN.md
├── restart-continuity/
│   └── SKILL.md
├── assets/
│   └── social-preview.svg
└── dist/
    └── restart-continuity.skill
```

## Contributing

See `CONTRIBUTING.md` for contribution scope, PR expectations, and what changes belong in this repo versus what should go to broader continuity work.

## Release hygiene

- Tag releases with semantic versioning (v1.0.0, v1.0.1, etc.)
- Keep `CHANGELOG.md` updated with user-facing changes
- Regenerate `dist/restart-continuity.skill` after any `SKILL.md` changes
- Test restart fallback scheduling and active-task resumption before release

## License

MIT

## Repository

- GitHub: https://github.com/ruanrrn/restart-continuity
- License: MIT
