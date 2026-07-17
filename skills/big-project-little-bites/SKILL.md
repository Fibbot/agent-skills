---
name: big-project-little-bites
description: Use when a spec or design is too large to build (or hold in context) in one session and must be split into a sequence of smaller units built across separate sessions weeks apart — decomposing into epics, roadmapping a multi-session build, one-epic-per-session with cold starts and handoff between them.
---

# Big Project, Little Bites

## Overview

Turn a large spec into a sequence of session-sized units, each built in its own fresh session, with a durable handoff so a cold start weeks later picks up cleanly.

**Core principle: the decomposition is the easy 80%. The value is in the seams.** A capable agent will already produce a sensible epic list, a foundation-first order, invariants, and a resume-doc without being told. What it will *not* do on its own is hunt for **second-order interactions** — the places two individually-correct features collide into a bug — and it finds far more of them by **fanning out one auditor per epic** than by planning solo. Spend your effort there, not on re-deriving the obvious structure.

## When to use

- A spec/design spanning many subsystems that won't fit one working session.
- Work that will run across many sessions with cold starts between them.
- Any time you're about to hand-write a long roadmap and a status doc.

**Not for:** a single feature that fits one session (use `superpowers:brainstorming` → `superpowers:writing-plans`), or work with no cross-session continuity.

This skill sits *between* `brainstorming` (decide what to build) and `writing-plans` (plan one unit). It produces the roadmap; each unit is still planned per-session with `writing-plans`.

## The method

### 1. Decompose — coherent units, then split to fit a session

Break into **epics** sized to how you think about the product. Split any epic too fat for one session into `NN_MM` sub-units (`04_01`, `04_02`) — the sub-unit is the session boundary, the epic is the concept.

**Epic numbers are stable IDs, not execution order.** They're referenced everywhere; renumbering breaks every reference. Keep a separate **Build order** section for what actually happens next. (Baselines reliably conflate these — the number becomes the order, then a reorder makes every ID a lie.)

### 2. Find the seam and build it first

Identify the **cross-cutting invariant** — the requirement that looks like one late feature but actually threads through every unit (permissions, tenancy, offline, encryption, "private items invisible everywhere"). Build its *seam* — the single choke-point that enforces it — in the **foundation epic**, with a no-op default, so later epics are written through it from day one. Retrofitting a cross-cutting invariant means auditing every prior unit for leaks, where each miss is a correctness/privacy bug, not an ordinary defect.

A good agent catches the seam on the *obvious* surface (e.g. read queries). It stops there. Which is why step 4 exists.

### 3. Write the handoff doc — the only artifact that spans sessions

One overview/index doc, read first and updated last every session. It must carry:

| Section | Why it survives a cold start |
|---|---|
| Status tracker (phase columns, not one status) | "planned but not built" is a real state |
| Epic one-liners + **build order** (separate from IDs) | what's next without re-reading everything |
| **Invariants** | rules no unit may break, stated once |
| **Decision log — ruling *and its reason*** | rulings without reasons get re-litigated by every cold session; the reason is the whole point |
| **Reference shelf** | facts expensive to rediscover (formats, API names, commands) |
| **Open questions**, numbered, answered inline with a date | product calls that outlive one session |

Plus a **runner prompt** with exactly one parameter (the unit name), so starting a session is: set param, paste. And **thick epic docs** — enough *why* to work from cold — but: **don't design implementation in epic docs.** Epic docs are *what* and *why*; the per-unit plan is *how*. Thick means more why, not more how. When reality diverges from a doc, fix the doc in the same session.

### 4. Audit the seams — the part that earns its keep

This is the move a solo planner skips. Once the epic list and a shared epic-doc template exist, **fan out one drafting agent per epic in parallel**, each given the same template, the invariants, and the decision log, and each told: *you are also a spec auditor — surface every gap, contradiction, and cross-feature interaction you find.*

Then hunt specifically for **second-order interactions** — not "does feature X work" but "what happens where X meets Y":

- dedup × visibility (importing something you already hold privately)
- per-item budgets/limits × items that can be private or shared
- aggregates/forecasts × private data leaking through a sum
- a global cap/count × a predicate that hides rows (hide rows → mint free capacity)
- any two features whose *individual* specs are fine but whose *intersection* is unspecified

Each is a real bug found before a line of code. Route every finding into the decision log (with a reason) or a numbered open question. **Run a coverage check**: enumerate every feature in the spec, assert each maps to exactly one unit, and name anything dropped or double-owned. (This catches the feature that silently belongs to no epic.)

## Common mistakes

- **Stopping at the decomposition.** The epic list is the cheap part; skipping the seam audit ships the interaction bugs to implementation.
- **Numbers as execution order.** Conflating stable ID with build order.
- **Decision log with rulings but no reasons.** The next cold session re-opens every one.
- **Designing in epic docs.** Implementation detail there drifts against real code immediately.
- **Auditing seams solo.** One planner finds a handful of interactions; one auditor per epic finds an order of magnitude more.
- **Planning files dumped into an existing repo.** Put the roadmap where it belongs, not inside whatever repo you happened to be in.

## Quick reference

```
plan/
  00_overview.md      # status · invariants · decision log (reasons!) · references · open Qs
  00_prompt.txt       # runner, one param = unit name
  NN_epic.md          # thick: what + why, not how. NN is a stable ID.
  NN_MM_subunit.md    # session-sized split of a fat epic
  testPlans/…         # per-unit, for a tester with no context
```

Order of operations: decompose → seam into foundation → overview + template → **fan-out audit + coverage check** → resolve findings into decision log / open questions → build one unit per session with `writing-plans`.
