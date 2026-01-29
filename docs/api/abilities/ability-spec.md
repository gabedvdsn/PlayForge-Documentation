# AbilitySpec

<span class="type-badge class">Class</span>

Runtime instance of an ability with context.

## Definition

```csharp
public class AbilitySpec
```

**Namespace:** `FarEmerald.PlayForge`

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `Base` | `Ability` | The source ability asset |
| `Owner` | `ISource` | Entity that owns this ability |
| `Level` | `int` | Current ability level |

## Methods

### GetOwner

```csharp
public ISource GetOwner()
```

Gets the owning entity.

---

### GetLevel

```csharp
public int GetLevel()
```

Gets the current level for scaling.

---

## Example

```csharp
if (abilitySystem.TryActivateAbility(fireball, out var proxy))
{
    AbilitySpec spec = proxy.Spec;
    Debug.Log($"Activated {spec.Base.GetName()} at level {spec.Level}");
}
```
