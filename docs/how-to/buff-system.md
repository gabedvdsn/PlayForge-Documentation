# Create a Buff System

Build stackable, refreshable buffs with visual feedback.

## Basic Buff

A simple timed stat boost:

```yaml
Name: "Strength Boost"
Asset Tag: Effect.Buff.Strength

Duration Policy: Durational
Duration: 15.0

Attribute Target: Attack
Target Impact: Current
Impact Operation: Add
Magnitude: 10

Granted Tags: [Status.Buff.Strength]
```

## Percentage Buff

Multiply instead of add:

```yaml
Name: "Damage Amplifier"
Asset Tag: Effect.Buff.DamageAmp

Duration Policy: Durational
Duration: 10.0

Attribute Target: Attack
Impact Operation: Multiply
Magnitude: 1.25  # +25%
```

## Refreshable Buff

Reapplication resets duration:

```yaml
Name: "Battle Fury"
Asset Tag: Effect.Buff.Fury

Duration Policy: Durational
Duration: 8.0
Reapplication Policy: RefreshDuration

Attribute Target: Attack
Magnitude: 15
```

## Stacking Buff

Each application adds a stack:

```yaml
Name: "Rampage"
Asset Tag: Effect.Buff.Rampage

Duration Policy: Durational
Duration: 5.0
Reapplication Policy: StackExistingContainers
Stack Amount: 10

Attribute Target: Attack
Magnitude: 3  # Per stack

Granted Tags: [Status.Rampage]
```

At 10 stacks: +30 Attack

## Buff with Multiple Stats

```yaml
Name: "Battle Preparation"
Asset Tag: Effect.Buff.BattlePrep

Duration Policy: Durational
Duration: 30.0

# Use multiple impacts or contained effects
Packets:
  - Effect: Effect_BattlePrep_Attack   # +10 Attack
    Application: OnApplication
  - Effect: Effect_BattlePrep_Defense  # +10 Defense
    Application: OnApplication
```

## Conditional Buff

Only applies under certain conditions:

```yaml
Name: "Berserker Rage"
Asset Tag: Effect.Buff.Berserker

Source Requirements:
  Require Tags: [Status.LowHealth]  # Applied when below 25% HP

Attribute Target: Attack
Impact Operation: Multiply
Magnitude: 1.5  # +50% when low health
```

## Buff with Visual Feedback

Add workers for effects:

```yaml
Workers:
  - Type: SpawnVFXWorker
    Prefab: VFX_Buff_Aura
    Attach To Target: true
    Destroy On Remove: true
    
  - Type: PlaySoundWorker
    Sound: buff_applied.wav
    Play On: OnApplication
```

## Buff UI Integration

```csharp
public class BuffDisplay : MonoBehaviour
{
    private AbilitySystemComponent _asc;
    
    void Start()
    {
        _asc = GetComponent<AbilitySystemComponent>();
        _asc.OnEffectApplied += OnEffectApplied;
        _asc.OnEffectRemoved += OnEffectRemoved;
    }
    
    void OnEffectApplied(GameplayEffectSpec spec)
    {
        if (spec.Base.Tags.HasTag(Tags.Effect.Buff))
        {
            // Add buff icon to UI
            AddBuffIcon(spec);
        }
    }
    
    void OnEffectRemoved(GameplayEffectSpec spec)
    {
        // Remove buff icon
        RemoveBuffIcon(spec);
    }
}
```

## Buff Priority

Handle buff conflicts:

```csharp
[Serializable]
public class BuffPriorityWorker : AbstractEffectWorker
{
    public int Priority = 1;
    public Tag ConflictTag;  // e.g., Effect.Buff.Speed
    
    public override void OnEffectApplied(GameplayEffectSpec spec, ITarget target)
    {
        // Remove lower priority buffs of same type
        var existing = target.AsGAS().GetEffectsWithTag(ConflictTag);
        foreach (var effect in existing)
        {
            var worker = effect.GetWorker<BuffPriorityWorker>();
            if (worker != null && worker.Priority < Priority)
            {
                target.AsGAS().RemoveGameplayEffect(effect);
            }
        }
    }
}
```
