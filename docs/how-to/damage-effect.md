# Create a Damage Effect

Learn to create different types of damage effects.

## Instant Damage

Basic single-hit damage:

```yaml
Name: "Slash Damage"
Asset Tag: Effect.Damage.Physical.Slash

Duration Policy: Instant

Attribute Target: Health
Target Impact: Current
Impact Operation: Add
Magnitude: -30

Target Requirements:
  Avoid Tags: [Status.Invulnerable]
```

## Damage Over Time (DOT)

Periodic damage effect:

```yaml
Name: "Poison"
Asset Tag: Effect.Debuff.Poison

Duration Policy: Durational
Duration: 10.0
Enable Periodic Ticks: true
Tick Interval: 2.0
Ticks: 5

Attribute Target: Health
Impact Operation: Add
Magnitude: -8  # Per tick = 40 total

Granted Tags: [Status.Poisoned]
```

## Stacking DOT

DOT that stacks for increased damage:

```yaml
Name: "Bleed"
Asset Tag: Effect.Debuff.Bleed

Duration Policy: Durational
Duration: 6.0
Enable Periodic Ticks: true
Tick Interval: 1.0

Reapplication Policy: StackExistingContainers
Stack Amount: 5

Magnitude: -5  # Per tick per stack
# At 5 stacks: -25 per tick

Granted Tags: [Status.Bleeding]
```

## Percentage-Based Damage

Damage based on target's max health:

```csharp
// Custom effect worker
[Serializable]
public class PercentHealthDamageWorker : AbstractEffectWorker
{
    public float PercentOfMaxHealth = 0.1f; // 10%
    
    public override void OnEffectApplied(GameplayEffectSpec spec, ITarget target)
    {
        var maxHealth = target.AsGAS().GetAttributeValue(Attributes.HealthMax);
        var damage = maxHealth * PercentOfMaxHealth;
        
        target.AsGAS().ModifyAttributeCurrentValue(Attributes.Health, -damage);
    }
}
```

## Elemental Damage

With resistance checking:

```yaml
Name: "Fire Blast"
Asset Tag: Effect.Damage.Fire

Magnitude: -50

Target Requirements:
  Avoid Tags: [Status.Immune.Fire]
```

```csharp
// Resistance worker
[Serializable]
public class ElementalResistanceWorker : AbstractEffectWorker
{
    public Attribute ResistanceAttribute;
    
    public override void OnEffectApplied(GameplayEffectSpec spec, ITarget target)
    {
        var resistance = target.AsGAS().GetAttributeValue(ResistanceAttribute);
        var reduction = 1f - (resistance / 100f);
        
        // Modify final damage
        spec.ModifyMagnitude(spec.GetMagnitude() * reduction);
    }
}
```

## Critical Hits

```csharp
[Serializable]
public class CriticalHitWorker : AbstractEffectWorker
{
    public float CritMultiplier = 2.0f;
    
    public override void OnEffectApplied(GameplayEffectSpec spec, ITarget target)
    {
        var source = spec.Source;
        var critChance = source.AsGAS().GetAttributeValue(Attributes.CritChance);
        
        if (Random.value * 100f < critChance)
        {
            spec.ModifyMagnitude(spec.GetMagnitude() * CritMultiplier);
            // Trigger crit VFX
        }
    }
}
```

## Level-Scaled Damage

```yaml
Magnitude: -40

Real Magnitude: Add Scaler
Magnitude Scaler:
  Type: LinearScaler
  Level Values: [-40, -55, -70, -90, -115]
```

Or with curves:

```yaml
Magnitude Scaler:
  Type: CurveScaler
  Curve: (AnimationCurve)
  Min Level: 1
  Max Level: 20
  Output Min: -40
  Output Max: -200
```
