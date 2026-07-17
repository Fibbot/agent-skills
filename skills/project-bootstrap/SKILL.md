---
name: project-bootstrap
description: Bootstrap a new or existing repo with the standard working setup - starter CLAUDE.md, lessons.md memory file, and agent_docs/testPlan.md template. Use when starting a project, when asked to "set up CLAUDE.md", "port my usual setup", or when a repo has no CLAUDE.md.
---

# Project Bootstrap

Instantiate the standard project scaffolding so every repo starts with the same workflow: plan-first, subagent-heavy, lessons-driven, verified-before-done.

## Files to create (from `templates/` in this skill)

1. `CLAUDE.md` (repo root) — from `templates/CLAUDE.md`
2. `lessons.md` (repo root) — from `templates/lessons.md`
3. `agent_docs/testPlan.md` — from `templates/testPlan.md`

If any already exist, merge — never overwrite. Preserve existing project-specific content and add only the missing sections.

## Procedure

1. **Explore first.** Read `package.json`/`pyproject.toml`/`project.godot`/etc., entry points, and directory structure before writing anything.
2. **Copy templates** into place.
3. **Fill the top half of CLAUDE.md** (Project Overview, Tech Stack, Core Architecture, Development Commands) with what exploration actually established. Leave sections you can't confirm as headers with a `TODO` note — do not fabricate. Delete the `FILL IN THE ABOVE...` marker once filled.
4. **Leave the bottom half (Workflow Orchestration onward) verbatim.** It is workflow policy, not project description.
5. Confirm the CLAUDE.md reference to the test plan template points at `agent_docs/testPlan.md`.

## Notes

- `lessons.md` starts nearly empty by design — it accrues corrections over the life of the project.
- If the project has an existing CLAUDE.md with real content, propose a diff instead of replacing it.
