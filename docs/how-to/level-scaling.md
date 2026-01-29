# Level Scaling

Configure scalers for level-based stat and effect scaling.

## Scaler Types

| Type | Use Case |
|------|----------|
| `LinearScaler` | Explicit values per level |
| `CurveScaler` | Smooth curves via AnimationCurve |
| `FormulaScaler` | Mathematical expressions |
| `RandomizedScaler` | Ranges with variance |

## Linear Scaler

Explicit values for each level:

```yaml
Magnitude: -50

Magnitude Scaler:
  Type: LinearScaler
  Level Values: [-50, -65, -80, -100, -125]
  # Level 1: -50
  # Level 2: -65
  # Level 3: -80
  # Level 4: -100
  # Level 5: -125
```

## Curve Scaler

Smooth interpolation:

```yaml
Magnitude Scaler:
  Type: CurveScaler
  Curve: (AnimationCurve)  # Edit in inspector
  Min Level: 1
  Max Level: 50
  Output Min: -50
  Output Max: -500
```

## Attribute Scaling

Scale base values with level:

```yaml
# In EntityIdentity or AttributeSet
Attribute: HealthMax
Base Value: 100

Level Scaler:
  Type: LinearScaler
  Values: [100, 120, 145, 175, 210, 250]
```

## Effect Duration Scaling

```yaml
Duration: 5.0

Duration Scaler:
  Type: LinearScaler
  Values: [5.0, 6.0, 7.0, 8.0, 10.0]
```

## Cooldown Reduction

```yaml
# Cooldown effect
Duration: 10.0

Duration Scaler:
  Type: LinearScaler
  Values: [10.0, 9.0, 8.0, 7.0, 5.0]  # Decreases with level
```

## Level Provider Linking

Link abilities/effects to item or entity level:

```yaml
# On Item
Starting Level: 1
Max Level: 10

Abilities:
  - Ability: Ability_FlameStrike
    Link Mode: LinkedToProvider  # Uses item level
    
Passive Effects:
  - Effect: Effect_FireDamageBonus
    Link Mode: LinkedToProvider
```

## Custom Scaler

```csharp
[Serializable]
public class ExponentialScaler : AbstractAttributeScaler
{
    public float BaseValue = 10f;
    public float GrowthRate = 1.1f;
    
    public override float GetValueAtLevel(int level)
    {
        return BaseValue * Mathf.Pow(GrowthRate, level - 1);
    }
}
```

## Combining Scalers

Use derivations for complex scaling:

```yaml
# Base damage from weapon
Attribute: WeaponDamage
Level Scaler: LinearScaler [10, 15, 20, 25, 30]

# Final damage = WeaponDamage × AttackMultiplier
Attribute: FinalDamage
Derivations:
  - Type: MultiplyDerivation
    Source A: WeaponDamage
    Source B: AttackMultiplier
```

## Testing Scaling

```csharp
// Test scaler output
var scaler = new LinearScaler { LevelValues = new[] { 10f, 20f, 30f } };

for (int level = 1; level <= 3; level++)
{
    Debug.Log($"Level {level}: {scaler.GetValueAtLevel(level)}");
}
```
