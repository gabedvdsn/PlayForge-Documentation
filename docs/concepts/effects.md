# Effects

**GameplayEffects** modify attributes, grant tags, and trigger game logic. They're the workhorses of PlayForge.

## Overview

An effect defines:

- **What** to modify (Impact Specification)
- **How long** (Duration Specification)  
- **What conditions** (Tag Requirements)
- **What tags** to grant (Granted Tags)

## Creating Effects

1. **Create > PlayForge > Effect**
2. Configure specifications

```yaml
Name: "Poison"
Asset Tag: Effect.Debuff.Poison
Granted Tags: [Status.Poisoned]

Duration Policy: Durational
Duration: 5.0
Tick Interval: 1.0

Attribute Target: Health
Impact Operation: Add
Magnitude: -10  # Per tick
```

## Duration Policies

| Policy | Behavior |
|--------|----------|
| `Instant` | Applied once, immediately removed |
| `Durational` | Lasts for specified duration |
| `Infinite` | Lasts until manually removed |

### Instant
```yaml
Duration Policy: Instant
# Impact applied once
```

### Durational
```yaml
Duration Policy: Durational
Duration: 10.0  # seconds
```

### Infinite
```yaml
Duration Policy: Infinite
# Removed manually or on unequip
```

## Impact Specification

### Target Impact

| Target | Modifies |
|--------|----------|
| `Current` | Current value only (temporary) |
| `Base` | Base value (permanent) |
| `CurrentAndBase` | Both values |

### Impact Operations

| Operation | Formula |
|-----------|---------|
| `Add` | value + magnitude |
| `Multiply` | value × magnitude |
| `Override` | magnitude |

## Periodic Ticks (DOT/HOT)

```yaml
Duration Policy: Durational
Duration: 10.0
Enable Periodic Ticks: true
Tick Interval: 2.0
Ticks: 5

Magnitude: -15  # Per tick
```

## Stacking

| Policy | Behavior |
|--------|----------|
| `ReplaceExisting` | Remove old, apply new |
| `RefreshDuration` | Reset timer |
| `StackExistingContainers` | Add stack |
| `NothingIfExists` | Ignore reapplication |

```yaml
Reapplication Policy: StackExistingContainers
Stack Amount: 5  # Max stacks
```

## Tag Requirements

```yaml
Source Requirements:
  Require Tags: [Status.Alive]
  Avoid Tags: [Status.Silenced]

Target Requirements:
  Avoid Tags: [Status.Invulnerable]
```

## Applying Effects

```csharp
// Generate spec with context
var spec = source.GenerateEffectSpec(origin, poisonEffect);

// Apply to target
target.ApplyGameplayEffect(spec);
```

## Removing Effects

```csharp
// By tag
target.AsGAS().RemoveEffectsByTag(Tags.Effect.Debuff);

// Specific effect
target.AsGAS().RemoveGameplayEffect(poisonEffect);
```

## Common Patterns

### Damage
```yaml
Duration Policy: Instant
Attribute Target: Health
Impact Operation: Add
Magnitude: -50
```

### Buff
```yaml
Duration Policy: Durational
Duration: 15.0
Attribute Target: Attack
Impact Operation: Multiply
Magnitude: 1.25  # +25%
```

### DOT
```yaml
Duration Policy: Durational
Duration: 10.0
Enable Periodic Ticks: true
Tick Interval: 1.0
Magnitude: -10
```
