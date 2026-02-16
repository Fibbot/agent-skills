# agent_game_systems.md

## Purpose
Define and enforce the implementation of all core game systems: the deployment loop, extraction mechanics, EXP calculations, stat progression, class/breakstat trees, and crafting. All game math must be deterministic and server-authoritative.

## Scope

## Output Requirements
Any system implementation must include:
1. **Formula documentation** — Every calculation expressed as a clear mathematical formula in comments.
2. **Deterministic guarantee** — Given identical inputs, the system produces identical outputs. No RNG without seeded PRNG.
3. **Server parity** — Client-side preview calculations must match server-side authoritative calculations exactly.
4. **Edge case coverage** — Boundary conditions documented and handled (zero HP, max level, overflow).

---

## Phase 1: Deployment & Run Execution



## Phase 2: Character Progression

### 2.1 Stat Allocation


### 2.2 Primary Stats


---

## Phase 3: Itemization & Crafting

### 3.1 Gear Tiering (Strictly Vertical)

### 3.3 Crafting Matrix

### 3.4 End-Game Gear Sets


## Phase 4: Leaderboard Metrics

### 4.1 Tracked Metrics

### 4.2 Leaderboard Categories

### 4.3 Reset Cadence


## Implementation Rules

### Determinism

### Balance Levers

### Validation

