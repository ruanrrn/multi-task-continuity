# Multi-Task Continuity

English | [简体中文](README.zh-CN.md)

![Multi-Task Continuity banner](assets/social-preview.svg)

![OpenClaw Skill](https://img.shields.io/badge/OpenClaw-Skill-283618?style=flat-square)
![Focus-Restart--Safe Multi--Task Workflow](https://img.shields.io/badge/Focus-Restart--Safe%20Multi--Task%20Workflow-DDA15E?style=flat-square&labelColor=283618)
![Works-Standalone](https://img.shields.io/badge/Works-Standalone-FEFAE0?style=flat-square&labelColor=606C38)
![Artifact-.skill Included](https://img.shields.io/badge/Artifact-.skill%20Included-DDE5B6?style=flat-square&labelColor=606C38)
![README-Bilingual](https://img.shields.io/badge/README-Bilingual-FEFAE0?style=flat-square&labelColor=BC6C25)
![License-MIT](https://img.shields.io/badge/License-MIT-FEFAE0?style=flat-square&labelColor=283618)

Coordinate multiple user requests as one restart-safe operating workflow for OpenClaw agents.

## Overview

`multi-task-continuity` is a standalone OpenClaw skill for agents that need to handle more than one active request without losing task order, user visibility, or restart recovery.

It combines three lanes that often fail when managed informally:

- task orchestration across concurrent or interrupted requests
- continuity-file maintenance for in-flight work
- resume-first recovery after a restart or session reset

The goal is narrow and operational: keep the agent's active work coherent when chat stops being a single linear task.

## Why this exists

Multi-request work usually breaks at the boundaries between otherwise reasonable behaviors.

An agent may prioritize well but fail to persist the current lane. It may write `TODO.md` but not keep `memory/active-task.md` aligned. It may survive a restart in theory but come back speaking about the wrong task in practice.

`multi-task-continuity` exists to close that gap. It defines one working model for triage, persistence, staged progress updates, and restart recovery so those behaviors reinforce each other instead of drifting apart.

## Scope

Use this repo when the job requires the combined workflow.

Good fit:

- users send multiple tasks across separate messages
- some work is long-running, parallelizable, or blocker-sensitive
- priorities may change while work is already in flight
- the active plan must survive restarts or session resets
- `TODO.md` and `memory/active-task.md` must stay aligned as state changes

Not a fit:

- trivial one-shot requests
- purely synchronous single-step work
- narrow fixes that only touch one continuity file
- cases where the smaller companion skills are sufficient on their own

This repository is an umbrella workflow skill, not a mandatory base layer for every agent.

## What the skill covers

The skill instructs the agent to run multi-request work as one controlled workflow:

- split incoming messages into discrete tasks instead of treating chat as FIFO
- choose a safe execution order and use parallel lanes when the work truly allows it
- keep the main thread focused on orchestration, user communication, and decisions
- persist the per-chat unfinished queue in `TODO.md`
- persist the single resume-first lane in `memory/active-task.md`
- update both continuity files whenever blockers, priorities, important IDs, or next steps change materially
- send staged progress updates while work is in flight
- resume the correct top task first after restart, then rebuild the broader queue

## Workflow summary

A normal pass through this workflow has five phases:

1. Build the task map: identify goals, urgency, dependencies, conflicts, runtime, and safe parallelism.
2. Choose the active lanes: start unblockers and urgent work first, then launch worthwhile long-running work early.
3. Persist the truth: write the queue to `TODO.md` and the top resume-first task to `memory/active-task.md`.
4. Report progress: tell the user what finished, what is still running, what is blocked, and what changed.
5. Recover after restart: resume from `memory/active-task.md` first, then reconstruct the remaining queue from `TODO.md`.

## When to use it

Reach for `multi-task-continuity` when the risk is not the individual task itself, but the coordination overhead around several tasks.

Typical triggers:

- "Handle these three requests in parallel and keep me updated."
- "Do the urgent fix first, but do not lose the rest of the queue."
- "If a restart happens, resume the right task and tell me what you picked back up."
- "Keep `TODO.md` and `memory/active-task.md` accurate while priorities shift."

## Representative outcomes

### Mixed short and long work

A user asks for a bug fix, a log summary, and a PR review in quick succession.

A good agent should identify the blocker-sensitive task, start any safe background lane early, use waiting time for shorter work, persist the queue, and report partial results as they land.

### Urgent interruption

A production issue arrives while other work is already in progress.

A good agent should immediately re-rank the queue, rewrite both continuity files to reflect the new top task, and explain the priority change before continuing.

### Restart during active work

The user wants an active fix to survive a restart while secondary work keeps moving.

A good agent should keep the top lane explicit in `memory/active-task.md`, preserve the broader queue in `TODO.md`, and resume the correct task first after restart.

## Related skill repos

These repositories are related examples, not required dependencies:

- `task-orchestrator`: focused orchestration and prioritization lane - <https://github.com/ruanrrn/task-orchestrator>
- `task-state-sync`: focused continuity-file maintenance lane - <https://github.com/ruanrrn/task-state-sync>

Use this repository when you want the combined operating model in one place instead of composing the narrower lanes manually.

## Install

Use either path:

1. Import `dist/multi-task-continuity.skill` into an OpenClaw environment.
2. Copy `multi-task-continuity/` into your skills directory if you want the editable source.

## What this repo contains

- `multi-task-continuity/` - the skill source
- `dist/multi-task-continuity.skill` - the packaged artifact ready to import
- `assets/social-preview.svg` - the repository banner and suggested social-preview asset

## Social preview

Suggested social preview asset: `assets/social-preview.svg`

Suggested one-line copy:

> One workflow for orchestration, state sync, and restart-safe recovery.

GitHub note:

- The current `gh` CLI and GraphQL `UpdateRepositoryInput` do not expose a writable custom social preview field.
- To use this image as the repository social preview, upload `assets/social-preview.svg` manually in the repo settings UI.

## Repository layout

```text
multi-task-continuity/
├── LICENSE
├── README.md
├── README.zh-CN.md
├── CONTRIBUTING.md
├── assets/
│   └── social-preview.svg
├── multi-task-continuity/
│   └── SKILL.md
└── dist/
    └── multi-task-continuity.skill
```

## Contributing

See `CONTRIBUTING.md` for contribution scope, PR expectations, and the boundary that keeps this repo focused on multi-request continuity rather than general agent workflow design.

## Release hygiene

- regenerate `dist/multi-task-continuity.skill` after each material skill change
- keep `README.md`, `README.zh-CN.md`, and `multi-task-continuity/SKILL.md` aligned
- keep the repository scoped to the combined workflow only
- update related-repo links if the companion repos move or change names

## Repository

- GitHub: `https://github.com/ruanrrn/multi-task-continuity`
- License: MIT
