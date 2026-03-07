---
name: restart-continuity
description: Preserve and resume in-flight work across restarts. Use when a task spans restarts, a gateway restart is planned or happened, or the assistant must proactively continue the pre-restart task and tell the user what resumed.
---

# Restart Continuity

Treat restarts like brief blackouts, not amnesia.

## Before a restart

- Update `memory/active-task.md` with the current goal, latest working state, blockers, the exact next action, and the user-facing sentence to send after restart.
- Record any live IDs that matter, such as approval IDs, job IDs, process IDs, ports, sessions, or file paths.
- If code or config changed, name the changed files explicitly.
- If multiple tasks exist, mark one as the top task and keep the others clearly secondary.
- For any intentional restart, create a one-shot cron fallback that sends a `systemEvent` back into the main session 60-120 seconds later with a short instruction to resume `memory/active-task.md` and send the queued user update. Write that cron `jobId` into `memory/active-task.md` before restarting.
- Treat the cron fallback as mandatory backup, not optional polish. The built-in post-restart ping is helpful, but it is not enough on its own.

## After a restart

- Read `memory/active-task.md` before doing unrelated work.
- Resume the top unfinished task immediately if it is safe and not blocked.
- In the first substantive reply, tell the user what task resumed and what you are doing next.
- Treat failure to send that proactive restart update as a workflow failure, not a minor omission.
- Do not ask the user to say "continue" on an already active task.
- Keep updating `memory/active-task.md` as the task progresses.
- If a restart fallback cron job was scheduled, remove or disable it as soon as successful resumption is confirmed so it does not fire later and create duplicate nudges.
- If the built-in restart ping does not produce visible continuation quickly, rely on the cron fallback rather than waiting for the user to remind you.

## Fallback cron template

Before an intentional restart, schedule a one-shot fallback like this:

- `sessionTarget`: `main`
- `payload.kind`: `systemEvent`
- `schedule.kind`: `at`
- `schedule.at`: about 60-120 seconds in the future
- `payload.text`: a short instruction such as `Restart follow-up: resume memory/active-task.md now, continue the top unfinished task, send the queued user update, then remove this fallback job.`

Write the returned `jobId` into `memory/active-task.md`, and remove the job after successful resumption.

## Active-task format

Keep `memory/active-task.md` short and operational:

- `Goal:` what success looks like
- `Current status:` latest true state
- `Latest important IDs:` approvals, sessions, processes, jobs, including any restart fallback cron `jobId`
- `Next step:` one concrete next action
- `If blocked:` exact unblock condition
- `User update after restart:` one sentence to send proactively
- `Done when:` short completion test

## When the task finishes

- Remove or clear `memory/active-task.md` once the top task is complete and no immediate continuation work remains.
- If follow-up work remains, rewrite `memory/active-task.md` so the next unfinished task becomes the new top task.
- Do not leave stale approval IDs or outdated next steps in the file.

## Blocking rules

Pause and ask only if one of these is true:

- Approval or credentials are required
- The next action is destructive or external
- The user must choose between materially different options

Otherwise, keep moving.

## For this workspace

- Read `SOUL.md`, `USER.md`, today's and yesterday's memory files, and `memory/active-task.md` every session.
- Keep `memory/active-task.md` current whenever work is likely to survive a restart.
- When a restart was intentional, resume that task first and mention the resumed task in the first substantive reply.
