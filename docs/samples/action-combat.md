# Action Combat Sample

Fast-paced combat system with combos, dodging, and status effects.

## Overview

This sample demonstrates:

- Combo system with hit confirmation
- Dodge with invincibility frames
- Status effects (stun, burn, poison)
- Staggering and knockback
- Enemy AI integration

## Combo System

### Combo Tags

```yaml
Tags:
  Combo
  ├── Combo.Ready        # Can start combo
  ├── Combo.Hit1         # First hit landed
  ├── Combo.Hit2         # Second hit landed
  └── Combo.Finisher     # Can use finisher
```

### Light Attack (Combo Starter)

```yaml
# Ability_LightAttack.asset

Name: "Light Attack"
Activation Policy: Active

Tag Requirements:
  Source Requirements:
    Avoid Tags: [Status.Stunned, Status.Attacking]

Behaviour:
  Stages:
    - Stage Policy: All
      Tasks:
        - Type: GrantTagTask
          Tags: [Status.Attacking]
        - Type: AnimationTask
          Animation: "light_attack"
        - Type: ApplyEffectsAbilityTask
          Effects: [Effect_LightAttack_Damage]
        - Type: GrantTagTask
          Tags: [Combo.Hit1]
          Duration: 0.8  # Window to continue combo
        - Type: RemoveTagTask
          Tags: [Status.Attacking]
```

### Heavy Attack (Combo Follow-up)

```yaml
# Ability_HeavyAttack.asset

Name: "Heavy Attack"
Activation Policy: Active

Tag Requirements:
  Source Requirements:
    Require Tags: [Combo.Hit1]  # Must have landed light attack
    Avoid Tags: [Status.Stunned]

Behaviour:
  Stages:
    - Tasks:
        - Type: RemoveTagTask
          Tags: [Combo.Hit1]
        - Type: AnimationTask
          Animation: "heavy_attack"
        - Type: ApplyEffectsAbilityTask
          Effects: [Effect_HeavyAttack_Damage]
        - Type: GrantTagTask
          Tags: [Combo.Hit2]
          Duration: 1.0
```

### Finisher (Combo Ender)

```yaml
# Ability_Finisher.asset

Name: "Finisher"

Tag Requirements:
  Source Requirements:
    Require Tags: [Combo.Hit2]

Behaviour:
  Stages:
    - Tasks:
        - Type: RemoveTagTask
          Tags: [Combo.Hit2]
        - Type: AnimationTask
          Animation: "finisher"
        - Type: ApplyEffectsAbilityTask
          Effects: [Effect_Finisher_Damage, Effect_Knockback]
```

## Dodge System

### Dodge Ability

```yaml
# Ability_Dodge.asset

Name: "Dodge"
Activation Policy: Active

Cost: Effect_Dodge_Stamina  # -20 stamina
Cooldown: Effect_Dodge_Cooldown  # 0.5 seconds

Tag Requirements:
  Source Requirements:
    Avoid Tags: [Status.Stunned, Status.Dodging]

Behaviour:
  Stages:
    # Startup
    - Stage Policy: All
      Tasks:
        - Type: GrantTagTask
          Tags: [Status.Dodging]
          
    # I-Frames
    - Stage Policy: All
      Apply Usage Effects: true
      Tasks:
        - Type: ApplyRemoveEffectsTask
          Effects: [Effect_Invulnerable]  # Grants Status.Invulnerable
        - Type: MovementTask
          Direction: InputDirection
          Distance: 3.0
          Duration: 0.3
          
    # Recovery
    - Stage Policy: All
      Tasks:
        - Type: DelayTask
          Duration: 0.2
        - Type: RemoveTagTask
          Tags: [Status.Dodging]
```

### Invulnerability Effect

```yaml
# Effect_Invulnerable.asset

Name: "Invulnerable"
Duration Policy: Durational
Duration: 0.25  # I-frame window

Granted Tags: [Status.Invulnerable]
```

## Status Effects

### Stun

```yaml
# Effect_Stun.asset

Name: "Stun"
Asset Tag: Effect.Debuff.Stun

Duration Policy: Durational
Duration: 2.0

Granted Tags: [Status.Stunned, Status.CrowdControl]

Workers:
  - Type: SpawnVFXWorker
    Prefab: VFX_Stun_Stars
    Attach To Target: true
```

### Burn (DOT)

```yaml
# Effect_Burn.asset

Name: "Burn"
Asset Tag: Effect.Debuff.Burn

Duration Policy: Durational
Duration: 4.0
Enable Periodic Ticks: true
Tick Interval: 0.5

Reapplication Policy: RefreshDuration

Attribute Target: Health
Impact Operation: Add
Magnitude: -5  # Per tick

Granted Tags: [Status.Burning]

Workers:
  - Type: SpawnVFXWorker
    Prefab: VFX_Fire
    Attach To Target: true
```

### Stagger

```yaml
# Effect_Stagger.asset

Name: "Stagger"
Duration Policy: Durational
Duration: 0.3

Granted Tags: [Status.Staggered]

Workers:
  - Type: InterruptAbilitiesWorker
  - Type: PlayAnimationWorker
    Animation: "hit_react"
```

## Integration Code

```csharp
public class ActionCombatController : MonoBehaviour, ISource
{
    [Header("Abilities")]
    [SerializeField] private Ability _lightAttack;
    [SerializeField] private Ability _heavyAttack;
    [SerializeField] private Ability _finisher;
    [SerializeField] private Ability _dodge;
    
    private AbilitySystemComponent _asc;
    
    void Update()
    {
        HandleInput();
    }
    
    void HandleInput()
    {
        // Light attack / Combo
        if (Input.GetMouseButtonDown(0))
        {
            TryCombo();
        }
        
        // Dodge
        if (Input.GetKeyDown(KeyCode.Space))
        {
            _asc.TryActivateAbility(_dodge, out _);
        }
    }
    
    void TryCombo()
    {
        var tags = _asc.GetAppliedTags();
        
        // Check combo state and execute appropriate attack
        if (tags.Contains(Tags.Combo.Hit2))
        {
            _asc.TryActivateAbility(_finisher, out _);
        }
        else if (tags.Contains(Tags.Combo.Hit1))
        {
            _asc.TryActivateAbility(_heavyAttack, out _);
        }
        else
        {
            _asc.TryActivateAbility(_lightAttack, out _);
        }
    }
    
    // Damage with stagger
    public void OnHit(GameplayEffectSpec damageSpec)
    {
        // Check if we should stagger
        var tags = GetAppliedTags();
        if (!tags.Contains(Tags.Status.SuperArmor))
        {
            // Apply stagger
            var staggerSpec = GenerateEffectSpec(null, _staggerEffect);
            ApplyGameplayEffect(staggerSpec);
        }
        
        // Apply damage
        ApplyGameplayEffect(damageSpec);
    }
}
```

## Enemy AI Integration

```csharp
public class EnemyAI : MonoBehaviour, ITarget
{
    private AbilitySystemComponent _asc;
    
    void Update()
    {
        var tags = _asc.GetAppliedTags();
        
        // Can't act while stunned or staggered
        if (tags.Contains(Tags.Status.Stunned) || 
            tags.Contains(Tags.Status.Staggered))
        {
            return;
        }
        
        // Normal AI behavior
        UpdateAI();
    }
    
    // React to status effects
    void OnEffectApplied(GameplayEffectSpec spec)
    {
        if (spec.Base.Tags.GrantedTags.Contains(Tags.Status.Burning))
        {
            // Play burning animation
            _animator.SetBool("OnFire", true);
        }
    }
}
```

## Damage Numbers

```csharp
[Serializable]
public class DamageNumberWorker : AbstractEffectWorker
{
    public GameObject DamageNumberPrefab;
    public Color NormalColor = Color.white;
    public Color CritColor = Color.yellow;
    
    public override void OnEffectApplied(GameplayEffectSpec spec, ITarget target)
    {
        float damage = Mathf.Abs(spec.GetCalculatedMagnitude());
        bool isCrit = spec.IsCritical;
        
        var position = (target as MonoBehaviour)?.transform.position ?? Vector3.zero;
        position += Vector3.up * 2f;
        
        var numberObj = Instantiate(DamageNumberPrefab, position, Quaternion.identity);
        var numberUI = numberObj.GetComponent<DamageNumber>();
        
        numberUI.Show(damage, isCrit ? CritColor : NormalColor);
    }
}
```
