# AbilitySystemComponent

<span class="type-badge class">Class</span>

Central component for managing abilities and effects on an entity.

## Definition

```csharp
public class AbilitySystemComponent : MonoBehaviour
```

**Namespace:** `FarEmerald.PlayForge`

## Properties

### Callbacks

```csharp
public AbilityCallbacks Callbacks { get; }
```

Event callbacks for ability lifecycle events.

---

## Methods

### Initialize

```csharp
public void Initialize(EntityIdentity identity)
```

Initializes the ability system with an entity identity.

| Parameter | Type | Description |
|-----------|------|-------------|
| identity | `EntityIdentity` | The entity configuration |

---

### GrantAbility

```csharp
public void GrantAbility(Ability ability, int level = 1)
```

Grants an ability to this entity.

| Parameter | Type | Description |
|-----------|------|-------------|
| ability | `Ability` | The ability to grant |
| level | `int` | Initial ability level |

---

### RemoveAbility

```csharp
public void RemoveAbility(Ability ability)
```

Removes a granted ability.

---

### HasAbility

```csharp
public bool HasAbility(Ability ability)
```

Checks if the ability is granted.

**Returns:** `true` if the ability is granted.

---

### TryActivateAbility

```csharp
public bool TryActivateAbility(Ability ability, out AbilityProxy proxy)
```

Attempts to activate an ability.

| Parameter | Type | Description |
|-----------|------|-------------|
| ability | `Ability` | The ability to activate |
| proxy | `out AbilityProxy` | The execution proxy if successful |

**Returns:** `true` if activation succeeded.

---

### CancelAbility

```csharp
public void CancelAbility(Ability ability)
```

Cancels an active ability.

---

### GetAbilityLevel

```csharp
public int GetAbilityLevel(Ability ability)
```

Gets the current level of a granted ability.

---

### SetAbilityLevel

```csharp
public void SetAbilityLevel(Ability ability, int level)
```

Sets the level of a granted ability.

---

### ApplyGameplayEffect

```csharp
public void ApplyGameplayEffect(GameplayEffectSpec spec)
```

Applies a gameplay effect to this entity.

---

### RemoveGameplayEffect

```csharp
public void RemoveGameplayEffect(GameplayEffect effect)
```

Removes all instances of an effect.

---

### RemoveEffectsByTag

```csharp
public void RemoveEffectsByTag(Tag tag)
```

Removes all effects matching a tag.

---

### GenerateEffectSpec

```csharp
public GameplayEffectSpec GenerateEffectSpec(IEffectOrigin origin, GameplayEffect effect)
```

Creates a new effect spec with this entity as source.

---

### GetAppliedTags

```csharp
public List<Tag> GetAppliedTags()
```

Gets all tags currently applied to this entity.

---

### GetAttributeValue

```csharp
public float GetAttributeValue(Attribute attribute)
```

Gets the current value of an attribute.

---

## Events

### AbilityCallbacks

```csharp
public class AbilityCallbacks
{
    public event Action<AbilityCallbackStatus> AbilityActivated;
    public event Action<AbilityCallbackStatus> AbilityStageActivated;
    public event Action<AbilityCallbackStatus> AbilityEnded;
    public event Action<AbilityCallbackStatus> AbilityCancelled;
}
```

---

## Example

```csharp
public class PlayerAbilities : MonoBehaviour
{
    [SerializeField] private EntityIdentity identity;
    [SerializeField] private Ability fireball;
    
    private AbilitySystemComponent _asc;
    
    void Start()
    {
        _asc = GetComponent<AbilitySystemComponent>();
        _asc.Initialize(identity);
        _asc.GrantAbility(fireball, 1);
        
        _asc.Callbacks.AbilityActivated += OnAbilityActivated;
    }
    
    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Alpha1))
        {
            _asc.TryActivateAbility(fireball, out _);
        }
    }
    
    void OnAbilityActivated(AbilityCallbackStatus status)
    {
        Debug.Log($"Activated: {status.Ability.GetName()}");
    }
}
```
