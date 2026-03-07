# Restart Continuity

Preserve and resume in-flight OpenClaw work across gateway restarts without amnesia.

## Why this exists

Restarts shouldn't erase context. When a gateway restarts mid-task, agents often forget what they were doing and require the user to manually remind them to continue. This creates friction and interrupts flow, especially for long-running tasks that span multiple restart cycles.

OpenClaw has built-in post-restart ping mechanisms, but they're not sufficient for reliable task resumption. The assistant needs a disciplined approach to persist task state before restarts and automatically resume afterward, without waiting for human prompts.

This skill provides that discipline: a standardized workflow for tracking active tasks, persisting critical state, and ensuring work continues seamlessly across restarts.

## What you get

- `restart-continuity/SKILL.md` - the complete restart continuity workflow
- `dist/restart-continuity.skill` - packaged artifact ready to install

## Install

Use either path:

1. Import `dist/restart-continuity.skill` into an OpenClaw environment.
2. Copy `restart-continuity/` into your skills directory if you want the editable source.

## When to use the skill

Use this skill when:

- A task spans restarts (intentional or unplanned)
- A gateway restart is planned and work should survive it
- The assistant must proactively continue the pre-restart task
- You need to track live IDs that matter (approvals, sessions, processes, jobs)
- Long-running workflows need resilience against restart interruptions

## Skill outcome

The skill establishes a complete restart-safe workflow:

**Before a restart:**
- Update `memory/active-task.md` with goal, status, blockers, next action, and user-facing restart message
- Record live IDs (approvals, sessions, processes, jobs, ports)
- For intentional restarts, create a one-shot cron fallback with a `systemEvent` that triggers resumption 60-120 seconds later
- Write the fallback cron `jobId` into `memory/active-task.md`
- Treat the cron fallback as mandatory, not optional

**After a restart:**
- Read `memory/active-task.md` before unrelated work
- Resume the top unfinished task immediately if safe and unblocked
- In the first substantive reply, tell the user what task resumed and what you're doing next
- Treat missing restart notification as a workflow failure
- Remove or disable the fallback cron job after successful resumption

**Active-task format:**
```markdown
Goal: what success looks like
Current status: latest true state
Latest important IDs: approvals, sessions, processes, jobs, fallback cron jobId
Next step: one concrete next action
If blocked: exact unblock condition
User update after restart: one sentence to send proactively
Done when: short completion test
```

**When the task finishes:**
- Remove or clear `memory/active-task.md` once complete and no continuation remains
- If follow-up exists, rewrite `memory/active-task.md` with the next unfinished task as the new top task
- Don't leave stale approval IDs or outdated next steps

## Repository layout

```text
restart-continuity/
├── LICENSE
├── README.md
├── restart-continuity/
│   └── SKILL.md
└── dist/
    └── restart-continuity.skill
```

## Release hygiene

- Keep the repository description aligned with the skill trigger language
- Regenerate `dist/restart-continuity.skill` after each material skill change
- Keep the repo narrow and practical; no unrelated debug junk

## Repository

- GitHub: `https://github.com/ruanrrn/restart-continuity`
- License: MIT
