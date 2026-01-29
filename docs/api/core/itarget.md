# ITarget

<span class="type-badge interface">Interface</span>

Represents an entity that can receive gameplay effects.

## Definition

```csharp
public interface ITarget
```

**Namespace:** `FarEmerald.PlayForge`

## Methods

### AsGAS

```csharp
AbilitySystemComponent AsGAS()
```

Gets the AbilitySystemComponent for this entity.

**Returns:** The ability system component, or `null` if none.

---

### GetAppliedTags

```csharp
List<Tag> GetAppliedTags()
```

Gets all tags currently applied to this entity.

**Returns:** List of active tags.

---

### ApplyGameplayEffect

```csharp
void ApplyGameplayEffect(GameplayEffectSpec spec)
```

Applies a gameplay effect to this entity.

| Parameter | Type | Description |
|-----------|------|-------------|
| spec | `GameplayEffectSpec` | The effect specification |

---

## Example Implementation

### Full Entity

```csharp
public class GameEntity : MonoBehaviour, ITarget
{
    private AbilitySystemComponent _abilitySystem;
    
    void Awake()
    {
        _abilitySystem = GetComponent<AbilitySystemComponent>();
    }
    
    public AbilitySystemComponent AsGAS() => _abilitySystem;
    
    public List<Tag> GetAppliedTags() => _abilitySystem?.GetAppliedTags() ?? new();
    
    public void ApplyGameplayEffect(GameplayEffectSpec spec)
    {
        _abilitySystem?.ApplyGameplayEffect(spec);
    }
}
```

### Simple Destructible

```csharp
public class Destructible : MonoBehaviour, ITarget
{
    [SerializeField] private float _health = 100f;
    
    public AbilitySystemComponent AsGAS() => null;
    
    public List<Tag> GetAppliedTags() => new() { Tags.Status.Alive };
    
    public void ApplyGameplayEffect(GameplayEffectSpec spec)
    {
        // Only handle instant damage
        if (spec.Base.DurationSpecification.DurationPolicy != EEffectDurationPolicy.Instant)
            return;
            
        _health += spec.GetCalculatedMagnitude();
        
        if (_health <= 0)
            Destroy(gameObject);
    }
}
```

---

## Usage

```csharp
void DamageTarget(ITarget target, GameplayEffect damageEffect)
{
    // Check tags
    var tags = target.GetAppliedTags();
    if (tags.Contains(Tags.Status.Invulnerable))
        return;
    
    // Apply damage
    var spec = source.GenerateEffectSpec(origin, damageEffect);
    target.ApplyGameplayEffect(spec);
}
```
