# Create Your First Ability

This guide walks through creating a complete ability with damage, cost, and cooldown.

!!! note "Creating Abilities"
    In this chapter, we will cover how to efficiently create abilities. Abilities fall under **Step 3: Effect Origins** in the [Asset Workflow](../concepts/asset-workflow.md).

## Goal

Create a **Fireball** ability that:

- Costs 25 mana
- Has a 4-second cooldown
- Deals 60 fire damage
- Scales damage with level

## Step 1: Create the Damage Effect

**Create > PlayForge > Effect** → `Effect_Fireball_Damage`

```yaml
# Definition
Name: "Fireball Damage"
Description: "Deals fire damage"

# Tags
Asset Tag: Effect.Damage.Fire.Fireball

# Duration
Duration Policy: Instant

# Impact
Attribute Target: Health
Target Impact: Current
Impact Operation: Add
Magnitude: -60

# Scaling (optional)
Real Magnitude: Add Scaler
Magnitude Scaler:
  Type: LinearScaler
  Level Values: [-60, -75, -90, -110, -130]
```

## Step 2: Create the Cost Effect

**Create > PlayForge > Effect** → `Effect_Fireball_Cost`

```yaml
Name: "Fireball Cost"
Duration Policy: Instant

Attribute Target: Mana
Impact Operation: Add
Magnitude: -25
```

## Step 3: Create the Cooldown Effect

**Create > PlayForge > Effect** → `Effect_Fireball_Cooldown`

```yaml
Name: "Fireball Cooldown"
Asset Tag: Ability.Fireball.Cooldown

Duration Policy: Durational
Duration: 4.0
```

## Step 4: Create the Ability

**Create > PlayForge > Ability** → `Ability_Fireball`

```yaml
# Definition
Name: "Fireball"
Description: "Hurls a ball of fire"
Activation Policy: Active

# Tags
Asset Tag: Ability.Fireball
Context Tags: [Ability.Type.Spell, Element.Fire]

# Levels
Starting Level: 1
Max Level: 5

# Cost & Cooldown
Cost: Effect_Fireball_Cost
Cooldown: Effect_Fireball_Cooldown
```

## Step 5: Configure Behaviour

In the **Behaviour** section:

1. Set **Use Implicit Targeting** to `true` (or add targeting task)
2. Add a Stage:

```yaml
Stage 0:
  Stage Policy: All
  Apply Usage Effects: true
  
  Tasks:
    - Type: ApplyEffectsAbilityTask
      Effects: [Effect_Fireball_Damage]
```

## Step 6: Add Tag Requirements (Optional)

```yaml
Tag Requirements:
  Source Requirements:
    Require Tags: [Status.Alive]
    Avoid Tags: [Status.Silenced, Status.Stunned]
```

## Step 7: Test

```csharp
public class AbilityTest : MonoBehaviour
{
    [SerializeField] private Ability fireball;
    private AbilitySystemComponent _asc;
    
    void Start()
    {
        _asc = GetComponent<AbilitySystemComponent>();
        _asc.GrantAbility(fireball, 1);
    }
    
    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Alpha1))
        {
            if (_asc.TryActivateAbility(fireball, out _))
                Debug.Log("🔥 Fireball!");
            else
                Debug.Log("Cannot cast");
        }
    }
}
```

## Result

You now have a complete ability with:

- ✅ Mana cost (25)
- ✅ Cooldown (4s)
- ✅ Damage (60-130 based on level)
- ✅ Tag requirements
- ✅ Level scaling

## Next Steps

- Add [visual effects with workers](custom-workers.md)
- Add [targeting](tag-requirements.md) for non-self abilities
- Create [buff effects](buff-system.md)
