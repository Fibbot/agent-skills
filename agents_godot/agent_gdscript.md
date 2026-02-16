# agent_gdscript.md

## Purpose
Enforce GDScript coding standards, Godot 4.x API conventions, and architectural patterns across the entire client codebase. Every script written or modified must conform to these rules.

## Scope
- GDScript style and naming conventions
- Godot 4.x node architecture and scene composition
- Signal-driven communication patterns
- Resource and autoload management
- Performance patterns for mobile hardware targets

---

## GDScript Style Rules

### Naming Conventions
- **Classes/Nodes:** `PascalCase` — `DeploymentManager`, `RunVisualizer`
- **Functions/Variables:** `snake_case` — `calculate_exp_multiplier()`, `current_depth`
- **Constants:** `UPPER_SNAKE_CASE` — `MAX_AGGRESSOR_LEVEL`, `BASE_EXTRACTION_TIME`
- **Signals:** `snake_case`, past tense — `run_completed`, `extraction_triggered`, `hp_depleted`
- **Enums:** `PascalCase` type, `UPPER_SNAKE_CASE` values:
```gdscript
enum BreakstatTier { TIER_1, TIER_2, TIER_3 }
enum BaseClass { WARRIOR, ROGUE, MAGE, POET }
```
- **Private members:** Prefix with `_` — `_internal_timer`, `_calculate_burst_window()`
- **File names:** `snake_case.gd` matching the node/class — `deployment_config.gd`

### Type Hints (Mandatory)
- All function signatures must include parameter types and return types.
- All member variables must be typed.
- Use `-> void` explicitly for functions with no return value.
```gdscript
var current_hp: float = 100.0
var aggressor_level: int = 0

func calculate_velocity(depth: int, aggressor: int) -> float:
    return depth * (1.0 + aggressor * 0.15)

func _on_extraction_triggered() -> void:
    pass
```

### Documentation
- Every public function gets a `##` doc comment describing purpose, parameters, and return value.
- Every exported variable gets a `##` doc comment.
- Complex math (EXP curves, EHP calculations) must include the formula in comments.

---

## Architecture Patterns

### Scene Composition
- Prefer composition over inheritance. Use child nodes with focused scripts.
- Scenes should be self-contained and testable in isolation.
- Never reference parent nodes directly — use signals to communicate upward.
- Use `@export` for inter-scene dependencies injected from the editor.

### Signal Architecture
- **Rule:** Signals go UP, calls go DOWN.
  - Parent calls child methods directly.
  - Children emit signals; parents connect to them.
- **Cross-system communication:** Route through an autoload EventBus.
```gdscript
# event_bus.gd (Autoload)
signal run_started(loadout: Dictionary)
signal run_completed(result: RunResult)
signal extraction_phase_entered(hp_remaining: float)
signal player_died()
signal exp_awarded(amount: int)
signal leaderboard_updated()
```
- Never connect signals in `_ready()` to nodes that may not exist yet. Use deferred connections or autoloads.

### Autoloads (Singletons)
Keep autoloads minimal and focused:
- `EventBus` — Global signal routing only. No state.
- `GameState` — Current session state: active character, loadout, run status.
- `PlayerData` — Persistent player data: roster, inventory, stats. Interfaces with save/load and server sync.
- `ServerAPI` — All HTTP communication with backend. Returns typed results via signals/await.
- `AudioManager` — BGM/SFX control (stubbed for MVP).

### Resource Pattern
Use custom `Resource` classes for static data definitions:
```gdscript
# res/zone_data.gd
class_name ZoneData
extends Resource

@export var zone_id: String
@export var zone_name: String
@export var enemy_archetypes: Array[EnemyArchetype]
@export var gathering_bias: Dictionary  # {profession: weight}
@export var wave_milestones: Array[int]  # [50, 100, 200]
@export var exp_multipliers: Array[float]  # corresponding multipliers
```

---

## Godot 4.x Specific Rules

### Async Patterns
- Use `await` for HTTP requests, tween completions, and timer waits.
- Never use `yield()` (Godot 3 pattern).
- Coroutines that await must return the awaited type or `void`.

### Input Handling
- All input goes through `_unhandled_input()` or the InputMap.
- Define all actions in Project Settings > Input Map.
- Touch and mouse inputs must be interchangeable (portrait-locked design).
- Never use `_input()` in gameplay nodes — reserve for UI overlay captures.

### Node Lifecycle
- `_ready()`: Initialize, connect signals, set initial state.
- `_enter_tree()` / `_exit_tree()`: For dynamic nodes only.
- `_process()`: Only when continuous per-frame updates are needed. Prefer timers and signals.
- `_physics_process()`: Only for movement/collision in the run visualizer.

### Performance Rules
- Avoid `get_node()` with string paths in loops. Cache references in `_ready()`.
- Use `@onready` for node references:
```gdscript
@onready var hp_bar: ProgressBar = $UI/HPBar
@onready var wave_counter: Label = $UI/WaveCounter
```
- Object pooling for enemy sprites in the run visualizer — do not instantiate/free per wave.
- Minimize `_process()` usage. If a node doesn't need per-frame updates, don't define it.
- Use `set_process(false)` / `set_physics_process(false)` when nodes are inactive.

---

## Error Handling
- All server API calls must handle failure (timeout, 4xx, 5xx).
- Use typed return values or error signals — never silently fail.
- Log errors with `push_error()` for debugging; `push_warning()` for non-critical issues.
- Never use `assert()` in production paths — only in debug/test builds.

## File Organization
```
project/
├── autoloads/          # EventBus, GameState, PlayerData, ServerAPI
├── scenes/
│   ├── ui/             # All UI screens (title, roster, hub, deployment, leaderboard)
│   ├── run/            # Run visualizer scene and components
│   └── shared/         # Reusable UI components (buttons, panels, dialogs)
├── scripts/
│   ├── models/         # Data classes (RunResult, CharacterStats, Loadout)
│   ├── systems/        # Game logic (ExpCalculator, ExtractionResolver, CraftingEngine)
│   └── util/           # Math helpers, formatters, validators
├── resources/          # .tres files (ZoneData, EnemyArchetype, GearDefinition)
└── tests/              # GdUnit4 test scripts
```
