# ADR-001: Entity Model

Date: 2019-08-12
Status: Accepted
Author: Engine Lead

## Context
We needed a way to represent game objects for the first title. The team came from
OOP-heavy backgrounds and wanted something familiar so we could move fast.

## Decision
Every entity derives from the engine's `Actor` base class. The framework already nests several
levels deep (`Actor` → `VStyleActor` → `Sprite`, plus `Button`, `TextField`, and friends), and
game behaviour is added by subclassing downward: our `Character`, then `Player` / `Enemy` /
`Pickup`, on top of `Actor`.

## Rationale
- Familiar to the whole team; no new paradigm to learn.
- Quick to write the first handful of entity types for the MVP.

## Consequences

**Positive**
- A new engineer can read the class tree and follow it right away.

**Negative**
- Deep hierarchies: new behaviour often forces an edit to a shared base class.
- Diamond inheritance has already shown up (a flying, damageable pickup).
- Object data is spread across the hierarchy, which hurts cache locality in the update loop.

**To watch**
- Revisit the update-loop cache behaviour once entity counts per scene grow.
