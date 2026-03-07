# Contributing to Multi-Task Continuity

Keep contributions coherent, practical, and worthy of an umbrella skill.

## Scope

Good contributions:

- clearer end-to-end workflow guidance
- better examples of task orchestration, state sync, and restart recovery working together
- bilingual README improvements that keep English and Chinese versions aligned
- packaging and repository polish that strengthens the repo as a complete standalone entry point

Avoid:

- turning the repo into a dumping ground for unrelated agent habits
- duplicating whole companion skills line-for-line without adding umbrella-level value
- widening the repo beyond multitask orchestration, continuity, and recovery

## Workflow

1. Make the smallest coherent improvement.
2. Keep the umbrella repo self-contained and independently understandable.
3. Update `README.md` and `README.zh-CN.md` together when user-facing behavior changes.
4. Regenerate `dist/multi-task-continuity.skill` after material skill changes.

## Pull request guidance

A good PR should explain:

- what changed
- why the change improves the combined operating model
- whether the packaged `.skill` artifact was regenerated
- whether both READMEs were updated when needed

## Repo principle

This repo should stay the complete workflow entry point of the family, not an uncurated bundle of everything adjacent.
