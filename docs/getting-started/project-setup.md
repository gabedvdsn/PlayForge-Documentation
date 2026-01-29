# Project Setup

Best practices for organizing your PlayForge project.

## Folder Structure

```
Assets/
├── _Game/
│   ├── Data/
│   │   ├── Abilities/
│   │   ├── Effects/
│   │   ├── Entities/
│   │   ├── Items/
│   │   ├── Attributes/
│   │   └── Tags/
│   ├── Prefabs/
│   └── Scripts/
└── PlayForge/
```

## Naming Conventions

| Type | Pattern | Example |
|------|---------|---------|
| Ability | `Ability_Name` | `Ability_Fireball` |
| Effect | `Effect_Category_Name` | `Effect_Damage_Fire` |
| Entity | `Entity_Type_Name` | `Entity_Enemy_Goblin` |
| Item | `Item_Category_Name` | `Item_Weapon_Sword` |
| Attribute | `Attribute_Name` | `Attribute_Health` |

## Tag Hierarchy

Set up a clear tag hierarchy early:

```
Ability
├── Ability.Active
├── Ability.Passive
└── Ability.Triggered

Effect
├── Effect.Damage
│   ├── Effect.Damage.Physical
│   └── Effect.Damage.Fire
├── Effect.Buff
└── Effect.Debuff

Status
├── Status.Alive
├── Status.Dead
├── Status.Stunned
└── Status.Burning

Entity
├── Entity.Player
├── Entity.Enemy
└── Entity.NPC
```

## Core Attributes

Create these essential attributes:

```csharp
// Vital Stats
Health, HealthMax, Mana, ManaMax

// Combat
Attack, Defense, Speed, CritChance

// Progression  
Level, Experience
```

## Entity Setup

Every entity needs:

```
GameObject
├── AbilitySystemComponent
├── AttributeSystemComponent
└── YourEntityController
```

```csharp
public class EntityController : MonoBehaviour
{
    [SerializeField] private EntityIdentity identity;
    
    void Start()
    {
        var attrSystem = GetComponent<AttributeSystemComponent>();
        var abilitySystem = GetComponent<AbilitySystemComponent>();
        
        // Initialize attributes first
        attrSystem.Initialize(identity);
        
        // Then abilities
        abilitySystem.Initialize(identity);
    }
}
```

## PlayForge Manager

Open **Window > PlayForge > Manager** to:

- Browse all PlayForge assets
- Validate asset configurations
- Analyze balance metrics
- Quick-edit assets

---

**Next:** [Core Concepts →](../concepts/index.md)
