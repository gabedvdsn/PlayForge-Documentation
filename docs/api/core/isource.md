# ISource

<span class="type-badge interface">Interface</span>

Represents an entity that can apply effects and activate abilities.

## Definition

```csharp
public interface ISource : ITarget
```

**Namespace:** `FarEmerald.PlayForge`

**Inherits:** [ITarget](itarget.md)

## Methods

### AsGAS

```csharp
AbilitySystemComponent AsGAS()
```

Gets the AbilitySystemComponent for this entity.

**Returns:** The ability system component.

---

### GetAppliedTags

```csharp
List<Tag> GetAppliedTags()
```

Gets all tags currently applied to this entity.

**Returns:** List of active tags.

---

### GetAffiliation

```csharp
List<Tag> GetAffiliation()
```

Gets faction/affiliation tags for this entity.

**Returns:** List of affiliation tags.

---

### ApplyGameplayEffect

```csharp
void ApplyGameplayEffect(GameplayEffectSpec spec)
```

Applies an effect to this entity.

---

### GenerateEffectSpec

```csharp
GameplayEffectSpec GenerateEffectSpec(IEffectOrigin origin, GameplayEffect effect)
```

Creates an effect spec with this entity as the source.

| Parameter | Type | Description |
|-----------|------|-------------|
| origin | `IEffectOrigin` | Origin context (ability, item) |
| effect | `GameplayEffect` | Effect template |

**Returns:** A new effect specification.

---

### FindAbilitySystem

```csharp
bool FindAbilitySystem(out AbilitySystemComponent asc)
```

Attempts to get the ability system component.

**Returns:** `true` if found.

---

## Example Implementation

```csharp
public class EntityController : MonoBehaviour, ISource
{
    [SerializeField] private EntityIdentity _identity;
    
    private AbilitySystemComponent _abilitySystem;
    
    void Awake()
    {
        _abilitySystem = GetComponent<AbilitySystemComponent>();
    }
    
    // ITarget
    public AbilitySystemComponent AsGAS() => _abilitySystem;
    
    public List<Tag> GetAppliedTags() => _abilitySystem.GetAppliedTags();
    
    public void ApplyGameplayEffect(GameplayEffectSpec spec)
    {
        _abilitySystem.ApplyGameplayEffect(spec);
    }
    
    // ISource
    public List<Tag> GetAffiliation() => _identity.Tags.ContextTags;
    
    public GameplayEffectSpec GenerateEffectSpec(IEffectOrigin origin, GameplayEffect effect)
    {
        return _abilitySystem.GenerateEffectSpec(origin, effect);
    }
    
    public bool FindAbilitySystem(out AbilitySystemComponent asc)
    {
        asc = _abilitySystem;
        return asc != null;
    }
}
```
