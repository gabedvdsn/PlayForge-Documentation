# API Reference

Complete API documentation for PlayForge GAS.

## Namespaces

```csharp
using FarEmerald.PlayForge;           // Core runtime
using FarEmerald.PlayForge.Extended;  // Extended features
```

## Quick Navigation

### Core Components

| Class | Description |
|-------|-------------|
| [AbilitySystemComponent](core/ability-system-component.md) | Manages abilities and effects |
| [AttributeSystemComponent](core/attribute-system-component.md) | Manages entity attributes |
| [ISource](core/isource.md) | Entity that applies effects |
| [ITarget](core/itarget.md) | Entity that receives effects |

### Abilities

| Class | Description |
|-------|-------------|
| [Ability](abilities/ability.md) | Ability asset definition |
| [AbilitySpec](abilities/ability-spec.md) | Runtime ability instance |
| [AbilityBehaviour](abilities/ability-behaviour.md) | Ability execution behavior |

### Effects

| Class | Description |
|-------|-------------|
| [GameplayEffect](effects/gameplay-effect.md) | Effect asset definition |
| [GameplayEffectSpec](effects/gameplay-effect-spec.md) | Runtime effect instance |

### Attributes

| Class | Description |
|-------|-------------|
| [Attribute](attributes/attribute.md) | Attribute definition |
| [AttributeSet](attributes/attribute-set.md) | Collection of attributes |

### Tags

| Class | Description |
|-------|-------------|
| [Tag](tags/tag.md) | Tag definition |
| [AvoidRequireTagGroup](tags/avoid-require-tag-group.md) | Tag requirements |

### Scalers

| Class | Description |
|-------|-------------|
| [AbstractAttributeScaler](scalers/abstract-attribute-scaler.md) | Base scaler class |

---

## Common Patterns

```csharp
// Get attribute values
float health = attrSystem.GetCurrentValue(Attributes.Health);

// Apply effects
var spec = source.GenerateEffectSpec(origin, effect);
target.ApplyGameplayEffect(spec);

// Activate abilities
if (abilitySystem.TryActivateAbility(ability, out var proxy))
{
    Debug.Log("Activated!");
}

// Check tags
bool isStunned = target.GetAppliedTags().Contains(Tags.Status.Stunned);
```
