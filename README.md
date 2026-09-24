# agent-skills

Reusable Claude Code skills, subagent definitions, and project scaffolding.

## Layout

```
skills/     # one dir per skill, SKILL.md + bundled resources
agents/     # subagent definitions (.claude/agents format)
```

## Install

```sh
# skills (per-user)
cp -r skills/<name> ~/.claude/skills/

# subagents (per-user, or drop into a repo's .claude/agents/)
cp agents/<name>.md ~/.claude/agents/
```

## Skills

| Skill | Trigger |
|---|---|
| `big-project-little-bites` | Spec too large for one session — decompose into epics with handoff docs and seam audits |
| `project-bootstrap` | New/existing repo needs the standard CLAUDE.md + lessons.md + testPlan setup |
| `tailwind-mobile-audit` | Mobile-first / touch / iOS audit of a Tailwind codebase |
| `security-audit-pii` | Whole-codebase PII lifecycle audit + right-sized go/no-go gate |
| `authz-matrix-audit` | Multi-role/tenant apps: expected vs probed authz matrix, local-only rollback-safe probes |
| `gdscript-standards` | Writing or reviewing Godot 4 GDScript |

## Subagents

| Agent | Role |
|---|---|
| `staff-engineer` | On-demand architecture review of a plan; APPROVED / REQUEST CHANGES / REJECTED. Not a default gate |
| `ux-reviewer` | Report-only audit: visual polish, interaction design, basic a11y → change plan |
| `perf-optimizer` | Report-only, measure-first performance audit → ranked change plan |

## Conventions

- A **skill** is a procedure Claude triggers by task type (how to do X).
- A **subagent** is a persona reviewer that runs in a fresh context. Subagents report; they don't edit files. The main session implements what the user approves.
- Project templates (starter CLAUDE.md, lessons.md, testPlan.md) live inside `skills/project-bootstrap/templates/` and are instantiated by that skill.
