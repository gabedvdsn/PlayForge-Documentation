# Abilities

**Abilities** are interactive actions with costs, cooldowns, targeting, and multi-stage execution.

## Overview

An ability defines:

- **Definition** — Name, activation policy
- **Tags** — Identity and requirements
- **Cost & Cooldown** — Resource costs and timing
- **Behaviour** — Targeting and execution stages

## Creating Abilities

1. **Create > PlayForge > Ability**
2. Configure:

```yaml
Name: "Fireball"
Asset Tag: Ability.Fireball
Activation Policy: Active

Starting Level: 1
Max Level: 5

Cost: Effect_Fireball_Cost
Cooldown: Effect_Fireball_Cooldown
```

## Activation Policies

| Policy | Description |
|--------|-------------|
| `Active` | Manually activated |
| `Passive` | Always active when learned |
| `Triggered` | Activated by events |

## Cost and Cooldown

Both are **GameplayEffects**:

**Cost:**
```yaml
Duration Policy: Instant
Attribute Target: Mana
Magnitude: -30
```

**Cooldown:**
```yaml
Duration Policy: Durational
Duration: 5.0
Asset Tag: Ability.Fireball.Cooldown
```

## Ability Behaviour

```yaml
Behaviour:
  Targeting: (optional)
  Use Implicit Targeting: true
  
  Stages:
    - Stage 0
    - Stage 1
```

### Stages

Execute sequentially, each containing tasks:

```yaml
Stage:
  Stage Policy: All  # When to advance
  Apply Usage Effects: true
  Tasks:
    - ApplyEffectsAbilityTask
```

### Stage Policies

| Policy | Advances When |
|--------|---------------|
| `All` | All tasks complete |
| `Any` | Any task completes |
| `Maintained` | Stays active until cancelled |

## Ability Tasks

| Task | Purpose |
|------|---------|
| `ApplyEffectsAbilityTask` | Apply effects to targets |
| `DelayAbilityTask` | Wait for duration |
| `AnimationAbilityTask` | Play animation |

### ApplyEffectsAbilityTask

```csharp
public class ApplyEffectsAbilityTask : AbstractAbilityTask
{
    public List<GameplayEffect> Effects;
    
    public override async UniTask Activate(AbilityDataPacket data, CancellationToken token)
    {
        if (!data.TryGetFirstTarget(out var target))
            return;
            
        foreach (var effect in Effects)
        {
            target.ApplyGameplayEffect(
                target.GenerateEffectSpec(data.Spec, effect)
            );
        }
    }
}
```

## Granting and Activating

```csharp
// Grant
abilitySystem.GrantAbility(fireballAbility, level: 1);

// Activate
if (abilitySystem.TryActivateAbility(fireballAbility, out var proxy))
{
    Debug.Log("Activated!");
}

// Cancel
abilitySystem.CancelAbility(fireballAbility);
```

## Level Management

```csharp
abilitySystem.SetAbilityLevel(fireballAbility, 3);
int level = abilitySystem.GetAbilityLevel(fireballAbility);
```

## Validation Rules

```yaml
Source Activation Rules:
  - Type: CooldownValidation
  - Type: CostValidation
  
Target Activation Rules:
  - Type: TagValidation
    Require Tags: [Status.Alive]
```

## Callbacks

```csharp
abilitySystem.Callbacks.AbilityActivated += status =>
{
    Debug.Log($"Activated: {status.Ability.GetName()}");
};
```
