<!--
Sync Impact Report
==================
Version change: (unratified template) → 1.0.0
Bump rationale: Initial ratification; all placeholders replaced with concrete governance.

Modified principles:
  - [PRINCIPLE_1_NAME] → I. Readability First
  - [PRINCIPLE_2_NAME] → II. Data-Driven Configuration
  - [PRINCIPLE_3_NAME] → III. Modular Composition
  - [PRINCIPLE_4_NAME] → IV. Extensible Systems
  - [PRINCIPLE_5_NAME] → removed (user specified four priorities)

Added sections:
  - Technology & Project Constraints (was [SECTION_2_NAME])
  - Development Workflow & Quality Gates (was [SECTION_3_NAME])

Removed sections:
  - Fifth core principle slot

Follow-up TODOs:
  - TODO(PROJECT_NAME): no game title found in the repo; using "Godot Game" placeholder.
  - Assumption to confirm: Godot 4.x with GDScript as the primary language.
-->

# Godot Game Constitution

## Core Principles

### I. Readability First

Code is written for the next developer to read, not for the compiler. A new contributor MUST be
able to understand any script's purpose without asking its author.

- All GDScript MUST follow the official Godot GDScript style guide (snake_case functions and
  variables, PascalCase classes and nodes, CONSTANT_CASE constants).
- All variables, parameters, and function return values MUST use static typing
  (`var health: int`, `func take_damage(amount: int) -> void`).
- Every script that defines a reusable class MUST declare `class_name` and begin with a `##`
  doc comment stating its responsibility in one or two sentences.
- Public functions and signals MUST have `##` doc comments; private members are prefixed with `_`.
- Functions SHOULD stay under ~40 lines and scripts under ~400 lines; exceeding either requires
  splitting or a written justification in review.
- Names MUST describe intent (`attack_cooldown_seconds`, not `t` or `cd`). Magic numbers are
  prohibited outside of configuration resources (see Principle II).
- Each top-level folder MUST contain a short `README.md` explaining what lives there and how to
  add to it.

**Rationale**: Fast onboarding depends on code that explains itself. Static typing and doc
comments also enable editor autocompletion and in-editor documentation for newcomers.

### II. Data-Driven Configuration

Gameplay values and player-facing text live in data, never in logic. Designers MUST be able to
rebalance or rename content without editing a script.

- All tunable values (stats, costs, cooldowns, speeds, spawn rates, damage formulas'
  coefficients) MUST be defined in custom `Resource` classes (`.tres`) or `@export` properties,
  not as literals inside logic.
- Every unit, character, ability, and item MUST have a data resource (e.g. `UnitData`,
  `AbilityData`) that defines its stats and identity.
- All player-facing strings (names, descriptions, UI text) MUST go through Godot's localization
  system (`tr()` with translation keys); display names MUST NOT be used as identifiers.
- Content MUST be referenced by a stable string ID or resource path, so renaming a display name
  never breaks code, saves, or references.
- Global balance knobs (difficulty multipliers, economy rates) MUST live in a single, clearly
  named settings resource or autoload rather than scattered across scripts.
- Exported properties MUST use range/hint annotations (`@export_range`, `@export_enum`) and be
  grouped (`@export_group`) so they are safe and legible in the Inspector.

**Rationale**: Balancing is iterative. When values and names are data, tuning passes take
minutes in the Inspector instead of code changes, reviews, and risk of logic regressions.

### III. Modular Composition

Units and characters are assembled from reusable parts, not built as one-off scripts. Adding a
new unit SHOULD require creating data and a scene, with zero new code in the common case.

- Behavior MUST be expressed through composition: small, single-purpose component nodes
  (e.g. `HealthComponent`, `MovementComponent`, `HitboxComponent`, `AbilityComponent`) attached
  to entity scenes. Deep inheritance chains (more than two levels below a base entity) are
  prohibited.
- Each component MUST be self-contained: it owns its state, exposes a typed public API and
  signals, and MUST NOT reach into sibling or parent internals via hard-coded node paths.
- Components communicate with each other and with the rest of the game through signals or
  explicitly injected references (`@export var health: HealthComponent`), never via
  `get_node("../../...")` chains.
- New units and characters MUST be created as inherited scenes or scene templates of a shared
  base, configured by a data resource (Principle II).
- Each reusable scene MUST be runnable in isolation (F6) without crashing, so it can be tested
  and inspected on its own.

**Rationale**: Composition keeps each piece small and understandable, lets content creators mix
existing behaviors into new units, and confines bugs to the component that owns them.

### IV. Extensible Systems

New systems and features plug into the game; they do not require rewriting existing ones.
Existing code is open for extension and closed for modification wherever practical.

- Cross-system communication MUST use signals or a central event bus autoload; systems MUST NOT
  call into each other's internals directly.
- Autoloads (singletons) are reserved for genuinely global services (event bus, save/load,
  settings, audio). Each new autoload requires justification in review.
- Variable behaviors (abilities, AI states, status effects, win conditions) MUST be modeled as
  interchangeable strategy objects (e.g. `Resource` or `Node` subclasses with a shared
  interface), so new variants are added by creating a new class, not by extending
  `match`/`if` chains.
- Registries (units, abilities, items) MUST discover content from data folders or registration
  calls rather than hard-coded lists.
- Systems MUST define their public interface (signals, public methods, data resources) before
  implementation, and that interface MUST be documented in the system's folder README.
- Breaking a documented system interface requires updating all callers in the same change and
  noting it in the change description.

**Rationale**: Loose coupling and clearly defined extension points let features be added in
parallel by different contributors without regressions in unrelated systems.

## Technology & Project Constraints

- **Engine**: Godot 4.x (stable). Engine upgrades require a dedicated change with a migration note.
- **Language**: GDScript with static typing is the default. Other languages (C#, GDExtension)
  require written justification (e.g. measured performance need).
- **Project layout**: organize by feature/domain, not by file type. Recommended top level:
  `entities/` (units, characters), `components/`, `systems/`, `data/` (resources),
  `ui/`, `autoload/`, `assets/`, `localization/`, `tests/`.
- **Naming**: files and folders use `snake_case`; scene and script for the same entity share a
  name (`goblin_archer.tscn`, `goblin_archer.gd`).
- **Testing**: automated tests use a GDScript test framework (e.g. GUT or gdUnit4). Components,
  data validation, and system interfaces MUST have tests; pure visual/feel code MAY rely on
  manual playtesting.
- **Performance**: optimizations that reduce readability (Principle I) MUST be justified by
  profiler measurements and commented with the reason.

## Development Workflow & Quality Gates

- Every change is reviewed against the four Core Principles before merge. Reviewers MUST check:
  1. No hard-coded tunable values or display strings in logic (II).
  2. New units/characters use components and data resources, not bespoke scripts (III).
  3. New systems communicate via signals/event bus and document their interface (IV).
  4. Static typing, doc comments, and naming conventions are followed (I).
- The project MUST open in the Godot editor with no script errors or new warnings; warnings
  treated as errors where Godot project settings allow.
- Automated tests MUST pass before merge.
- Adding a new unit, character, or system for the first time of its kind MUST include or update
  a "how to add one" guide in the relevant folder README.
- Any deviation from a principle MUST be recorded in the change description with the reason and
  the simpler alternative that was rejected.

## Governance

This constitution supersedes informal conventions and personal preferences. Where it conflicts
with other project documents, the constitution wins until amended.

- **Amendments**: proposed as a change to `.specify/memory/constitution.md` with a rationale,
  reviewed and approved by the project maintainer(s), and accompanied by a migration plan if
  existing code becomes non-compliant.
- **Versioning**: semantic versioning applies.
  MAJOR for removing or redefining a principle; MINOR for adding a principle or section or
  materially expanding guidance; PATCH for clarifications and wording fixes.
- **Compliance**: all specs, plans, and reviews MUST verify compliance with the Core Principles.
  Added complexity MUST be justified against the simpler alternative. A periodic review (at least
  once per milestone) checks the codebase for drift and records follow-up tasks.

**Version**: 1.0.0 | **Ratified**: 2026-10-02 | **Last Amended**: 2026-10-02
