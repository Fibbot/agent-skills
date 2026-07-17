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
| `security-audit-pii` | PII lifecycle audit + production go/no-go gate (complements built-in `/security-review`) |
| `gdscript-standards` | Writing or reviewing Godot 4 GDScript |

## Subagents

| Agent | Role |
|---|---|
| `staff-engineer` | Reviews implementation plans before code is written; APPROVED / REQUEST CHANGES / REJECTED |
| `ux-reviewer` | Audits and implements visual-polish and interaction-design fixes |
| `perf-optimizer` | Measure-first performance audits across frontend/backend/infra |

## Conventions

- A **skill** is a procedure Claude triggers by task type (how to do X).
- A **subagent** is a persona reviewer that runs in a fresh context.
- Project templates (starter CLAUDE.md, lessons.md, testPlan.md) live inside `skills/project-bootstrap/templates/` and are instantiated by that skill.
