# AvoidRequireTagGroup

<span class="type-badge class">Class</span>

Tag requirement specification with require and avoid lists.

## Definition

```csharp
[Serializable]
public class AvoidRequireTagGroup
```

**Namespace:** `FarEmerald.PlayForge`

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `RequireTags` | `List<Tag>` | Tags that must ALL be present |
| `AvoidTags` | `List<Tag>` | Tags that must ALL be absent |

## Methods

### Validate

```csharp
public bool Validate(List<Tag> tags)
```

Checks if the tag list meets all requirements.

**Returns:** `true` if all required tags present AND no avoided tags present.

### HasAnyRequirements

```csharp
public bool HasAnyRequirements { get; }
```

Returns `true` if any requirements are defined.

### TotalCount

```csharp
public int TotalCount { get; }
```

Gets total number of requirements (require + avoid).

## Example

```yaml
Source Requirements:
  Require Tags: [Status.Alive]
  Avoid Tags: [Status.Stunned, Status.Silenced]
```

```csharp
var requirements = new AvoidRequireTagGroup
{
    RequireTags = new List<Tag> { Tags.Status.Alive },
    AvoidTags = new List<Tag> { Tags.Status.Stunned }
};

var entityTags = target.GetAppliedTags();
bool canApply = requirements.Validate(entityTags);
```
