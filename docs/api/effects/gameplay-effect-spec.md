# GameplayEffectSpec

<span class="type-badge class">Class</span>

Runtime instance of an effect with context.

## Definition

```csharp
public class GameplayEffectSpec
```

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `Base` | `GameplayEffect` | Source effect asset |
| `Source` | `ISource` | Entity applying the effect |
| `Origin` | `IEffectOrigin` | Origin context (ability, item) |
| `Level` | `int` | Level for scaling |
| `Stacks` | `int` | Current stack count |

## Methods

### GetCalculatedMagnitude

```csharp
public float GetCalculatedMagnitude()
```

Gets the final magnitude after scaling.

---

### GetOwner

```csharp
public ISource GetOwner()
```

Gets the source entity.

---

## Example

```csharp
// Create spec
var spec = source.GenerateEffectSpec(abilitySpec, damageEffect);

// Modify before applying
spec.Level = 5;

// Apply
target.ApplyGameplayEffect(spec);
```
