# AttributeSystemComponent

<span class="type-badge class">Class</span>

Manages entity attributes with base/current values, derivations, and constraints.

## Definition

```csharp
public class AttributeSystemComponent : MonoBehaviour
```

**Namespace:** `FarEmerald.PlayForge`

## Methods

### Initialize

```csharp
public void Initialize(EntityIdentity identity)
```

Initializes attributes from an entity identity.

---

### GetCurrentValue

```csharp
public float GetCurrentValue(Attribute attribute)
```

Gets the current calculated value of an attribute.

| Parameter | Type | Description |
|-----------|------|-------------|
| attribute | `Attribute` | The attribute to query |

**Returns:** The current value.

---

### GetBaseValue

```csharp
public float GetBaseValue(Attribute attribute)
```

Gets the underlying base value of an attribute.

---

### SetCurrentValue

```csharp
public void SetCurrentValue(Attribute attribute, float value)
```

Sets the current value directly.

---

### SetBaseValue

```csharp
public void SetBaseValue(Attribute attribute, float value)
```

Sets the base value directly.

---

### ModifyCurrentValue

```csharp
public void ModifyCurrentValue(Attribute attribute, float delta)
```

Adds a delta to the current value.

| Parameter | Type | Description |
|-----------|------|-------------|
| attribute | `Attribute` | Target attribute |
| delta | `float` | Amount to add (negative for subtract) |

---

### ModifyBaseValue

```csharp
public void ModifyBaseValue(Attribute attribute, float delta)
```

Adds a delta to the base value.

---

### HasAttribute

```csharp
public bool HasAttribute(Attribute attribute)
```

Checks if this entity has the specified attribute.

---

### RefreshAllDerivedValues

```csharp
public void RefreshAllDerivedValues()
```

Recalculates all derived attribute values.

---

## Events

### OnAttributeChanged

```csharp
public event Action<Attribute, float, float> OnAttributeChanged;
```

Fired when any attribute value changes.

| Parameter | Type | Description |
|-----------|------|-------------|
| attribute | `Attribute` | Changed attribute |
| oldValue | `float` | Previous value |
| newValue | `float` | New value |

---

### OnBaseValueChanged

```csharp
public event Action<Attribute, float, float> OnBaseValueChanged;
```

Fired when a base value changes.

---

## Example

```csharp
public class HealthDisplay : MonoBehaviour
{
    [SerializeField] private Attribute healthAttribute;
    [SerializeField] private Attribute healthMaxAttribute;
    
    private AttributeSystemComponent _attrSystem;
    
    void Start()
    {
        _attrSystem = GetComponent<AttributeSystemComponent>();
        _attrSystem.OnAttributeChanged += OnAttributeChanged;
        
        UpdateDisplay();
    }
    
    void OnAttributeChanged(Attribute attr, float oldVal, float newVal)
    {
        if (attr == healthAttribute || attr == healthMaxAttribute)
        {
            UpdateDisplay();
        }
        
        if (attr == healthAttribute && newVal <= 0)
        {
            HandleDeath();
        }
    }
    
    void UpdateDisplay()
    {
        float health = _attrSystem.GetCurrentValue(healthAttribute);
        float maxHealth = _attrSystem.GetCurrentValue(healthMaxAttribute);
        
        Debug.Log($"Health: {health}/{maxHealth}");
    }
    
    void HandleDeath()
    {
        Debug.Log("Entity died!");
    }
}
```
