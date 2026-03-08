---
name: multi-task-continuity
description: "Run multi-request work as one coherent, restart-safe workflow. Use when: (1) a user sends multiple tasks across separate messages, (2) some work is long-running or parallelizable, (3) the active plan must survive restarts or session resets, (4) TODO.md and memory/active-task.md must stay aligned while priorities change."
---

# Multi-Task Continuity

Treat multiple user requests as one living workflow, not as a pile of unrelated chat turns. This skill bundles orchestration, continuity, and state sync into one recommended operating model.

## What this skill owns

Use this skill when the agent would otherwise need to combine several narrower continuity skills by hand and the whole combined workflow matters more than keeping every lane separate.

It is a convenience bundle, not a required foundation.

Use it when the agent must do all of the following together:

- decide task order and safe parallelism
- keep staged progress visible to the user
- maintain per-chat unfinished state in `TODO.md`
- maintain the top resume-first task in `memory/active-task.md`
- survive restarts without losing the plot

If you only need one narrow behavior, use the smaller companion skills directly. Reach for this bundle when you want the combined behavior with less composition overhead, not because the smaller repos are insufficient.

## Unified operating model

Think in three layers:

1. `orchestrate` - decide ordering, urgency, dependencies, and parallel lanes
2. `persist` - write the real task state into continuity files when it changes materially
3. `resume` - recover the top task first after restart, then rebuild the remaining queue

Failure in any layer breaks the experience.

## Phase 1: Build the task map

Before acting, split the user's incoming requests into discrete tasks.

For each task, note:

- goal
- urgency
- dependencies
- conflicts
- likely runtime
- whether it can run in parallel
- what output should be reported to the user

Do not default to strict arrival order unless the user explicitly asks for it.

## Phase 2: Choose the active lanes

Use this default order unless the user overrides it:

1. unblockers and urgent work
2. long-running independent tasks worth starting early
3. quick wins that fit into wait windows
4. cleanup and secondary polish

When possible:

- keep the main thread on orchestration, decisions, and user communication
- push slower execution into subthreads or subagents
- avoid launching conflicting work at the same time

Pause and ask only when tasks truly conflict, require consent, or need a user decision.

## Phase 3: Persist the truth

As soon as state changes materially, update the continuity files.

### Write `TODO.md`

Use `TODO.md` as the per-chat unfinished queue.

Keep the current chat section short and operational:

- `Context`
- `Goal`
- `In progress`
- `Next step`
- `Blockers`
- `Important IDs`
- `Resume message`

Update it when:

- multiple tasks are active
- blockers appear or clear
- important IDs appear
- the next step changes
- a finished task should be removed

### Write `memory/active-task.md`

Use `memory/active-task.md` for the single top task that must resume first.

Keep only:

- `Goal`
- `Current status`
- `Latest important IDs`
- `Next step`
- `If blocked`
- `User update after restart`
- `Done when`

Update it when:

- one task becomes the clear top priority
- restart pain would be high without a scratchpad
- the resume sentence changed
- the top task finishes and another task takes over

If no active top task remains, clear the file instead of preserving stale fiction.

## Phase 4: Report progress like a sane operator

Do not wait for all work to finish before replying.

Progress updates should say:

- what finished
- what is still running
- what is blocked
- what changed in priority, if anything
- what you are doing next

Do not spam routine tool narration. Do not disappear silently either.

## Phase 5: Handle restarts intentionally

When a restart is planned during active work:

- update both continuity files first
- make sure `memory/active-task.md` names the true top task
- if required, schedule the one-shot fallback cron job described by restart continuity
- record the fallback `jobId` in `memory/active-task.md`

After restart:

- read `memory/active-task.md` first
- resume the top unfinished task immediately if safe
- send the queued user-facing resume update in the first substantive reply
- then rebuild the broader queue from `TODO.md` if more tasks remain
- remove or disable the fallback cron job once successful resumption is confirmed

## State consistency rules

Treat these as invariants:

- the top task in `memory/active-task.md` must make sense inside the current chat section in `TODO.md`
- finished tasks should not remain active in either file
- important IDs should not contradict each other
- the recorded next step must still be the next step

If the files disagree, repair the state before continuing complex work.

## Example workflow

User sends:

- "Fix the deploy script"
- "Also review this PR"
- "And remind me what broke after the restart"

Handle it like this:

1. classify the deploy script as likely blocker-sensitive
2. launch the PR review in a background lane if appropriate
3. update `TODO.md` with the full queue for the current chat
4. set `memory/active-task.md` to the deploy issue if it is the top task
5. report the deploy result as soon as it lands
6. if a restart happens mid-flight, resume the deploy lane first, then continue the review lane

## When not to use this skill

Do not trigger this skill for:

- one-off trivial tasks
- purely synchronous single-step work
- narrow continuity fixes where only `TODO.md` or only `memory/active-task.md` needs adjustment
- tasks that do not span messages, waits, or restarts

## Failure modes to avoid

- treating chat like a FIFO queue
- forgetting to update continuity state during long work
- letting `TODO.md` and `memory/active-task.md` drift apart
- restarting without a real recovery trail
- waiting silently on long tasks
- holding partial results until everything is done
- asking the user to say "continue" on already-active work
