# Multi-Task Continuity

An OpenClaw skill for running multiple user requests as one coordinated, restart-safe workflow instead of a chat-shaped pileup.

## Why this exists

Most agents can either do multitasking or do continuity, but not both at the same time without turning state into compost.

One skill says how to prioritize work. Another says how to survive restarts. Another says how to keep `TODO.md` and `memory/active-task.md` from drifting apart. Individually, each piece helps. In practice, the real failure mode happens in the gaps between them: the agent starts parallel work, forgets to persist the active lane, restarts mid-flight, and comes back speaking confidently about the wrong task.

`multi-task-continuity` closes that gap.

It combines three behaviors into one operational workflow:

- orchestrate multiple incoming tasks intelligently
- persist the real state as priorities and blockers change
- resume the correct top task after a restart, then rebuild the broader queue

This is the umbrella skill for agents that need to behave like competent operators under real chat conditions.

## What the skill teaches

The skill tells the agent to:

- split incoming requests into real tasks instead of treating chat as FIFO
- choose safe parallelism and launch long-running valuable work early
- keep the main thread on orchestration and user communication
- write per-chat unfinished state to `TODO.md`
- write the top resume-first lane to `memory/active-task.md`
- schedule restart fallbacks when intentional restarts would otherwise risk dropping work
- send staged progress updates instead of waiting for a grand finale
- repair continuity state before continuing if the files drift apart

## When to use it

Use `multi-task-continuity` when:

- a user sends multiple tasks across separate messages
- some work is long-running, parallelizable, or blocker-sensitive
- priorities may change during execution
- the active plan must survive restarts or session resets
- `TODO.md` and `memory/active-task.md` need to stay aligned throughout the work

Do not use it for trivial one-shot tasks. That would be like bringing air traffic control to a bicycle ride.

## Workflow overview

### 1. Build the task map

Split the incoming requests into discrete tasks and capture:

- goal
- urgency
- dependencies
- conflicts
- likely runtime
- parallel safety
- expected user-visible output

### 2. Choose the active lanes

Default ordering:

1. unblockers and urgent work
2. long-running independent work worth starting early
3. quick wins that fit into wait windows
4. cleanup and secondary polish

### 3. Persist the truth

- write the per-chat queue into `TODO.md`
- write the single resume-first task into `memory/active-task.md`
- update both when priorities, blockers, IDs, or next steps change materially

### 4. Report progress

Tell the user:

- what finished
- what is still running
- what is blocked
- what changed in priority
- what happens next

### 5. Recover after restart

- resume the top task from `memory/active-task.md`
- send the queued restart update in the first substantive reply
- rebuild the remaining queue from `TODO.md`
- clear stale fallback state once recovery succeeds

## Example

User sends:

- "Fix the deploy script"
- "Review this PR too"
- "And keep me posted if a restart interrupts anything"

A good agent should:

1. inspect the deploy issue first if it is blocker-sensitive
2. launch the PR review in a background lane if safe
3. write the current chat queue to `TODO.md`
4. write the deploy lane to `memory/active-task.md` if it is the top task
5. report the deploy result as soon as it lands
6. after restart, resume the deploy lane first and then continue the review lane

## Relationship to smaller skills

This umbrella skill replaces the need to manually compose several narrower behaviors every time.

Related building blocks:

- `task-orchestrator`: <https://github.com/ruanrrn/task-orchestrator>
- `task-state-sync`: <https://github.com/ruanrrn/task-state-sync>
- `todo-continuity`: workspace/local continuity pattern
- `restart-continuity`: workspace/local restart recovery pattern

If you only need one narrow capability, use the smaller skill. If you need the whole operating model, use this one.

## What you get

- `multi-task-continuity/` - the skill source
- `dist/multi-task-continuity.skill` - packaged artifact ready to import

## Install

Use either path:

1. Import `dist/multi-task-continuity.skill` into an OpenClaw environment.
2. Copy `multi-task-continuity/` into your skills directory if you want the editable source.

## Repository layout

```text
multi-task-continuity/
├── LICENSE
├── README.md
├── multi-task-continuity/
│   └── SKILL.md
└── dist/
    └── multi-task-continuity.skill
```

## Release hygiene

- Regenerate `dist/multi-task-continuity.skill` after each material skill change
- Keep the repo focused on the umbrella workflow only
- Keep the README examples aligned with the actual workflow in `SKILL.md`
- Update links to companion skills when those repos change

## Repository

- GitHub: `https://github.com/ruanrrn/multi-task-continuity`
- License: MIT
