---
name: project-bootstrap
description: Bootstrap a new or existing repo with the standard working setup - starter CLAUDE.md and agent_docs/testPlan.md template. Use when starting a project, when asked to "set up CLAUDE.md", "port my usual setup", or when a repo has no CLAUDE.md.
---

# Project Bootstrap

Instantiate the standard project scaffolding so every repo starts with the same workflow: plan-first, subagent-heavy, verified-before-done.

## Files to create (from `templates/` in this skill)

1. `CLAUDE.md` (repo root) — from `templates/CLAUDE.md`
2. `agent_docs/testPlan.md` — from `templates/testPlan.md`

**Never create a `lessons.md`** or any other file that sessions read at start and append to. That
pattern was tried and grew to ~1,000 lines before it was retired. The template's Workflow
Orchestration § 3 says where a surprise goes instead.

If any already exist, merge — never overwrite. Preserve existing project-specific content and add only the missing sections.

## Procedure

1. **Explore first.** Read `package.json`/`pyproject.toml`/`project.godot`/etc., entry points, and directory structure before writing anything.
2. **Copy templates** into place.
3. **Fill the top half of CLAUDE.md** (Project Overview, Tech Stack, Core Architecture, Development Commands) with what exploration actually established. Leave sections you can't confirm as headers with a `TODO` note — do not fabricate. Delete the `FILL IN THE ABOVE...` marker once filled.
4. **Leave the bottom half (Workflow Orchestration onward) verbatim.** It is workflow policy, not project description.
5. Confirm the CLAUDE.md reference to the test plan template points at `agent_docs/testPlan.md`.

## Notes

- If the repo already has a `lessons.md` (or similar), do not carry it forward. Propose triaging it:
  entries a test can enforce become tests, entries tied to one site become comments there,
  2–10 cross-cutting tripwires become Common Gotchas, and the rest is archived or deleted.
- If the project has a test runner, add the budget check the template describes.
- If the project has an existing CLAUDE.md with real content, propose a diff instead of replacing it.
