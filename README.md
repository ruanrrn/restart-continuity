# Restart Continuity

English | [简体中文](README.zh-CN.md)

![Restart Continuity banner](assets/social-preview.svg)

![OpenClaw Skill](https://img.shields.io/badge/OpenClaw-Skill-2D3142?style=flat-square)
![Focus-Restart Recovery](https://img.shields.io/badge/Focus-Restart%20Recovery-EF8354?style=flat-square&labelColor=2D3142)
![Works-Standalone](https://img.shields.io/badge/Works-Standalone-EAEAEA?style=flat-square&labelColor=4F5D75)
![Artifact-.skill Included](https://img.shields.io/badge/Artifact-.skill%20Included-BFC0C0?style=flat-square&labelColor=4F5D75)
![README-Bilingual](https://img.shields.io/badge/README-Bilingual-EAEAEA?style=flat-square&labelColor=4F5D75)
![License-MIT](https://img.shields.io/badge/License-MIT-EAEAEA?style=flat-square&labelColor=2D3142)

Preserve and resume in-flight OpenClaw work across gateway restarts so active tasks survive the restart boundary with usable context.

## Overview

`restart-continuity` is a narrowly scoped OpenClaw skill for restart-safe recovery.

It tells an agent how to preserve the top active task before a restart, carry forward the identifiers and next action that still matter afterward, and proactively resume that work once the environment is back.

The repo is intentionally focused. It is useful on its own, and it does not try to absorb broader multitask orchestration, general state maintenance, or unrelated workflow policy.

## Why this exists

A restart should behave like a short blackout, not a memory wipe.

In practice, agents often lose the exact task, approval IDs, job IDs, process state, or user-facing follow-up message that should survive a restart. The result is predictable: the user has to restate work that was already in progress, and the assistant looks unreliable.

`restart-continuity` exists to make that failure mode explicit and fixable. It provides a repeatable operational pattern for capturing the right restart state before interruption and resuming the correct task first after recovery.

## Scope

Use this repo when the main problem is restart-boundary continuity.

Good fit:

- a gateway restart is planned and active work must survive it
- a task already spans a restart and should be resumed deliberately
- the assistant must persist the top unfinished task and its next action
- live identifiers such as approvals, jobs, sessions, ports, or file paths must survive the restart boundary
- the assistant should proactively tell the user what resumed after restart

Not a fit:

- ongoing multitask prioritization during normal work
- generic synchronization between continuity files while no restart is involved
- broad workflow governance unrelated to restarts

Use `restart-continuity` for the restart boundary itself, not for every continuity problem in the system.

## What the skill covers

The skill standardizes the operational pieces that determine whether restart recovery is trustworthy:

- update `memory/active-task.md` before restart with the real top task, current status, blockers, next action, and queued user-facing update
- record live identifiers that will still matter after recovery, including approvals, job IDs, sessions, processes, ports, and file paths
- create a one-shot fallback cron job for intentional restarts and persist its `jobId`
- resume the top unfinished task immediately after restart when it is safe and not blocked
- send the proactive restart update in the first substantive reply after recovery
- clear stale fallback state after successful resumption so the recovery trail stays clean

## Workflow summary

A normal `restart-continuity` pass looks like this:

1. Capture the real top task in `memory/active-task.md` before restart.
2. Persist the identifiers and exact next step required for post-restart recovery.
3. Schedule a one-shot fallback cron job for intentional restarts.
4. Restart the gateway only after the recovery trail is complete.
5. Read `memory/active-task.md` first after restart and resume the unfinished top task.
6. Tell the user what resumed, then remove stale fallback state.

## When to use it

Reach for `restart-continuity` when the question is not "how should this workflow work in general?" but "how do we survive a restart without dropping the active task?"

Typical triggers:

- "Restart the gateway, but make sure this publish job continues afterward."
- "We restarted in the middle of work; resume the correct task first."
- "Persist the approvals and next step before restarting."
- "Send the user a proactive update once the task is resumed."

## Representative outcomes

### Planned restart during active work

A gateway restart is required while a repository publish, review, or debugging task is still in progress.

A good agent should record the true top task, persist the important IDs, schedule the fallback cron job, and restart only after the recovery instructions are durable.

### Post-restart recovery

The environment comes back after a planned or unplanned restart.

A good agent should read `memory/active-task.md` before unrelated work, resume the unfinished top task when safe, and tell the user what resumed in the first substantive reply.

### Cleanup after successful resumption

The resumed task completes or another unfinished task becomes primary.

A good agent should clear stale restart state, remove any fallback cron job, and either empty or rewrite `memory/active-task.md` so the next top task is accurate.

## Related skill repos

These repositories are related examples, not required dependencies:

- `task-orchestrator`: orchestration-focused companion for multitask scheduling and prioritization - <https://github.com/ruanrrn/task-orchestrator>
- `task-state-sync`: state-accuracy companion for keeping continuity files aligned during live work - <https://github.com/ruanrrn/task-state-sync>
- `multi-task-continuity`: umbrella repo that combines orchestration, state sync, and restart-safe recovery - <https://github.com/ruanrrn/multi-task-continuity>

Start here when the main failure happens at the restart boundary.

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

> Preserve and resume in-flight work across restarts without losing the active task.

GitHub note:

- The current `gh` CLI and GraphQL `UpdateRepositoryInput` do not expose a writable custom social preview field.
- To use this image as the repository social preview, upload `assets/social-preview.svg` manually in the repo settings UI.

## Repository layout

```text
restart-continuity/
├── LICENSE
├── README.md
├── README.zh-CN.md
├── CONTRIBUTING.md
├── assets/
│   └── social-preview.svg
├── restart-continuity/
│   └── SKILL.md
└── dist/
    └── restart-continuity.skill
```

## Contributing

See `CONTRIBUTING.md` for contribution scope, PR expectations, and the repo boundary that keeps this project focused on restart-safe recovery instead of broader continuity tooling.

## Release hygiene

- regenerate `dist/restart-continuity.skill` after each material skill change
- keep `README.md`, `README.zh-CN.md`, `SKILL.md`, and repository metadata aligned
- preserve the narrow scope: restart continuity first, broader workflow concerns elsewhere
- remove stale restart examples, IDs, or wording that could imply the repo owns more than it does

## Repository

- GitHub: `https://github.com/ruanrrn/restart-continuity`
- License: MIT
