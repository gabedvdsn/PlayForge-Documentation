# Abilities

**Abilities** are interactive actions with costs, cooldowns, targeting, and multi-stage execution.

## Overview

An ability defines:

- **Definition** — Name, activation policy
- **Tags** — Identity and requirements
- **Cost & Cooldown** — Resource costs and timing
- **Behaviour** — Targeting and execution stages

!!! note "Level Provider Asset"
    Ability is a `Level Provider`. For more information, see [Forge/Level Providers](forge.md).

## Creating Abilities

1. **Forge Manager > Create > Ability**
2. Configure:

```yaml
Name: "Guitar Solo"
Asset Tag: Ability.GuitarSolo
Activation Policy: UseLocalPolicy
Activate Immediately: false  # Useful for passive/aura abilities
Texture: GuitarSolo_Icon

Context Tags: [Context.Ability.Ultimate]
Active Granted Tags: [Status.Music]  # Tags granted while ability is active
Passive Granted Tags: []  # Tags granted while ability is learned

# Rest of implementation...
```

## Activation Policies

| Policy                     | Description                                             |
|----------------------------|---------------------------------------------------------|
| `Use Local Policy`         | Use entity's activation policy                          |
| `Always Activate`          | Always (with restrictions) allow ability activation     |
| `Activate If Idle`         | Activate if system is not busy                          |
| `Queue Activation If Busy` | Queue activation if system is busy; auto-activate later |

## Cost and Cooldown

Both are **GameplayEffects**. To scale cost and cooldown with level, link the source ability to both effects.

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
Asset Tag: Effect.Cooldown.Fireball  # Cooldown is tracked internally via the asset tag
```

## Ability Behaviour

This section is the true identity of the ability, and is what sets PlayForge apart from other GAS frameworks: utilize composition to create meaningful abilities without the drawbacks of an inheritance-based approach.

```yaml
Behaviour:
  Targeting: (optional)
  Use Implicit Targeting: true  # Auto-include source as target in the primary position
  
  Stages:
    - Stage 0
      Tasks
    - Stage 1
      Tasks
```

```yaml
Stage 0:
  - Task 0:
      Type: AnimationTask
Stage 1:
  - Task 0:
      Type: Create Monobehaviour Process
      Process: Fireball
```

### Stages

Ability behaviour executes stages sequentially, and only advances when the `Stage Policy` requirements are met.

```yaml
Stage:
  Stage Policy: All  # When to advance
  Apply Usage Effects: true
  Tasks:
    - ApplyEffectsAbilityTask
```

### Stage Policies

| Policy | Advances When      |
|--------|--------------------|
| `All`  | All tasks complete |
| `Any`  | Any task completes |

## Ability Tasks

Tasks comprise of an initial setup, the asynchronous activation period, and a final cleanup.

### Creating Ability Tasks

There are many authored tasks, and creating your own is simple. Note that a task's setup and cleanup methods are always called, regardless of task completion or status.

| Task Type        | Inherit From            |
|------------------|-------------------------|
| Targeting        | `AbstractTargetingTask` |
| DelayAbilityTask | `AbstractAbilityTask`   |

### Task Example

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
abilitySystem.GiveAbility(fireballAbility, level: 1, out int abilityIndex);

// Activate
var activationRequest = abilitySystem.CreateActivationRequest(abilityIndex);
if (abilitySystem.TryActivateAbility(activationRequest))
{
    Debug.Log("Activated!");
}
```

## Runtime Proxy

When an ability is activated, a runtime instance is created called the `Runtime Proxy`. This proxy manages the lifetime of the ability across its stages and tasks asynchronously.

```csharp
var activationRequest = abilitySystem.CreateActivationRequest(abilityIndex);
if (abilitySystem.TryActivateAbility(activationRequest))
{
    // Runtime Proxy is created
    // Proxy begins at Stage 0
    // Proxy disposes itself after all stages are complete (or on interruption)
}
```

## Ability Injections

To manage the ability during its runtime, inject commands directly into the runtime proxy. There are 5 authored injections:

| Injection                       | Behaviour                                           |
|---------------------------------|-----------------------------------------------------|
| Interrupt                       | Runtime proxy is cancelled completely               |
| Skip Current Stage              | Skip current stage without maintaining              |
| Skip and Maintain Current Stage | Skip and maintain current stage                     |
| Stop Maintain Last              | Stop maintaining the most recently maintained stage |
| Stop Maintain All               | Stop maintaining all maintained stages              |

!!! note "Stage Maintenance"
    When a stage is maintained, it is set aside to finish its normal runtime behaviour, and the runtime proxy immediately moves on to the next stage. The ability remains active until all maintained stages complete.

## Level Management

```csharp
abilitySystem.GiveAbility(fireballAbility, level: 1, out int abilityIndex);

abilitySystem.SetAbilityLevel(abilityIndex, 3);
int level = abilitySystem.GetAbilityLevel(abilityIndex);
```

## Validation Rules

Assign validation rules for source and target. These are modular rules that must validate for the ability to activate.

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

## More Information
[API: Ability](../api/abilities/ability.md)
[API: Ability Behaviour](../api/abilities/ability.md)
[API: Ability Instance](../api/abilities/ability.md)