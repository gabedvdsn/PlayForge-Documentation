# Quick Start

Create a damage ability with cost and cooldown in under 5 minutes. But first, lay the ground work

## What We're Building

A **Fireball** ability that:

- Costs 20 mana
- Has a 3-second cooldown  
- Deals 50 damage
- Scales with level

## Step 1: Create the Damage Effect

1. Open Forge via **Tools > PlayForge > Forge**
2. Click **Ability** from the **Create** tab
3. Name it `Effect_Fireball_Damage`
4. Configure:

| Setting | Value |
|---------|-------|
| **Name** | "Fireball Damage" |
| **Asset Tag** | Effect.Damage.Fire |
| **Duration Policy** | Instant |
| **Attribute Target** | Health |
| **Impact Operation** | Add |
| **Magnitude** | -50 |

## Step 2: Create Cost and Cooldown Effects

Next to the **Cost** field, select the

**Cost Effect** (`Effect_Fireball_Cost`):

| Setting | Value |
|---------|-------|
| Duration Policy | Instant |
| Attribute Target | Mana |
| Impact Operation | Add |
| Magnitude | -20 |

**Cooldown Effect** (`Effect_Fireball_Cooldown`):

| Setting | Value |
|---------|-------|
| Duration Policy | Durational |
| Duration | 3.0 |
| Asset Tag | Ability.Fireball.Cooldown |

## Step 3: Create the Ability

1. **Create > PlayForge > Ability**
2. Name it `Ability_Fireball`
3. Configure:

| Setting | Value |
|---------|-------|
| **Name** | "Fireball" |
| **Asset Tag** | Ability.Fireball |
| **Starting Level** | 1 |
| **Max Level** | 5 |
| **Cost** | Effect_Fireball_Cost |
| **Cooldown** | Effect_Fireball_Cooldown |

### Add Behaviour

1. Set **Use Implicit Targeting** to `true`
2. Add a **Stage** with:
   - **Apply Usage Effects**: `true`
   - Add **ApplyEffectsAbilityTask** with `Effect_Fireball_Damage`

## Step 4: Test It

```csharp
public class TestEntity : MonoBehaviour
{
    [SerializeField] private EntityIdentity identity;
    [SerializeField] private Ability fireballAbility;
    
    private AbilitySystemComponent _abilitySystem;
    
    void Start()
    {
        _abilitySystem = GetComponent<AbilitySystemComponent>();
        _abilitySystem.Initialize(identity);
        _abilitySystem.GrantAbility(fireballAbility, level: 1);
    }
    
    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Space))
        {
            if (_abilitySystem.TryActivateAbility(fireballAbility, out _))
                Debug.Log("🔥 Fireball!");
            else
                Debug.Log("❌ On cooldown or no mana");
        }
    }
}
```

## Adding Level Scaling

Make damage scale with level:

1. Open `Effect_Fireball_Damage`
2. Set **Real Magnitude** to `Add Scaler`
3. Add a **Linear Scaler** with values: `[-50, -65, -80, -95, -110]`

Now level 1 deals 50 damage, level 5 deals 110!

---

**Next:** [Project Setup →](project-setup.md)
