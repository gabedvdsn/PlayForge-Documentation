# Attribute Sets

**Attribute Sets** group related attributes with base values, constraints, and level scaling. They define an entity's stat profile.

## Overview

An Attribute Set includes:

- **Attributes** — Which stats the entity has
- **Base Values** — Starting values per attribute
- **Scaling** — Lowest-level value modifications
- **Overflow** — Upper and lower bounds
- **Constraints** — Auto settings and rounding rules
- **SubSets** — Nested attribute collections

## Creating Attribute Sets

1. **Forge Manager > Create > AttributeSet**
2. Configure attributes:

## Attribute Entries

Each entry defines how an attribute behaves for entity's using this attribute set:

```yaml
Attribute Entry:
  Attribute: Health
  Magnitude: 100
  Target: CurrentAndBase  # Initialize current and base value to 100
  
  # Attribute Scaling
  Real Magnitude: UseMagnitude 
  Cached Scaler:  # Some scaler or none
  
  Collision Policy: UseSetCollisionPolicy  # If the same attribute is declared, which to choose?
  
  Overflow:  # Attribute value bounds
  Constraints:  # Value behaviour
```

### Attribute Scaling & Derivation

Modify the attribute's value by introducing a cached scaler. The cached scaler acts as an attribute impact derivation.
As opposed to regular scalers, cached scalers access cached attribute values, which expose significantly more information than normal attribute values.

In this example, `Attack` scales with `Strength`.

```yaml
Attribute: Attack
Magnitude: 40

Real Magnitude: AddScaler
Scaler:
  - Type: Attribute-backed Scaler
    Capture Attribute: Strength
    Operation: MultiplyByOperand
    Operand: 2.5
```

True attack value: `40 + (Strength) * 2.5`. If `Strength = 16`, then `Attack = 40 + 16 * 2.5 = 80`.

In this example, `Health` scales with level.

```yaml
Attribute: Health
Magnitude: 100

Real Magnitude: MultiplyWithScaler
Scaler:
  - Type: Simple Scaler
    Level Values: [1, 1.2, 1.4, 1.6, 1.8, 2]
```

True attack value: `100 * (Level Value)`. If `Level = 4`, then `Health = 100 * 1.6 = 160`.

### Overflow (Bounds)

Declare overflow and bounding logic for this attribute:

```yaml
Attribute Value: 50
Overflow Policy: ZeroToBase  # default
Bounds Floor: Current/Base  # Current and base lower bounds
Bounds Ceil: Current/Base  # Current and base upper bounds
```

| Policy      | Description                 |
|-------------|-----------------------------|
| ZeroToBase  | From 0/0 to base/base       |
| FloorToBase | From floor to base/base     |
| ZeroToCeil  | From 0/0 to ceil            |
| FloorToCeil | From declared floor to ceil |
| Unlimited   | Unbounded                   |

### Constraints (Behaviour)

Apply constraints to the attribute value. 
Depending on your constraints setup, incoming modifications may be artificially inflated/deflated.

```yaml
Auto Clamp: true  # Always clamp value to declared overflow bounds
Auto Scale with Base: false  # Changes to base are propagated proportionally to current value
Rounding Mode: None  # Floor, Ceil, Round, SnapTo
Snap Interval: 0  # N/A if mode != SnapTo
```

## SubSets

Compose attribute sets from other sets:

```yaml
Attribute Set: PlayerFull
SubSets:
  - AttrSet_Vitals      # Health, Mana
  - AttrSet_Combat      # Attack, Defense
  - AttrSet_Movement    # MoveSpeed
```

SubSet attributes are merged into the parent set.

!!! note "Collision Policy"
    If the same attribute exists in multiple subsets, the collision policy is used to decide which declaration to use. This decision is made once at initialization.

## Initialization

Attribute sets are applied during entity initialization:

```csharp
// In GameplayAbilitySystem.Initialize(EntityIdentity data)

AttributeSystem.Setup(Data.AttributeSet);  # Pass initial data
            
// More initialization logic...

AttributeSystem.Initialize();  # Initialize attribute system

// Rest of implementation
```

### Manual Application

```csharp
// Provide a new attribute
attributeSystem.ProvideAttribute(Attribute, AttributeBlueprint);

// Apply an additional attribute set
attributeSystem.ProvideAttributeSet(AttributeSet);
```

## Best Practices

| Do | Don't |
|----|-------|
| Group related attributes | One massive set |
| Use SubSets for composition | Duplicate attributes |
| Constrain bounded values | Allow negative health |
| Scale with level appropriately | Flat values everywhere |
| Name sets by purpose | Generic names |