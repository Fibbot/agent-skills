# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

## Project Context

## Tech Stack

## Core Architecture

### Key Design Patterns


## Database Schema (xx migrations)


## Development Commands

## Common Queries

## Next Implementation Steps


FILL IN THE ABOVE WHEN ENOUGH CONTEXT HAS BEEN GAINED
---


## Workflow Orchestration

### 1. Plan Mode Default
- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
- If something goes sideways, STOP and re-plan immediately — don't keep pushing
- Use plan mode for verification steps, not just building
- Write detailed specs upfront to reduce ambiguity

### 2. Subagent Strategy
- Use subagents liberally to keep main context window clean
- Offload research, exploration, and parallel analysis to subagents
- For complex problems, throw more compute at it via subagents
- One task per subagent for focused execution

### 3. Recording What You Learned
- **There is no lessons file, and you must not create one.** No `lessons.md`, `learnings.md`,
  `gotchas.md`, `notes.md`, or "Lessons" section in a plan or tracker. A file that every session
  reads and every session appends to grows until it costs more than it saves.
- **When something surprised you, take the FIRST step below that applies. Then stop.**
  1. **Make the mistake fail.** Use a type, a lint rule, or a test. If you can write one, write it
     and write no prose.
  2. **Comment the site.** Put 1–3 lines on the exact function, config key or migration that bit
     you: the rule, then the reason. Whoever edits that code reads it. Nobody else pays for it.
  3. **Add a Common Gotcha to this file only if ALL of these hold.** Otherwise go to step 4.
     - It bites across many files, so no single site comment would be read in time.
     - It is true of the code today, and it changes what you would write.
     - It is specific to this repo, not general advice about programming or its frameworks.
     - It fits in one bullet of 3 lines or fewer, with no dates, ticket numbers, or story.
     - The user approved it. Name it in your handoff as a proposal; do not add it silently.
  4. **Write nothing.** This is the expected result for most tasks. "Nothing to record" is a
     complete answer.
- **Never record any of these, in any file:**
  - What happened during a task. That belongs in the commit message or the plan.
  - A general engineering principle.
  - Anything a test, type, this file, the README or a site comment already says.
  - A known, unfixed bug. That goes in the backlog.
- A user correction about *how to work with them* belongs in your memory, not in this repo.

### 4. Verification Before Done
- Never mark a task complete without proving it works
- Diff behavior between main and your changes when relevant
- Ask yourself: "Would a staff engineer approve this?"
- Run tests, check logs, demonstrate correctness

### 5. Demand Elegance (Balanced)
- For non-trivial changes: pause and ask "is there a more elegant way?"
- If a fix feels hacky: "Knowing everything I know now, implement the elegant solution"
- Skip this for simple, obvious fixes — don't over-engineer
- Challenge your own work before presenting it

### 6. Autonomous Bug Fixing
- When given a bug report: just fix it. Don't ask for hand-holding
- Point at logs, errors, failing tests — then resolve them
- Zero context switching required from the user
- Go fix failing CI tests without being told how

## Task Management

1. **Plan First**: Write plan to `todo.md` with checkable items
2. **Verify Plan**: Check in before starting implementation
3. **Track Progress**: Mark items complete as you go
4. **Explain Changes**: High-level summary at each step
5. **Document Results**: Add review section to `todo.md`

## Core Principles

- **Simplicity First**: Make every change as simple as possible. Impact minimal code.
- **No Laziness**: Find root causes. No temporary fixes. Senior developer standards.
- **Minimal Impact**: Changes should only touch what's necessary. Avoid introducing bugs.

---

Update this file when a command, convention or architecture fact in it becomes wrong. Do not
append history, status, or anything section 3 says not to record. **Keep this file under about
200 lines.** If the project has a test runner, add a check that fails when a lessons-style file
exists or this file exceeds its line budget. A prose rule alone has not held.

Always output a `testPlan.md` at the repo root for the manual tester, following `agent_docs/testPlan.md` exactly. It lists only what a human has to look at with their own eyes — no commands, no mechanism. Anything a command can verify (lint, typecheck, build, tests, probes) is your job before handoff; report it as one line in the review section of `todo.md`.

## Common Gotchas

<!-- Repo-specific tripwires only. Admission rules: Workflow Orchestration § 3. Empty is fine. -->
