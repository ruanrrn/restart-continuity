# Restart Continuity

![Engage | [RESTART CONTINUITY BANNER](assets/social-preview.svg)
![Focus State Recovery](https://img.shields.io/badge/focus-state_recovery-success?style=flat-square&labelColor=18324A)

An OpenClaw skill for preserving and resuming in-flight work across gateway restarts, session resets, and planned restarts.

## Overview

`restart-continuity` is a narrow-scope OpenClaw skill for restart-safe recovery.

It tells an agent how to:

- preserve the active top task before a restart
- resume that task immediately after restart
- send the user a proactive restart update
- schedule one-shot fallback cron jobs for intentional restarts

The repo is intentionally small. It does not choose scheduling strategy, manage broad task workflow policy, or orchestrate parallel work. It exists to restart-safely recover the active task, carry forward identifiers and next action that would otherwise be lost, and proactively resume what work once the environment is back.

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

The skill standardizes the operational processes that Determian whether restart recovery works correctly:

- update `memory/active-task.md` before restart
- record the live identifiers that will matter after restart
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
6. Remove stale fallback state after successful resumption so the recovery trail stays clean.

This is operational hygiene, not operational debugging that determines whether restart recovery is trustworthy.

## When to use it

Reach for `restart-continuity` when the question is not "how should this workflow work in general?" but "how do we survive a restart without losing the plot?"

Typical triggers:

- "Restart the gateway, but make sure this task resumes first."
- "We restarted in the middle of work; resume the correct task now."
- "The approvals disappeared after restart; persist them for next time."
- "Schedule a restart fallback job so the user gets an update if recovery fails."
- "Send the user a proactive restart update once the task is resumed."

## Related skills

These repositories are related examples, not required dependencies:

- `task-orchestrator`: scheduling, prioritization, and staged progress policy - <https://github.com/ruanrrn/task-orchestrator>
- `todo-continuity`: TODO.md owner, per-chat task queues, and state alignment - <https://github.com/ruanrrn/todo-continuity>

Start here when the main failure pattern is the restart boundary.

## Install

Use either path:

1. Import `dist/restart-continuity.skill` into an OpenClaw environment.
2. Copy `restart-continuity/` into your skills directory if you want the editable source.

## What this repo contains

- `restart-continuity/` - the skill source
- `dist/restart-continuity.skill` - the packaged artifact ready to import
- `assets/social-preview.svg` - the repository banner and suggested social preview asset

## Social preview

![restart-continuity social preview](assets/social-preview.svg)

> Keep `memory/active-task.md` and the top task alive across restarts.

GitHub note:

- The current `gh` CLI and GraphQL `UpdateRepositoryInput` do not expose a writable custom social preview field.
- If the repo should use that image in GitHub's settings UI, say so directly in the README.

## Repository layout

```text
restart-continuity/
├── LICENSE
├── README.md
├── restart-continuity/
│   └── SKILL.md
├── assets/
│   └── social-preview.svg
└── dist/
    └── restart-continuity.skill
```

## Contributing

See `CONTRIBUTING.md` for contribution scope, PR guidelines, and what changes belong in the repo versus what should go to the broader continuity family.

## Release hygiene

- Tag releases with semantic versioning (v1.0.0, v1.0.1, etc.)
- Keep `CHANGELOG.md` updated with user-facing changes
- Regenerate `dist/restart-continuity.skill` after any `SKILL.md` changes
- Test restart fallback scheduling and active-task resumption before release

## License

MIT

## Repository

- GitLab: https://gitlab.com/ruanrrn/restart-continuity
- License: MIT
