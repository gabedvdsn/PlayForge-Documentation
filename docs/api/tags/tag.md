# Tag

<span class="type-badge class">Class</span>

Hierarchical identifier for categorization and requirements.

## Definition

```csharp
public class Tag : ScriptableObject
```

**Namespace:** `FarEmerald.PlayForge`

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `Name` | `string` | Tag name |
| `Parent` | `Tag` | Parent tag in hierarchy |
| `Children` | `List<Tag>` | Child tags |

## Methods

### Contains

```csharp
public bool Contains(Tag tag)
```

Checks if this tag contains another tag hierarchically.

**Returns:** `true` if `tag` is this tag or a descendant.

---

### IsChildOf

```csharp
public bool IsChildOf(Tag parent)
```

Checks if this tag is a descendant of the parent.

**Returns:** `true` if this tag is under `parent` in the hierarchy.

## Example

```csharp
// Check if tag is any status
bool isStatus = Tags.Status.Contains(someTag);

// Check if tag is specifically stunned
bool isStunned = someTag == Tags.Status.Stunned;

// Get all tags on target
var tags = target.GetAppliedTags();
bool hasCrowdControl = tags.Any(t => Tags.Status.CrowdControl.Contains(t));
```
