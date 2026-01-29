# Workers

**Workers** are modular components that extend effect, attribute, and entity behavior with custom logic.

## Overview

Workers can:

- React to events (damage dealt, effect applied)
- Modify calculations (damage multipliers)
- Trigger side effects (VFX, sounds)
- Implement complex mechanics

## Worker Types

| Type | Attached To | Purpose |
|------|-------------|---------|
| Effect Workers | GameplayEffect | Extend effect behavior |
| Attribute Workers | Attribute | Modify calculations |
| Entity Workers | EntityIdentity | Entity-level behavior |

## Effect Workers

```yaml
# On a damage effect
Workers:
  - Type: DamageNumberWorker
    Show Crit: true
  - Type: SpawnVFXWorker
    Prefab: VFX_Hit
```

### Built-in Effect Workers

| Worker | Purpose |
|--------|---------|
| `SpawnVFXWorker` | Visual effects |
| `PlaySoundWorker` | Audio |
| `DamageNumberWorker` | Floating numbers |
| `ApplyForceWorker` | Knockback |

### Creating Custom Effect Workers

```csharp
[Serializable]
public class LifestealWorker : AbstractEffectWorker
{
    [Range(0f, 1f)]
    public float LifestealPercent = 0.1f;
    
    public override void OnEffectApplied(GameplayEffectSpec spec, ITarget target)
    {
        if (spec.Base.ImpactSpecification.Magnitude >= 0)
            return;  // Only for damage
            
        float damage = Mathf.Abs(spec.GetCalculatedMagnitude());
        float heal = damage * LifestealPercent;
        
        // Heal the source
        if (spec.Origin?.GetOwner() is ITarget source)
        {
            // Apply heal to source
        }
    }
}
```

### Effect Worker Callbacks

```csharp
public abstract class AbstractEffectWorker
{
    public virtual void OnEffectApplied(GameplayEffectSpec spec, ITarget target) { }
    public virtual void OnEffectTick(GameplayEffectSpec spec, ITarget target, int tick) { }
    public virtual void OnEffectExpired(GameplayEffectSpec spec, ITarget target) { }
    public virtual void OnEffectRemoved(GameplayEffectSpec spec, ITarget target) { }
    public virtual void OnStackAdded(GameplayEffectSpec spec, ITarget target, int stacks) { }
}
```

## Attribute Workers

```csharp
[Serializable]
public class DiminishingReturnsWorker : AbstractAttributeWorker
{
    public float Threshold = 50f;
    public float Factor = 0.5f;
    
    public override float ModifyFinalValue(CachedAttributeValue cached, float value)
    {
        if (value <= Threshold)
            return value;
            
        float excess = value - Threshold;
        return Threshold + (excess * Factor);
    }
}
```

## Entity Workers

```yaml
Worker Group:
  Workers:
    - Type: DeathHandlerWorker
      Death Attribute: Health
      Death Threshold: 0
      Death Tag: Status.Dead
      
    - Type: LevelUpWorker
      Experience Attribute: Experience
      Level Attribute: Level
```

### Death Handler Example

```csharp
[Serializable]
public class DeathHandlerWorker : AbstractEntityWorker
{
    public Attribute DeathAttribute;
    public float DeathThreshold = 0f;
    public Tag DeathTag;
    
    public override void OnAttributeChanged(
        ISource entity, Attribute attr, float oldVal, float newVal)
    {
        if (attr != DeathAttribute) return;
        
        if (newVal <= DeathThreshold && oldVal > DeathThreshold)
        {
            entity.AsGAS().GrantTag(DeathTag);
            OnEntityDeath?.Invoke(entity);
        }
    }
    
    public event Action<ISource> OnEntityDeath;
}
```

## Best Practices

| Do | Don't |
|----|-------|
| Single responsibility | Monolithic workers |
| Stateless design | Store state in workers |
| Fast callbacks (<1ms) | Heavy computation |
