# Effects

**GameplayEffects** modify attributes, grant tags, and trigger game logic. They're the workhorses of PlayForge.

## Overview

An effect defines:

- **What** to modify (Impact Specification)
- **How long** (Duration Specification)  
- **What conditions** (Tag Requirements)
- **What tags** to grant (Granted Tags)

!!! note "Level Provider Asset"
    GameplayEffect is a `Level Provider`. For more information, see [Forge/Level Providers](forge.md).

## Creating Effects

1. **Forge Manager > Create > Effect**
2. Configure specifications

```yaml
Name: "Poison"
Description: "Poison the enemy"
Texture: Poison_Icon
Asset Tag: Effect.Poison

Context Tags: [Context.Effect.Debuff]
Granted Tags: [Status.Poisoned]

# Rest of implementation...
```

## Impact Specification

Define the magnitude of impact to a specific attribute.

### Target Impact

What component of the attribute value does this effect target?

| Target | Modifies                               |
|--------|----------------------------------------|
| `Current` | Current value only (temporary)         |
| `Base` | Base value (permanent/retained impact) |
| `CurrentAndBase` | Both values                            |

### Impact Operations

What operation does this effect perform with respect to the current attribute value?

| Operation | Formula |
|-----------|---------|
| `Add` | value + magnitude |
| `Multiply` | value × magnitude |
| `Override` | magnitude |

### Affiliation

Define the source to target affiliation validation pipeline.

| Policy                 | Behaviour                                 |
|------------------------|-------------------------------------------|
| `Use Affiliation List` | Compare to list of valid affiliation tags |
| `Affiliated`           | Can impact only affiliated targets        |
| `Unaffiliated`         | Can impact only unaffiliated targets      |
| `Always Allow`         | Skip affiliation validation               |

When comparing affiliation, what level of affiliation-matching is required to validate?

| Affiliation Comparison | Behaviour                         |
|------------------------|-----------------------------------|
| `Any`                  | Any affiliation meets policy      |
| `All`                  | All affiliations must meet policy |

### Impact Behaviour

- `Impact Types`: What impact types does this effect use?
- `Reverse on Removal`: On effect removal, reverse total impact? (I.e. apply negated version of total tracked impact)

### Contained Effects

Chain effects using contained effects. Configure the contained effect to activate with respect to certain triggers.

| Activation Trigger | Behaviour                                           |
|--------------------|-----------------------------------------------------|
| `On Apply`         | Apply when effect is first applied                  |
| `On Tick`          | Apply when effect container's tick period completes |
| `On Remove`        | Apply when effect is removed                        |

## Duration Specification

Define how the effect behaves over time. There are 3 effect duration policies: ```Instant```, ```Durational```, and ```Infinite```.

| Policy | Behaviour                         |
|--------|-----------------------------------|
| `Instant` | Applied once, immediately removed |
| `Durational` | Lasts for specified duration      |
| `Infinite` | Lasts until manually removed      |

```yaml
Duration Policy: Instant # Effect applied and removed in the same frame
```

```yaml
Duration Policy: Durational
Duration: 10.0  # Seconds
Ticks: 2  # Number of ticks over entire duration
```

```yaml
Duration Policy: Infinite  # Must be removed manually
Tick Interval: 2.0  # Time between ticks
```

### Durational Effect Settings

Derive durational settings from custom logic using scalers. When scalers are used, additional parameters are included to indicate how to use computed scaler values.

```yaml
Settings: Duration, DeltaTime, Ticks, Additional Execute Ticks, Stack Amount
```

#### Duration

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

#### DeltaTime

```yaml
Real DeltaTime: Multiply with Scaler
DeltaTime Scaler:
  - Type: SimpleScaler
    Level Values: [1.0, 1.2, 1.4]
```

```yaml
Level 1: Time.deltaTime * 1.0
Level 2: Time.deltaTime * 1.2
Level 3: Time.deltaTime * 1.4
```

#### Ticks

```yaml
Ticks: 3
Real Ticks: AddScaler
Ticks Scaler:
  - Type: SimpleScaler
    Level Values: [0, 1, 2]
```

```yaml
Level 1: 3 + 0 = 3
Level 2: 3 + 1 = 4
Level 3: 3 + 2 = 5
```

#### Additional Execute Ticks

When a durational effect tick period is completed, apply the effect `GetExecuteTicks(GameplayEffectSpec spec, int ticks)` times to the target, where `ticks = 1` by default.

```yaml
Additional Execute Ticks: 0
Real Additional Execute Ticks: AddScaler
Additional Execute Ticks Scaler:
  - Type: SimpleScaler
    Level Values: [0, 0, 1]
```

```yaml
Level 1: 0 + 0 = 0
Level 2: 0 + 0 = 0
Level 3: 0 + 1 = 1
```

In the above example, at levels 1 and 2, execute ticks applied = `1 + 0 = 1`. At level 3, execute ticks applied = `1 + 1 = 2`.

#### Stack Amount

Only applicable when `ReApplicationPolicy = StackExistingContainers`. When an effect is stacked, how many stacks should be applied? 

```yaml
Stack Amount: 0
Real Stack Amount: UseScaler
Stack Amount Scaler:
  - Type: SimpleScaler
    Level Values: [1, 2, 4]
```

```yaml
Level 1: 1 = 1
Level 2: 2 = 2
Level 3: 4 = 4
```

### ReApplication Policy

Only applies for non-instant effects. What happens when the same effect is applied multiple times?

| Policy                      | Behavior                          |
|-----------------------------|-----------------------------------|
| `Do Nothing`                | Do not apply the effect           |
| `Replace Old Container`     | Replace old settings with new ones |
| `Append New Container`      | Create a new independent container |
| `Stack Existing Containers` | Stack effect container            |

### ReApplication Interaction

Only applies when `ReApplicationPolicy = StackExistingContainers`.

| Policy               | Behavior                     |
|----------------------|------------------------------|
| `Do Nothing`         | Only apply the effect        |
| `Refresh Containers` | Reset duration & tick period |
| `Extend Containers`  | Refresh & extend duration    |

### Stacking Behaviour

Effect containers can be stacked, modifying how they compute impact and durational settings.

| Stacking Policy                     | Behavior                                     |
|-------------------------------------|----------------------------------------------|
| `Stacks Share One Duration`         | All stacks share the same duration           |
| `Stacks Have Independent Durations` | Each stack tracks its own duration           |
| `Duration Taken From One Stack`     | Stack decreases by 1 when duration concludes |

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
// By effect
target.RemoveGameplayEffect(poisonEffect);

// By asset tag (only IGameplayAbilitySystem)
gas.RemoveGameplayEffect(poisonEffect.Tags.AssetTag);
target.AsGAS().RemoveGameplayEffect(poisonEffect.Tags.AssetTag);
```

## More Information
[API: Gameplay Effect](../api/effects/gameplay-effect.md)
[API: Gameplay Effect Instance](../api/effects/gameplay-effect-spec.md)