# Integrating PlayForge

How to integrate PlayForge into your Unity project with proper scene setup and initialization.

## Required Components

Every PlayForge entity needs two components:

| Component | Purpose |
|-----------|---------|
| `AbilitySystemComponent` | Manages abilities, effects, tags |
| `AttributeSystemComponent` | Manages attributes and values |

```
GameObject: "Player"
├── AbilitySystemComponent
├── AttributeSystemComponent
└── YourController
```

## Core Interfaces

### IGameplayAbilitySystem

Primary interface for entities with full GAS capabilities:

```csharp
public interface IGameplayAbilitySystem : ISource, IProxyTaskBehaviourCaller
{
    AttributeSystemComponent GetAttributeSystem();
    AbilitySystemComponent GetAbilitySystem();
    AnalysisWorkerCache GetAnalysisCache();
}
```

### ISource

Entity that can apply effects and activate abilities:

```csharp
public interface ISource : ITarget, IGameplayProcessHandler
{
    List<Tag> GetContextTags();
    TagCache GetTagCache();
    FrameSummary GetFrameSummary();
    GameplayEffectSpec GenerateEffectSpec(IEffectOrigin origin, GameplayEffect effect);
}
```

### ITarget

Entity that can receive effects:

```csharp
public interface ITarget : ITagHandler, IValidationReady
{
    IGameplayAbilitySystem AsGAS();
    List<Tag> GetAffiliation();
    void ApplyGameplayEffect(GameplayEffectSpec spec);
    bool TryModifyAttribute(Attribute attr, SourcedModifiedAttributeValue value, bool runEvents);
}
```

## Basic Implementation

### Entity Controller

```csharp
public class EntityController : MonoBehaviour, ISource, ITarget
{
    [SerializeField] private EntityIdentity identity;
    
    private AbilitySystemComponent _abilitySystem;
    private AttributeSystemComponent _attributeSystem;
    
    void Awake()
    {
        _attributeSystem = GetComponent<AttributeSystemComponent>();
        _abilitySystem = GetComponent<AbilitySystemComponent>();
    }
    
    void Start()
    {
        Initialize();
    }
    
    public void Initialize()
    {
        // Initialize systems with identity
        _attributeSystem.Initialize(identity);
        _abilitySystem.Initialize(identity);
        
        // Grant starting abilities
        foreach (var entry in identity.StartingAbilities)
        {
            _abilitySystem.GrantAbility(entry.Ability, entry.Level);
        }
        
        // Apply starting effects
        foreach (var effect in identity.StartingEffects)
        {
            var spec = GenerateEffectSpec(null, effect);
            ApplyGameplayEffect(spec);
        }
    }
    
    // ITarget Implementation
    public IGameplayAbilitySystem AsGAS() => _abilitySystem;
    
    public List<Tag> GetAffiliation() => identity.Tags.ContextTags;
    
    public void ApplyGameplayEffect(GameplayEffectSpec spec)
    {
        _abilitySystem.ApplyGameplayEffect(spec);
    }
    
    // ISource Implementation
    public List<Tag> GetContextTags() => identity.Tags.ContextTags;
    
    public GameplayEffectSpec GenerateEffectSpec(IEffectOrigin origin, GameplayEffect effect)
    {
        return _abilitySystem.GenerateEffectSpec(origin, effect);
    }
    
    // Additional interface methods...
}
```

## Initialization Order

!!! warning "Order Matters"
Always initialize AttributeSystem before AbilitySystem.

```csharp
void Start()
{
    // 1. Attributes first
    _attributeSystem.Initialize(identity);
    
    // 2. Abilities second (may depend on attributes)
    _abilitySystem.Initialize(identity);
    
    // 3. Starting abilities
    foreach (var entry in identity.StartingAbilities)
        _abilitySystem.GrantAbility(entry.Ability, entry.Level);
    
    // 4. Starting effects last
    foreach (var effect in identity.StartingEffects)
        ApplyGameplayEffect(GenerateEffectSpec(null, effect));
}
```

## Activating Abilities

```csharp
// Check and activate
if (_abilitySystem.TryActivateAbility(ability, out var proxy))
{
    Debug.Log($"Activated: {ability.GetName()}");
}

// With target data
var targetData = new AbilityTargetData { Target = enemy };
if (_abilitySystem.TryActivateAbility(ability, targetData, out var proxy))
{
    // Ability activated with target
}

// Cancel
_abilitySystem.CancelAbility(ability);
```

## Applying Effects

```csharp
// Generate spec with context
var spec = source.GenerateEffectSpec(origin, damageEffect);

// Apply to target
target.ApplyGameplayEffect(spec);

// Remove by effect
target.AsGAS().RemoveGameplayEffect(damageEffect);

// Remove by tag
target.AsGAS().RemoveEffectsByTag(Tags.Effect.Debuff);
```

## Reading Attributes

```csharp
var attrSystem = _abilitySystem.GetAttributeSystem();

// Get current value
if (attrSystem.TryGetAttributeValue(healthAttr, out var value))
{
    float current = value.CurrentValue;
    float baseVal = value.BaseValue;
}

// Check specific attribute
float health = attrSystem.GetCurrentValue(healthAttr);
```

## Listening to Events

### Attribute Changes

```csharp
_attributeSystem.Callbacks.OnAttributeChanged += (attr, oldVal, newVal) =>
{
    if (attr == healthAttr && newVal.CurrentValue <= 0)
    {
        HandleDeath();
    }
};
```

### Ability Events

```csharp
_abilitySystem.Callbacks.AbilityActivated += status =>
{
    Debug.Log($"Activated: {status.Ability.GetName()}");
};

_abilitySystem.Callbacks.AbilityEnded += status =>
{
    Debug.Log($"Ended: {status.Ability.GetName()}");
};
```

### Effect Events

```csharp
_abilitySystem.Callbacks.EffectApplied += spec =>
{
    Debug.Log($"Applied: {spec.Base.GetName()}");
};

_abilitySystem.Callbacks.EffectRemoved += spec =>
{
    Debug.Log($"Removed: {spec.Base.GetName()}");
};
```

## Tag Queries

```csharp
// Get all tags
List<Tag> tags = target.GetAppliedTags();

// Check tag weight
int stacks = target.GetTagWeight(Tags.Status.Burning);

// Query with condition
var query = new TagQuery
{
    Tag = Tags.Status.Stunned,
    Operator = EComparisonOperator.GreaterThan,
    Magnitude = 0
};
bool isStunned = target.QueryTags(query);
```

## Scene Setup Checklist

1. **Add Components**
    - AbilitySystemComponent
    - AttributeSystemComponent

2. **Assign EntityIdentity**
    - Reference in your controller

3. **Implement Interfaces**
    - ISource for full entities
    - ITarget for effect receivers only

4. **Initialize in Order**
    - AttributeSystem → AbilitySystem → Abilities → Effects

5. **Subscribe to Events**
    - Attribute changes
    - Ability activation
    - Effect application

## Minimal Setup (Target Only)

For entities that only receive effects (no abilities):

```csharp
public class EffectReceiver : MonoBehaviour, ITarget
{
    private AttributeSystemComponent _attributeSystem;
    
    public IGameplayAbilitySystem AsGAS() => null;
    
    public void ApplyGameplayEffect(GameplayEffectSpec spec)
    {
        // Handle effect application
        foreach (var impact in spec.GetImpacts())
        {
            _attributeSystem.ModifyAttribute(
                impact.Attribute, 
                impact.ToSourcedValue(spec)
            );
        }
    }
    
    public List<Tag> GetAffiliation() => new List<Tag>();
}
```

## Common Patterns

### Player Controller

```csharp
public class PlayerController : EntityController
{
    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Alpha1))
            TryActivateAbility(ability1);
            
        if (Input.GetKeyDown(KeyCode.Alpha2))
            TryActivateAbility(ability2);
    }
    
    void TryActivateAbility(Ability ability)
    {
        if (_abilitySystem.TryActivateAbility(ability, out _))
            PlayActivationFeedback(ability);
    }
}
```

### Enemy AI

```csharp
public class EnemyAI : EntityController
{
    void DecideAction()
    {
        // Check if can use ability
        if (_abilitySystem.CanActivateAbility(specialAttack))
        {
            var target = FindTarget();
            _abilitySystem.TryActivateAbility(specialAttack, 
                new AbilityTargetData { Target = target }, out _);
        }
    }
}
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Effects not applying | Check tag requirements |
| Attributes not updating | Verify initialization order |
| Abilities failing | Check cost/cooldown validation |
| Null reference | Ensure components assigned |