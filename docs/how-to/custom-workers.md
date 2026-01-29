# Custom Workers

Extend functionality with custom effect, attribute, and entity workers.

## Effect Worker

### Basic Structure

```csharp
using System;
using UnityEngine;
using FarEmerald.PlayForge;

[Serializable]
public class MyEffectWorker : AbstractEffectWorker
{
    // Serialized fields appear in inspector
    public float SomeValue = 1f;
    public GameObject VFXPrefab;
    
    public override void OnEffectApplied(GameplayEffectSpec spec, ITarget target)
    {
        // Called when effect is first applied
    }
    
    public override void OnEffectTick(GameplayEffectSpec spec, ITarget target, int tickIndex)
    {
        // Called on each periodic tick
    }
    
    public override void OnEffectExpired(GameplayEffectSpec spec, ITarget target)
    {
        // Called when duration ends naturally
    }
    
    public override void OnEffectRemoved(GameplayEffectSpec spec, ITarget target)
    {
        // Called when manually removed
    }
    
    public override void OnStackAdded(GameplayEffectSpec spec, ITarget target, int newStackCount)
    {
        // Called when a new stack is added
    }
}
```

### Lifesteal Worker

```csharp
[Serializable]
public class LifestealWorker : AbstractEffectWorker
{
    [Range(0f, 1f)]
    public float LifestealPercent = 0.15f;
    
    public override void OnEffectApplied(GameplayEffectSpec spec, ITarget target)
    {
        // Only process damage (negative magnitude)
        float magnitude = spec.GetCalculatedMagnitude();
        if (magnitude >= 0) return;
        
        float damage = Mathf.Abs(magnitude);
        float healAmount = damage * LifestealPercent;
        
        // Heal the source
        var source = spec.Source;
        if (source != null)
        {
            source.AsGAS().ModifyAttributeCurrentValue(
                Attributes.Health, 
                healAmount
            );
        }
    }
}
```

### Spawn VFX Worker

```csharp
[Serializable]
public class SpawnVFXWorker : AbstractEffectWorker
{
    public GameObject VFXPrefab;
    public bool AttachToTarget = true;
    public bool DestroyOnRemove = true;
    public Vector3 Offset;
    
    private GameObject _spawnedVFX;
    
    public override void OnEffectApplied(GameplayEffectSpec spec, ITarget target)
    {
        if (VFXPrefab == null) return;
        
        var targetTransform = (target as MonoBehaviour)?.transform;
        if (targetTransform == null) return;
        
        var position = targetTransform.position + Offset;
        _spawnedVFX = GameObject.Instantiate(VFXPrefab, position, Quaternion.identity);
        
        if (AttachToTarget)
            _spawnedVFX.transform.SetParent(targetTransform);
    }
    
    public override void OnEffectRemoved(GameplayEffectSpec spec, ITarget target)
    {
        if (DestroyOnRemove && _spawnedVFX != null)
            GameObject.Destroy(_spawnedVFX);
    }
}
```

## Attribute Worker

### Diminishing Returns

```csharp
[Serializable]
public class DiminishingReturnsWorker : AbstractAttributeWorker
{
    public float SoftCap = 50f;
    public float HardCap = 100f;
    public float DiminishingFactor = 0.5f;
    
    public override float ModifyFinalValue(CachedAttributeValue cached, float calculatedValue)
    {
        if (calculatedValue <= SoftCap)
            return calculatedValue;
            
        if (calculatedValue >= HardCap)
            return HardCap;
            
        // Apply diminishing returns between soft and hard cap
        float excess = calculatedValue - SoftCap;
        float diminished = excess * DiminishingFactor;
        
        return Mathf.Min(SoftCap + diminished, HardCap);
    }
}
```

### Percentage Display

```csharp
[Serializable]
public class PercentageAttributeWorker : AbstractAttributeWorker
{
    public Attribute MaxAttribute;  // e.g., HealthMax
    
    public override void OnValueChanged(
        CachedAttributeValue cached, 
        float oldValue, 
        float newValue)
    {
        var maxValue = cached.Owner.GetAttributeValue(MaxAttribute);
        float percentage = (newValue / maxValue) * 100f;
        
        Debug.Log($"{cached.Attribute.name}: {percentage:F1}%");
    }
}
```

## Entity Worker

### Death Handler

```csharp
[Serializable]
public class DeathHandlerWorker : AbstractEntityWorker
{
    public Attribute HealthAttribute;
    public float DeathThreshold = 0f;
    public Tag DeathTag;
    public GameplayEffect[] OnDeathEffects;
    
    public event Action<ISource> OnDeath;
    
    public override void OnAttributeChanged(
        ISource entity, 
        Attribute attribute, 
        float oldValue, 
        float newValue)
    {
        if (attribute != HealthAttribute) return;
        
        // Check for death transition
        if (newValue <= DeathThreshold && oldValue > DeathThreshold)
        {
            HandleDeath(entity);
        }
    }
    
    private void HandleDeath(ISource entity)
    {
        // Grant death tag
        entity.AsGAS().GrantTag(DeathTag);
        
        // Apply death effects
        foreach (var effect in OnDeathEffects)
        {
            var spec = entity.GenerateEffectSpec(null, effect);
            entity.ApplyGameplayEffect(spec);
        }
        
        // Notify listeners
        OnDeath?.Invoke(entity);
    }
}
```

### Level Up Handler

```csharp
[Serializable]
public class LevelUpWorker : AbstractEntityWorker
{
    public Attribute ExperienceAttribute;
    public Attribute LevelAttribute;
    public int[] XPThresholds = { 100, 300, 600, 1000, 1500 };
    public GameplayEffect LevelUpEffect;
    
    public event Action<ISource, int, int> OnLevelUp;
    
    public override void OnAttributeChanged(
        ISource entity, 
        Attribute attribute, 
        float oldValue, 
        float newValue)
    {
        if (attribute != ExperienceAttribute) return;
        
        int currentLevel = (int)entity.AsGAS().GetAttributeValue(LevelAttribute);
        int newLevel = CalculateLevel(newValue);
        
        if (newLevel > currentLevel)
        {
            PerformLevelUp(entity, currentLevel, newLevel);
        }
    }
    
    private int CalculateLevel(float xp)
    {
        for (int i = XPThresholds.Length - 1; i >= 0; i--)
        {
            if (xp >= XPThresholds[i])
                return i + 2;
        }
        return 1;
    }
    
    private void PerformLevelUp(ISource entity, int from, int to)
    {
        entity.AsGAS().SetAttributeBaseValue(LevelAttribute, to);
        
        if (LevelUpEffect != null)
        {
            var spec = entity.GenerateEffectSpec(null, LevelUpEffect);
            entity.ApplyGameplayEffect(spec);
        }
        
        OnLevelUp?.Invoke(entity, from, to);
    }
}
```

## Adding to Assets

```yaml
# On a GameplayEffect
Workers:
  - Type: LifestealWorker
    Lifesteal Percent: 0.15
    
  - Type: SpawnVFXWorker
    VFX Prefab: VFX_Hit
    Attach To Target: true
```

## Best Practices

| Do | Don't |
|----|-------|
| Keep workers stateless | Store state between applications |
| Single responsibility | Giant multi-purpose workers |
| Fast callbacks (<1ms) | Heavy computation in callbacks |
| Use serialized fields | Hardcode values |
