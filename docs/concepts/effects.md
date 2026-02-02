# Effects

**GameplayEffects** modify attributes, grant tags, and trigger game logic. They're the workhorses of PlayForge.

## Overview

An effect defines:

- **What** to modify (Impact Specification)
- **How long** (Duration Specification)  
- **What conditions** (Tag Requirements)
- **What tags** to grant (Granted Tags)

## Creating Effects

1. **Forge Manager > Create > Effect**
2. Configure specifications

```yaml
Name: "Poison"
Asset Tag: Effect.Debuff.Poison
Granted Tags: [Status.Poisoned]

Duration Policy: Durational
Duration: 5.0
Ticks: 2  # Ticks once every 2.5 seconds
TickOnApplication: true  # Ticks on application (naturally increased # of ticks by 1)

Attribute Target: Health
Impact Operation: Add
Magnitude: -10  # Per tick
```

## Duration Specification

Define how the effect behaves over time. There are 3 effect duration policies: ```Instant```, ```Durational```, and ```Infinite```.

| Policy | Behaviour                         |
|--------|-----------------------------------|
| `Instant` | Applied once, immediately removed |
| `Durational` | Lasts for specified duration      |
| `Infinite` | Lasts until manually removed      |

### Instant
```yaml
Duration Policy: Instant
# Impact applied once
```

### Durational
```yaml
Duration Policy: Durational
Duration: 10.0  # seconds
Ticks: 2  # number of ticks over entire duration
```

### Infinite
```yaml
Duration Policy: Infinite
Tick Interval: 2.0  # time between ticks
# Removed manually
```

## Durational Effect Settings

Derive durational settings from custom logic using scalers. When scalers are used, additional parameters are included to indicate how to use computed scaler values.

```yaml
Settings: Duration, DeltaTime, Ticks, Execute Ticks, Stack Amount
```

### Duration

```yaml
Duration: 10.0
Real Duration: AddScaler
Duration Scaler:
  - Type: SimpleScaler
    Level Values: [0.0, 2.5, 5.0]
```

```yaml
Level 1: 10.0 + 0   = 10.0
Level 2: 10.0 + 2.5 = 12.5
Level 3: 10.0 + 5.0 = 15.0
```

### DeltaTime

```yaml
Real DeltaTime: Multiply with Scaler
DeltaTime Scaler:
  - Type: SimpleScaler
    Level Values: [1.0, 1.2, 1.4]
```

```yaml
Level 1: dt * 1.0    = 10.0
Level 2: dt * 1.2    = 10.0
Level 3: dt * 1.4    = 10.0
```

## Impact Specification

### Target Impact

| Target | Modifies                               |
|--------|----------------------------------------|
| `Current` | Current value only (temporary)         |
| `Base` | Base value (permanent/retained impact) |
| `CurrentAndBase` | Both values                            |

### Impact Operations

| Operation | Formula |
|-----------|---------|
| `Add` | value + magnitude |
| `Multiply` | value × magnitude |
| `Override` | magnitude |

## Periodic Ticks

```yaml
Duration Policy: Durational
Duration: 10.0

Enable Periodic Ticks: true
Ticks: 5  # over 10.0s, ticks every 2.0s


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
