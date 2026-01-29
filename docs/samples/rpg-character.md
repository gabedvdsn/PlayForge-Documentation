# RPG Character Sample

A complete player character implementation with stats, abilities, equipment, and leveling.

## Overview

This sample demonstrates:

- Full attribute set with derivations
- Basic attack and special abilities
- Equipment system integration
- Level-up mechanics
- Health/mana regeneration

## Entity Configuration

### Player Entity

```yaml
# Entity_Player.asset

Name: "Player"
Asset Tag: Entity.Player

Granted Tags:
  - Status.Alive
  - Entity.Player

Attribute Set:
  # Vital Stats
  - Attribute: Health
    Base Value: 100
  - Attribute: HealthMax
    Base Value: 100
    Level Scaler: [100, 120, 145, 175, 210, 250, 295, 345, 400, 460]
  - Attribute: Mana
    Base Value: 50
  - Attribute: ManaMax
    Base Value: 50
    Level Scaler: [50, 60, 72, 86, 102, 120, 140, 162, 186, 212]
    
  # Combat Stats
  - Attribute: Attack
    Base Value: 10
    Level Scaler: [10, 12, 14, 17, 20, 24, 28, 33, 38, 44]
  - Attribute: Defense
    Base Value: 5
    Level Scaler: [5, 6, 7, 9, 11, 13, 15, 18, 21, 24]
  - Attribute: Speed
    Base Value: 1.0
    
  # Progression
  - Attribute: Level
    Base Value: 1
  - Attribute: Experience
    Base Value: 0

Starting Abilities:
  - Ability_BasicAttack (Level 1)
  
Starting Effects:
  - Effect_HealthRegen_Passive
  - Effect_ManaRegen_Passive

Workers:
  - Type: DeathHandlerWorker
    Death Attribute: Health
    Death Tag: Status.Dead
  - Type: LevelUpWorker
    XP Thresholds: [100, 300, 600, 1000, 1500, 2100, 2800, 3600, 4500, 5500]
```

## Abilities

### Basic Attack

```yaml
# Ability_BasicAttack.asset

Name: "Attack"
Activation Policy: Active
Asset Tag: Ability.BasicAttack

Cost: null  # No cost
Cooldown: Effect_BasicAttack_Cooldown  # 1 second

Behaviour:
  Use Implicit Targeting: false
  Targeting: SelectEnemyTask
  
  Stages:
    - Stage Policy: All
      Apply Usage Effects: true
      Tasks:
        - Type: AnimationTask
          Animation: "attack"
        - Type: ApplyEffectsAbilityTask
          Effects: [Effect_BasicAttack_Damage]
```

### Fireball (Special)

```yaml
# Ability_Fireball.asset

Name: "Fireball"
Activation Policy: Active
Asset Tag: Ability.Fireball

Starting Level: 1
Max Level: 5

Cost: Effect_Fireball_Cost  # 25 mana
Cooldown: Effect_Fireball_Cooldown  # 4 seconds

Tag Requirements:
  Source Requirements:
    Require Tags: [Status.Alive]
    Avoid Tags: [Status.Silenced]

Behaviour:
  Targeting: SelectEnemyTask
  
  Stages:
    - Stage Policy: All
      Apply Usage Effects: true
      Tasks:
        - Type: AnimationTask
          Animation: "cast_fire"
        - Type: ApplyEffectsAbilityTask
          Effects: [Effect_Fireball_Damage]
          
Workers:
  - Type: SpawnVFXWorker
    Prefab: VFX_Fireball
```

### Heal Self

```yaml
# Ability_Heal.asset

Name: "Heal"
Activation Policy: Active
Asset Tag: Ability.Heal

Cost: Effect_Heal_Cost  # 30 mana
Cooldown: Effect_Heal_Cooldown  # 8 seconds

Behaviour:
  Use Implicit Targeting: true  # Self-target
  
  Stages:
    - Apply Usage Effects: true
      Tasks:
        - Type: ApplyEffectsAbilityTask
          Effects: [Effect_Heal_Restore]  # +50 health
```

## Effects

### Regeneration

```yaml
# Effect_HealthRegen_Passive.asset

Name: "Health Regeneration"
Duration Policy: Infinite
Enable Periodic Ticks: true
Tick Interval: 1.0

Attribute Target: Health
Impact Operation: Add
Magnitude: 1  # +1 HP/sec
```

### Damage Scaling

```yaml
# Effect_Fireball_Damage.asset

Name: "Fireball Damage"
Duration Policy: Instant
Asset Tag: Effect.Damage.Fire

Attribute Target: Health
Impact Operation: Add
Magnitude: -50
Magnitude Scaler:
  Type: LinearScaler
  Values: [-50, -70, -95, -125, -160]
```

## Integration Code

```csharp
public class RPGPlayer : MonoBehaviour, ISource, ITarget
{
    [SerializeField] private EntityIdentity _playerIdentity;
    [SerializeField] private Ability _basicAttack;
    [SerializeField] private Ability _fireball;
    [SerializeField] private Ability _heal;
    
    private AbilitySystemComponent _abilitySystem;
    private AttributeSystemComponent _attributeSystem;
    
    void Start()
    {
        _abilitySystem = GetComponent<AbilitySystemComponent>();
        _attributeSystem = GetComponent<AttributeSystemComponent>();
        
        // Initialize
        _attributeSystem.Initialize(_playerIdentity);
        _abilitySystem.Initialize(_playerIdentity);
        
        // Grant abilities
        _abilitySystem.GrantAbility(_basicAttack, 1);
        _abilitySystem.GrantAbility(_fireball, 1);
        _abilitySystem.GrantAbility(_heal, 1);
        
        // Listen for level ups
        var levelWorker = _abilitySystem.GetWorker<LevelUpWorker>();
        levelWorker.OnLevelUp += OnLevelUp;
    }
    
    void Update()
    {
        // Basic attack
        if (Input.GetMouseButtonDown(0))
            _abilitySystem.TryActivateAbility(_basicAttack, out _);
            
        // Fireball
        if (Input.GetKeyDown(KeyCode.Alpha1))
            _abilitySystem.TryActivateAbility(_fireball, out _);
            
        // Heal
        if (Input.GetKeyDown(KeyCode.Alpha2))
            _abilitySystem.TryActivateAbility(_heal, out _);
    }
    
    void OnLevelUp(ISource entity, int from, int to)
    {
        Debug.Log($"Level Up! {from} → {to}");
        
        // Level up abilities
        _abilitySystem.SetAbilityLevel(_fireball, to);
    }
    
    // ISource / ITarget implementation...
    public AbilitySystemComponent AsGAS() => _abilitySystem;
    public List<Tag> GetAppliedTags() => _abilitySystem.GetAppliedTags();
    public List<Tag> GetAffiliation() => _playerIdentity.Tags.ContextTags;
    public void ApplyGameplayEffect(GameplayEffectSpec spec) => 
        _abilitySystem.ApplyGameplayEffect(spec);
    public GameplayEffectSpec GenerateEffectSpec(IEffectOrigin origin, GameplayEffect effect) =>
        _abilitySystem.GenerateEffectSpec(origin, effect);
    public bool FindAbilitySystem(out AbilitySystemComponent asc)
    {
        asc = _abilitySystem;
        return true;
    }
}
```

## Equipment Integration

```csharp
public void EquipWeapon(Item weapon)
{
    // Remove old weapon effects
    if (_equippedWeapon != null)
    {
        foreach (var effect in _equippedWeapon.PassiveEffects)
            _abilitySystem.RemoveGameplayEffect(effect);
    }
    
    // Apply new weapon effects
    _equippedWeapon = weapon;
    foreach (var effect in weapon.PassiveEffects)
    {
        var spec = _abilitySystem.GenerateEffectSpec(null, effect);
        _abilitySystem.ApplyGameplayEffect(spec);
    }
    
    // Grant weapon abilities
    foreach (var abilityEntry in weapon.Abilities)
        _abilitySystem.GrantAbility(abilityEntry.Ability, abilityEntry.Level);
}
```
