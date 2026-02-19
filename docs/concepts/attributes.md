# Attributes

**Attributes** are numeric values that define entity stats like Health, Mana, Attack, and Defense.

## Overview

Every attribute has two values:

| Value | Description |
|-------|-------------|
| **Base Value** | The underlying value, modified by "Base" effects |
| **Current Value** | The active gameplay value, derived from Base + modifiers |

```csharp
var attrSystem = IGameplayAbilitySystem.GetAttributeSystem();

// Get values
bool hasHealth = attrSystem.TryGetAttributeValue(healthAttribute, out AttributeValue health);
float baseHealth = health.BaseValue, currHealth = health.CurrentValue;

// Modify (recommended to do via Gameplay Effects instead of direct)
attrSystem.ModifyAttribute(healthAttribute, SourcedModifiedAttributeValue, runEvents = true);
```

## Creating Attributes

1. **Forge Manager > Create > Attribute**
2. Configure:

```yaml
Name: "Health"
Short Name: "HP"
Texture: health_icon.png
```

Attributes are created separately from their parameterization, which is handled in[Attribute Set](attribute-set.md) declarations.

## Attribute Sets

Define which attributes an entity has:

```yaml
Attribute Set:
  - Attribute: Health
    Base Value: 100
  - Attribute: Attack
    Base Value: 10
```

!!! note "Attribute Parameterization"
    Above demonstrates a limited view of attribute parameterization, with many options being omitted. Please see [Attribute Set](attribute-set.md) for more information on attribute bounds, constraints, and scaler application.

## Bounds

Bound attribute values:

```yaml
Attribute: Health
Constraints:
  AutoClamp: true
  AutoScaleWithBase: false  # Changes to base are propagated proportionally to current value
  RoundingMode: None  # Floor, Ceil, Round, SnapTo
  SnapInterval:
```

## Constraints

Bound attribute values:

```yaml
Attribute: Health
Constraints:
  AutoClamp: true
  AutoScaleWithBase: false  # Changes to base are propagated proportionally to current value
  RoundingMode: None  # Floor, Ceil, Round, SnapTo
  SnapInterval:
```


## Implicit Attribute Scaling

Configure cached attribute scalers in your Attribute Set to define how an attributes declared value changes with respect to changes in the system. 

In this example, health implicit value is ```100 + (Strength * 8)```.

```yaml
Attribute: Health
Magnitude: 100
Real Magnitude: Add Scaler
Level Scaler:
  Type: Cached Attribute-backed Scaler
  Magnitude: 8
  Capture Attribute: Strength
  Operation: Multiply
```

## Modifying Attributes

### Via Effects (Recommended)

```csharp
var spec = source.GenerateEffectSpec(origin, damageEffect);
target.ApplyGameplayEffect(spec);
```

### Direct Modification

Attribute modification requires a ```SourcedModifiedAttributeValue``` instance.

```csharp
attrSystem.ModifyAttribute(health, SourcedModifiedAttributeValue, runEvents: true);  // Take damage
```

!!! warning "Prefer Effects"
    Direct modification bypasses certain callbacks and tracking. Use effects for gameplay interactions.

## Querying Attributes

```csharp
var attrSystem = gas.GetAttributeSystem();

if (attrSystem.TryGetAttributeValue(health, out AttributeValue value) ...;  // Get value
if (attrSystem.TryGetAttributeValue(health, out CachedAttributeValue cachedValue) ...;  // Get cached value
```

## Listening to Changes

```csharp
IGameplayAbilitySystem gas;

gas.GetAttributeSystem().Callbacks.OnAttributeChanged += (impactData) =>
{
    if (impactData.Attribute == healthAttribute && impactData.LiveValue.CurrentValue <= 0)
        gas.MarkDead();
};
```

Or, using the action queue:

```csharp
gas.GetAttributeSystem().Callbacks.OnAttributeChanged += (impactData) =>
{
    if (impactData.Attribute == healthAttribute && impactData.LiveValue.CurrentValue <= 0)
        gas.GetActionQueue().Enqueue(new DeathAction(gas));
};
```

!!! note "Action Queue"
    The action queue approach defers the `MarkDead` call to the end of the frame. For more information, see [Action Queue](action-queue.md).

## Cached Attribute Values

Values for a particular attribute are cached and organized with respect to active or retained impact derivations. Instant effect impact is never retained unless it targets the base value.

| Derivation                | Value   | Description             |
|---------------------------|---------|-------------------------|
| **Level 10 Talent**       | +0/+50  | Provides +50 base value |
| **Gauntlets of Strength** | +0/+90  | Provides +90 base value |
| **Mithril Dagger**        | +25/+25 | Provides +90 base value |

