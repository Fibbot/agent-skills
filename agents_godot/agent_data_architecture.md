# agent_data_architecture.md

## Purpose
Define all data models, database schemas, serialization formats, and data flow patterns for the game. Ensure data integrity between client state, server persistence, and cross-platform synchronization.

## Scope
- Database schema design (relational + cache layers)
- Client-side data models (GDScript Resource/class definitions)
- Server-side data models and validation schemas
- Inventory system data flow
- Save/load and sync patterns
- Data migration strategy

## Output Requirements
1. **Schema definitions** — Every table/collection with types, constraints, indexes, and relationships.
2. **Client model parity** — Every server schema maps to a typed GDScript class.
3. **Validation rules** — Constraints enforced at both DB level and application level.
4. **Migration plan** — Schema changes must be versioned and backwards-compatible.

---

## Phase 1: Core Data Models

### 1.1 Account
```sql
accounts (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    last_login      TIMESTAMPTZ NOT NULL DEFAULT now(),
    premium         BOOLEAN NOT NULL DEFAULT false,
    premium_since   TIMESTAMPTZ,
    banned          BOOLEAN NOT NULL DEFAULT false,
    ban_reason      TEXT
)

platform_links (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id      UUID NOT NULL REFERENCES accounts(id),
    platform        TEXT NOT NULL CHECK (platform IN ('apple', 'google', 'steam')),
    platform_uid    TEXT NOT NULL,
    linked_at       TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE (platform, platform_uid),
    UNIQUE (account_id, platform)
)
```

### 1.2 Character
```sql

```

### 1.3 Inventory
```sql

```
---

## Phase 2: Static Data Definitions

### 2.1 Storage Format
- Static game data (zones, enemies, items, crafting recipes) stored as Godot `.tres` Resource files on the client.
- Server loads identical data from JSON/YAML config files at startup.
- **Version parity:** Client and server must reference the same data version. Include a `data_version` field in API handshake.

### 2.2 Key Static Definitions
```
EnemyArchetype:
    archetype_id, name, hp, damage

ItemDefinition:
    item_def_id, name, slot, tier 
```

---

## Phase 3: Client Data Layer

### 3.1 GDScript Data Classes


### 3.2 Client Cache Strategy


### 3.3 Serialization Rules


---

## Phase 4: Data Integrity Rules

### 4.1 Invariants (Enforced at DB + Application Level)


### 4.2 Transaction Boundaries


### 4.3 Migration Strategy


---

## Implementation Rules

### Indexes


### Naming Conventions


### Security
