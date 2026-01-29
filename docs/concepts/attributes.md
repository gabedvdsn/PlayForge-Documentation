# Attributes

**Attributes** are numeric values that define entity stats like Health, Mana, Attack, and Defense.

## Overview

Every attribute has two values:

| Value | Description |
|-------|-------------|
| **Base Value** | The underlying value, modified by "Base" effects |
| **Current Value** | The active gameplay value, derived from Base + modifiers |

```csharp
var attrSystem = entity.GetComponent<AttributeSystemComponent>();

// Get values
float health = attrSystem.GetCurrentValue(healthAttribute);
float baseHealth = attrSystem.GetBaseValue(healthAttribute);

// Modify
attrSystem.SetBaseValue(healthAttribute, 150f);
```

## Creating Attributes

1. **Create > PlayForge > Attribute**
2. Configure:

```yaml
Name: "Health"
Short Name: "HP"
Icon: health_icon.png

# Constraints
Minimum: 0
Maximum Source: HealthMax  # Dynamic cap
```

## Attribute Sets

Define which attributes an entity has:

```yaml
Attribute Set:
  - Attribute: Health
    Base Value: 100
  - Attribute: HealthMax
    Base Value: 100
  - Attribute: Attack
    Base Value: 10
```

## Derivations

Automatically calculate values from other attributes:

| Type | Formula | Use Case |
|------|---------|----------|
| `AddDerivation` | A + B | Flat bonuses |
| `MultiplyDerivation` | A × B | Percentages |
| `BaseMultiplierDerivation` | Base × (1 + B) | Level scaling |

### Example: Health Scaling

```yaml
Attribute: HealthMax
Derivations:
  - Type: BaseMultiplierDerivation
    Source: Level
    Multiplier: 10  # +10 max health per level
```

## Constraints

Bound attribute values:

```yaml
Attribute: Health
Constraints:
  Minimum: 0
  Maximum Source: HealthMax
  Rounding: RoundToInt
```

## Modifying Attributes

### Via Effects (Recommended)

```csharp
var spec = source.GenerateEffectSpec(origin, damageEffect);
target.ApplyGameplayEffect(spec);
```

### Direct Modification

```csharp
attrSystem.ModifyCurrentValue(health, -10f);  // Take damage
attrSystem.SetBaseValue(health, 100f);        // Set directly
```

!!! warning "Prefer Effects"
    Direct modification bypasses callbacks and tracking. Use effects for gameplay interactions.

## Listening to Changes

```csharp
attrSystem.OnAttributeChanged += (attr, oldVal, newVal) =>
{
    if (attr == healthAttribute && newVal <= 0)
        HandleDeath();
};
```

## Level Scaling

Use scalers for level-based values:

```yaml
Base Value: 100
Level Scaler:
  Type: LinearScaler
  Values: [100, 120, 145, 175, 210]  # Levels 1-5
```
