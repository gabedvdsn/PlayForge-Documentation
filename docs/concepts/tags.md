# Tags

**Tags** are hierarchical identifiers used throughout PlayForge for categorization, requirements, and conditional logic.

## Overview

Tags serve multiple purposes:

| Purpose | Example |
|---------|---------|
| **Identity** | `Ability.Fireball`, `Entity.Player` |
| **Categorization** | `Element.Fire`, `DamageType.Physical` |
| **Status Tracking** | `Status.Burning`, `Status.Stunned` |
| **Requirements** | Must have X, must not have Y |

## Tag Structure

Tags use **hierarchical dot notation**:

```
Category.Subcategory.Specific
```

Examples:
```
Ability.Active.Attack
Effect.Debuff.Poison
Status.CrowdControl.Stun
Entity.Enemy.Boss
```

## Hierarchy Benefits

Child tags match parent queries:

```
Status.Stunned      → matches Status.*
Effect.Damage.Fire  → matches Effect.Damage.* and Effect.*
```

This enables broad requirements ("any damage") or specific ones ("fire damage only").

## Tag Types

### Asset Tag
Unique identifier for an asset:
```yaml
Asset Tag: Ability.Fireball
```

### Context Tags
Additional categorization (multiple allowed):
```yaml
Context Tags: [Ability.Type.Spell, Element.Fire]
```

### Granted Tags
Applied while effect/ability is active:
```yaml
Granted Tags: [Status.Burning, Status.OnFire]
```

## Tag Requirements

Control when effects/abilities can be applied:

```yaml
Source Requirements:
  Require Tags: [Status.Alive]      # Must have ALL
  Avoid Tags: [Status.Silenced]     # Must have NONE

Target Requirements:
  Require Tags: [Entity.Enemy]
  Avoid Tags: [Status.Invulnerable]
```

### Validation Logic

```
Can Apply = 
    Source has ALL Required Tags
    AND Source has NONE of Avoided Tags
    AND Target has ALL Required Tags
    AND Target has NONE of Avoided Tags
```

## Querying Tags

```csharp
ITarget target = ...;

// Get all applied tags
List<Tag> tags = target.GetAppliedTags();

// Check for specific tag
bool isBurning = tags.Contains(Tags.Status.Burning);

// Hierarchical check
bool hasStatus = Tags.Status.Contains(someTag);
```

## Recommended Hierarchy

```
Status
├── Alive
├── Dead
├── CrowdControl
│   ├── Stun
│   ├── Root
│   └── Silence
└── DoT
    ├── Burning
    └── Poisoned

Entity
├── Player
├── Enemy
│   ├── Minion
│   └── Boss
└── NPC

Effect
├── Damage
│   ├── Physical
│   └── Fire
├── Buff
└── Debuff
```

## Best Practices

| Do | Don't |
|----|-------|
| `Status.Burning` | `Burning` |
| `Effect.Buff.Speed` | `SpeedBuff` |
| Keep depth 3-4 levels | 5+ level hierarchies |
