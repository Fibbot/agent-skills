---
name: gdscript-standards
description: GDScript style, Godot 4.x conventions, and client architecture patterns (signals-up/calls-down, autoload discipline, Resource-based static data, mobile performance). Use when writing or reviewing any GDScript / Godot 4 code.
---

# GDScript & Godot 4 Standards

Every script written or modified must conform to these rules.

## Style Rules

### Naming
- **Classes/Nodes:** `PascalCase` — `DeploymentManager`, `RunVisualizer`
- **Functions/Variables:** `snake_case` — `calculate_exp_multiplier()`, `current_depth`
- **Constants:** `UPPER_SNAKE_CASE` — `MAX_LEVEL`, `BASE_EXTRACTION_TIME`
- **Signals:** `snake_case`, past tense — `run_completed`, `hp_depleted`
- **Enums:** `PascalCase` type, `UPPER_SNAKE_CASE` values
- **Private members:** `_` prefix — `_internal_timer`, `_calculate_burst_window()`
- **Files:** `snake_case.gd` matching the node/class

### Type hints (mandatory)
- All function signatures typed (params + return); `-> void` explicit.
- All member variables typed.

```gdscript
var current_hp: float = 100.0

func calculate_velocity(depth: int, modifier: int) -> float:
    return depth * (1.0 + modifier * 0.15)
```

### Documentation
- `##` doc comments on every public function and exported variable.
- Complex math must include the formula in comments.

## Architecture Patterns

### Scene composition
- Composition over inheritance; child nodes with focused scripts.
- Scenes self-contained and testable in isolation.
- Never reference parent nodes directly — signals communicate upward.
- `@export` for inter-scene dependencies injected from the editor.

### Signals
- **Rule: signals go UP, calls go DOWN.** Parents call child methods; children emit signals.
- Cross-system communication routes through an autoload `EventBus`.
- Never connect signals in `_ready()` to nodes that may not exist yet — use deferred connections or autoloads.

### Autoloads (keep minimal and focused)
- `EventBus` — global signal routing only, no state.
- `GameState` — current session state.
- `PlayerData` — persistent data; interfaces with save/load and server sync.
- `ServerAPI` — all HTTP; returns typed results via signals/await.

### Resources for static data
Custom `Resource` classes with `@export`ed typed fields for all static definitions (zones, items, archetypes) stored as `.tres`.

## Godot 4.x Specifics

- `await`, never `yield()` (Godot 3).
- Input via `_unhandled_input()` or the InputMap; define actions in Project Settings. Touch and mouse interchangeable. Reserve `_input()` for UI overlay captures.
- `_ready()`: init, connect signals. `_process()` only for genuine per-frame needs — prefer timers and signals. `_physics_process()` only for movement/collision.

## Performance (mobile targets)

- Never `get_node()` with string paths in loops — cache with `@onready`.
- Object pooling for frequently spawned entities; no instantiate/free per wave.
- `set_process(false)` / `set_physics_process(false)` on inactive nodes.

## Error Handling

- Every server API call handles timeout/4xx/5xx. Typed returns or error signals — never silent failure.
- `push_error()` / `push_warning()` for logging. No `assert()` in production paths.

## File Organization

```
project/
├── autoloads/          # EventBus, GameState, PlayerData, ServerAPI
├── scenes/
│   ├── ui/             # screens
│   ├── run/            # gameplay scenes
│   └── shared/         # reusable components
├── scripts/
│   ├── models/         # data classes
│   ├── systems/        # game logic
│   └── util/           # helpers
├── resources/          # .tres static data
└── tests/              # GdUnit4
```
