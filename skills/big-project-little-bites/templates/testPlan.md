# Template: Writing a `testPlan.md`

`testPlan.md` at the repo root is what a human tester runs after an epic lands. An agent finishing
an epic writes it without being asked (it is a handoff deliverable per `CLAUDE.md`). It replaces
the previous epic's plan.

## The one rule

**A test plan lists only what a human has to look at with their own eyes.**

Anything a command can verify is not a test item. Lint, typecheck, build, `npm run check`, curl,
psql, grep, and probe scripts are the author's own job during the build. Run them, fix what they
find, and report one line in the epic's status paragraph, which lives at the end of that epic's
plan file
("lint 0 errors, tsc clean, build 90/90, check 40/40, probes green"). A regression guard worth
keeping becomes a `*.check.ts` or a script under `scripts/`. It never goes in the plan.

## Shape

One `###` heading per user-visible change. Under it, a **Where** line and one checkbox.

```markdown
### Military Service gets a section-level N/A box (D47)
Where: `/provider/profile` as `+pro1` → Military Service tab
- [ ] The first control is a checkbox reading **Not applicable (no military service)**. Ticking it
      greys out the fields below and turns the tab green.
```

- **Where** names the role, the route, and the tab or panel. Nothing else.
- **The checkbox** says what the tester should see. One or two sentences. Present tense.
- **Bold** exact on-screen copy the tester must match. Backtick routes and logins.
- Reference the decision or finding in the heading (`D47`, `F25`) so the tester can look up why.

## Rules

- **One checkbox per change.** If a change spans several tabs that behave identically, one box
  that names the tabs. Do not write a box per tab.
- **No commands.** If verifying it needs a terminal, devtools, psql, or the Supabase dashboard,
  it does not belong here.
- **No mechanism.** Say what appears, not why it appears. A false "why" has shipped a missing fix
  before; a correct one is still noise to the tester.
- **No hex codes, class names, or pixel sizes.** "Turns green" is the assertion.
- **No "unchanged", "carried forward", or "out of scope" sections.** The plan is this epic's
  changes only. Prior epics' boxes are gone when the new plan lands.
- **No re-listing prior epics' passes.** A tick from a previous run certifies nothing here.
- **Role check only when the epic changed who sees what.** One box per role that must NOT see
  it. Skip the section when authz did not move.
- **Setup is a short block at the top**, only if the seed does not already give the tester the
  state they need. Logins and the reset command live in `README.md`; link, do not repeat.
- **Soft cap: about 10 boxes.** Past that, merge boxes rather than adding steps. An epic that
  genuinely needs more is telling you it should have been two epics.
- **Accepted-behaviour notes:** at most one `> **NOTE**` blockquote, only if a tester would
  otherwise file it as a bug.

## Sample

```markdown
# testPlan.md

## Epic NN — <Title>

Closes **F<nn>**, **F<nn>**. Branch `<branch>`.

Setup: <only if the seed is not enough; otherwise delete this line>. Logins are in `README.md`.

### <Change> (<D or F ref>)
Where: `<route>` as `<login>` → <tab>
- [ ] <What you should see.>

### <Change> (<D or F ref>)
Where: `<route>` as `<login>` → <tab>
- [ ] <What you should see.>

### Role check
- [ ] As `<role>`, <surface> is absent / returns the not-found page.

> **NOTE — accepted, do not file.** <one known behaviour, if any>
```

## Author's self-check before handoff

- [ ] Every user-visible change the epic made has a box. Every box is a user-visible change.
- [ ] No box needs a terminal, devtools, or the dashboard.
- [ ] No box explains why something works.
- [ ] Fewer than about 10 boxes, or a stated reason why not.
- [ ] The machine verification roll-up is one line in the epic's status paragraph, not in this file.

