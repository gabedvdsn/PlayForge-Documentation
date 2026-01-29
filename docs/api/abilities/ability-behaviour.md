# AbilityBehaviour

<span class="type-badge class">Class</span>

Defines ability targeting and execution stages.

## Definition

```csharp
[Serializable]
public class AbilityBehaviour
```

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `Targeting` | `AbstractAbilityRelatedTask` | Optional targeting task |
| `UseImplicitTargeting` | `bool` | Auto-target self |
| `Stages` | `List<AbilityTaskBehaviourStage>` | Execution stages |

## Related Types

### AbilityTaskBehaviourStage

```csharp
[Serializable]
public class AbilityTaskBehaviourStage
{
    public EAbilityStageBehaviourPolicy StagePolicy;
    public bool ApplyUsageEffects;
    public List<AbstractAbilityTask> Tasks;
}
```

### EAbilityStageBehaviourPolicy

```csharp
public enum EAbilityStageBehaviourPolicy
{
    All,        // Wait for all tasks
    Any,        // Continue when any completes
    Maintained  // Stay active until cancelled
}
```
