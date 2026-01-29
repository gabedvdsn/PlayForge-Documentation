# GameplayEffect

<span class="type-badge class">Class</span>

ScriptableObject that defines an effect template for modifying attributes and granting tags.

## Definition

```csharp
public class GameplayEffect : ScriptableObject
```

**Namespace:** `FarEmerald.PlayForge`

## Properties

### Definition

```csharp
public GameplayEffectDefinition Definition { get; }
```

Name and description.

---

### Tags

```csharp
public GameplayEffectTags Tags { get; }
```

Asset tag, granted tags, requirements.

---

### DurationSpecification

```csharp
public GameplayEffectDurationSpecification DurationSpecification { get; }
```

Duration policy, timing, stacking behavior.

---

### ImpactSpecification

```csharp
public GameplayEffectImpactSpecification ImpactSpecification { get; }
```

Attribute target, operation, magnitude.

---

### Workers

```csharp
public List<AbstractEffectWorker> Workers { get; }
```

Custom behavior extensions.

---

## Related Types

### GameplayEffectDurationSpecification

```csharp
[Serializable]
public class GameplayEffectDurationSpecification
{
    public EEffectDurationPolicy DurationPolicy;
    public float Duration;
    public bool EnablePeriodicTicks;
    public float TickInterval;
    public int Ticks;
    public EEffectReApplicationPolicy ReApplicationPolicy;
    public int StackAmount;
}
```

### EEffectDurationPolicy

```csharp
public enum EEffectDurationPolicy
{
    Instant,     // Applied once
    Durational,  // Timed
    Infinite     // Until removed
}
```

### EEffectReApplicationPolicy

```csharp
public enum EEffectReApplicationPolicy
{
    ReplaceExisting,
    RefreshDuration,
    StackExistingContainers,
    NothingIfExists
}
```

### GameplayEffectImpactSpecification

```csharp
[Serializable]
public class GameplayEffectImpactSpecification
{
    public Attribute AttributeTarget;
    public EEffectImpactTarget TargetImpact;
    public ECalculationOperation ImpactOperation;
    public float Magnitude;
    public AbstractAttributeScaler MagnitudeScaler;
}
```

### EEffectImpactTarget

```csharp
public enum EEffectImpactTarget
{
    Current,
    Base,
    CurrentAndBase
}
```

### ECalculationOperation

```csharp
public enum ECalculationOperation
{
    Add,
    Multiply,
    Override
}
```

---

## Example

```yaml
# Effect_Poison.asset

Definition:
  Name: "Poison"
  Description: "Deals damage over time"

Tags:
  Asset Tag: Effect.Debuff.Poison
  Granted Tags: [Status.Poisoned]

Duration Specification:
  Duration Policy: Durational
  Duration: 10.0
  Enable Periodic Ticks: true
  Tick Interval: 2.0
  Ticks: 5
  Reapplication Policy: RefreshDuration

Impact Specification:
  Attribute Target: Health
  Target Impact: Current
  Impact Operation: Add
  Magnitude: -8
```
