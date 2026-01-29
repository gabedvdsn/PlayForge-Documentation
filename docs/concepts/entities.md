# Entities

**EntityIdentity** assets define complete characters, enemies, and NPCs with attributes, abilities, and tags.

## Overview

An EntityIdentity includes:

- **Definition** — Name, description
- **Attribute Set** — Stats and initial values
- **Starting Abilities** — Abilities granted on spawn
- **Starting Effects** — Effects applied on spawn
- **Tags** — Identity and granted tags

## Creating Entities

1. **Create > PlayForge > Entity Identity**
2. Configure:

```yaml
Name: "Player"
Asset Tag: Entity.Player
Granted Tags: [Status.Alive]
```

## Attribute Set

```yaml
Attribute Set:
  - Attribute: Health
    Base Value: 100
  - Attribute: HealthMax
    Base Value: 100
  - Attribute: Mana
    Base Value: 50
  - Attribute: Attack
    Base Value: 10
  - Attribute: Level
    Base Value: 1
```

## Starting Abilities

```yaml
Starting Abilities:
  - Ability: Ability_BasicAttack
    Level: 1
  - Ability: Ability_Dodge
    Level: 1
```

## Starting Effects

```yaml
Starting Effects:
  - Effect_HealthRegen_Passive
  - Effect_ManaRegen_Passive
```

## Scene Setup

```
GameObject: "Player"
├── AbilitySystemComponent
├── AttributeSystemComponent
└── PlayerController
```

## Initialization

```csharp
public class EntityController : MonoBehaviour, ISource, ITarget
{
    [SerializeField] private EntityIdentity identity;
    
    private AbilitySystemComponent _abilitySystem;
    private AttributeSystemComponent _attributeSystem;
    
    void Start()
    {
        _attributeSystem = GetComponent<AttributeSystemComponent>();
        _abilitySystem = GetComponent<AbilitySystemComponent>();
        
        // Initialize attributes first
        _attributeSystem.Initialize(identity);
        
        // Then abilities
        _abilitySystem.Initialize(identity);
        
        // Grant starting abilities
        foreach (var entry in identity.StartingAbilities)
            _abilitySystem.GrantAbility(entry.Ability, entry.Level);
            
        // Apply starting effects
        foreach (var effect in identity.StartingEffects)
        {
            var spec = _abilitySystem.GenerateEffectSpec(null, effect);
            _abilitySystem.ApplyGameplayEffect(spec);
        }
    }
    
    // ITarget
    public AbilitySystemComponent AsGAS() => _abilitySystem;
    public List<Tag> GetAppliedTags() => _abilitySystem.GetAppliedTags();
    public void ApplyGameplayEffect(GameplayEffectSpec spec) => 
        _abilitySystem.ApplyGameplayEffect(spec);
    
    // ISource
    public List<Tag> GetAffiliation() => identity.Tags.ContextTags;
    public GameplayEffectSpec GenerateEffectSpec(IEffectOrigin origin, GameplayEffect effect) =>
        _abilitySystem.GenerateEffectSpec(origin, effect);
}
```

## Common Entity Types

### Player
```yaml
Name: "Player"
Asset Tag: Entity.Player
Granted Tags: [Status.Alive, Entity.Player]

Attributes: [Health: 100, Mana: 50, Attack: 10]
Abilities: [BasicAttack, Dodge]
Effects: [HealthRegen, ManaRegen]
```

### Enemy
```yaml
Name: "Goblin"
Asset Tag: Entity.Enemy.Goblin
Granted Tags: [Status.Alive, Entity.Enemy]

Attributes: [Health: 30, Attack: 5]
Abilities: [BasicAttack]
```

### Boss
```yaml
Name: "Dragon"
Asset Tag: Entity.Enemy.Boss.Dragon
Granted Tags: [Status.Alive, Status.Immune.CrowdControl]

Attributes: [Health: 10000, Attack: 150]
Abilities: [DragonBreath, TailSwipe, Fly]
```
