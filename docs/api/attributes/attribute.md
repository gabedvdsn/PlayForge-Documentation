# Attribute

<span class="type-badge class">Class</span>

ScriptableObject defining a numeric attribute.

## Definition

```csharp
public class Attribute : ScriptableObject
```

**Namespace:** `FarEmerald.PlayForge`

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `Name` | `string` | Display name |
| `ShortName` | `string` | Abbreviated name |
| `Icon` | `Sprite` | UI icon |
| `Constraints` | `AttributeConstraints` | Min/max bounds |

## Related Types

### AttributeConstraints

```csharp
[Serializable]
public class AttributeConstraints
{
    public float? MinimumValue;
    public float? MaximumValue;
    public Attribute MinimumSource;
    public Attribute MaximumSource;
    public ERoundingMode RoundingMode;
}
```

### ERoundingMode

```csharp
public enum ERoundingMode
{
    None,
    Floor,
    Ceil,
    Round,
    RoundToInt
}
```
