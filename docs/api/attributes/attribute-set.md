# AttributeSet

<span class="type-badge class">Class</span>

Collection of attributes with initial values.

## Definition

```csharp
[Serializable]
public class AttributeSet
```

**Namespace:** `FarEmerald.PlayForge`

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `Attributes` | `List<AttributeEntry>` | Attribute definitions |

## Related Types

### AttributeEntry

```csharp
[Serializable]
public class AttributeEntry
{
    public Attribute Attribute;
    public float BaseValue;
    public AbstractAttributeScaler LevelScaler;
}
```

## Example

```yaml
Attribute Set:
  - Attribute: Health
    Base Value: 100
  - Attribute: HealthMax
    Base Value: 100
    Level Scaler:
      Type: LinearScaler
      Values: [100, 120, 145, 175, 210]
  - Attribute: Attack
    Base Value: 10
```
