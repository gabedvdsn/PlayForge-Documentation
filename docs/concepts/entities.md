# Entities

**EntityIdentity** assets define complete characters, enemies, and NPCs with attributes, abilities, and tags.

## Overview

An EntityIdentity includes:

- **Definition** — Name, description
- **Attribute Set** — Stats and initial values
- **Starting Abilities** — Abilities granted on spawn
- **Starting Effects** — Effects applied on spawn
- **Tags** — Identity and granted tags

!!! note "Level Provider Asset"
    Entity is a `Level Provider`. For more information, see [Forge/Level Providers](forge.md).

## Creating Entities

1. **Create > PlayForge > Entity Identity**
2. Configure:

```yaml
Name: "Warlock"
Asset Tag: Entity.Warlock
Texture: Warlock_Icon

Context Tags: [Context.Hero]
Granted Tags: []
Affiliation: [Affiliation.Evil]
```

## Configuring Your Entity

Assign an attribute set, starting abilities, and items.

```yaml
AttributeSet: AttributeSet_Default
```

```yaml
Starting Abilities:
  - Ability_BasicAttack
  - Ability_Curse
  - Ability_SummonDemon
  - Ability_Dodge
```

```yaml
Starting Items:
  - Staff of Power
  - Grimoire
```

## Initialization

```csharp
public class GameplayAbilitySystem : LazyMonoProcess, IGameplayAbilitySystem
{
    [SerializeField] private EntityIdentity identity;
    
    private void Initialize(...)
    {
        // Setup component systems
        AbilitySystem.Setup(Data.ActivationPolicy, Data.AllowDuplicateAbilities);
        AttributeSystem.Setup(Data.AttributeSet);
        
        // Initialize support systems
        InitializeEndOfFrameSystem();
        SetupDeferredContexts();
        CollectInitialWorkers();
        
        // Initialize component systems
        AbilitySystem.Initialize(Data.StartingAbilities);
        AttributeSystem.Initialize();
    }    
}
```

## Scene Setup

```
GameObject: "My MonoBehaviour Entity"
└── GameplaytAbilitySystem Component
```