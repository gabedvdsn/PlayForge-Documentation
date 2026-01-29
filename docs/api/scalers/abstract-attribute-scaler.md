# AbstractAttributeScaler

<span class="type-badge abstract">Abstract Class</span>

Base class for level-based value scaling.

## Definition

```csharp
[Serializable]
public abstract class AbstractAttributeScaler
```

**Namespace:** `FarEmerald.PlayForge`

## Methods

### GetValueAtLevel

```csharp
public abstract float GetValueAtLevel(int level)
```

Gets the scaled value for a given level.

| Parameter | Type | Description |
|-----------|------|-------------|
| level | `int` | The level to calculate for |

**Returns:** The calculated value at that level.

## Implementations

### LinearScaler

Explicit values per level.

```csharp
[Serializable]
public class LinearScaler : AbstractAttributeScaler
{
    public float[] LevelValues;
    
    public override float GetValueAtLevel(int level)
    {
        int index = Mathf.Clamp(level - 1, 0, LevelValues.Length - 1);
        return LevelValues[index];
    }
}
```

```yaml
Type: LinearScaler
Level Values: [50, 65, 80, 100, 125]
```

### CurveScaler

Interpolation via AnimationCurve.

```csharp
[Serializable]
public class CurveScaler : AbstractAttributeScaler
{
    public AnimationCurve Curve;
    public int MinLevel = 1;
    public int MaxLevel = 50;
    public float OutputMin;
    public float OutputMax;
}
```

```yaml
Type: CurveScaler
Curve: (AnimationCurve)
Min Level: 1
Max Level: 50
Output Min: 50
Output Max: 500
```

### RandomizedScaler

Range with variance.

```csharp
[Serializable]
public class RandomizedScaler : AbstractAttributeScaler
{
    public float BaseValue;
    public float Variance;  // Percentage or absolute
}
```

## Creating Custom Scalers

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
