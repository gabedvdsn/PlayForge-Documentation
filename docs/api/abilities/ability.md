# Ability

<span class="type-badge class">Class</span>

ScriptableObject that defines an ability template.

## Definition

```csharp
public class Ability : ScriptableObject, ILevelProvider
```

**Namespace:** `FarEmerald.PlayForge`

## Properties

### Definition

```csharp
public AbilityDefinition Definition { get; }
```

Name, description, activation policy.

---

### Tags

```csharp
public AbilityTags Tags { get; }
```

Asset tag, context tags, granted tags, requirements.

---

### Cost

```csharp
public GameplayEffect Cost { get; }
```

Effect applied as ability cost.

---

### Cooldown

```csharp
public GameplayEffect Cooldown { get; }
```

Effect applied as cooldown tracker.

---

### Behaviour

```csharp
public AbilityBehaviour Behaviour { get; }
```

Targeting and execution stages.

---

### StartingLevel

```csharp
public int StartingLevel { get; }
```

Initial level when granted.

---

### MaxLevel

```csharp
public int MaxLevel { get; }
```

Maximum achievable level.

---

## Methods

### GetName

```csharp
public string GetName()
```

Gets the display name.

---

### GetDescription

```csharp
public string GetDescription()
```

Gets the description text.

---

### GetLevel / SetLevel

```csharp
public int GetLevel()
public void SetLevel(int level)
```

ILevelProvider implementation.

---

## Related Types

### AbilityDefinition

```csharp
[Serializable]
public class AbilityDefinition
{
    public string Name;
    public string Description;
    public EAbilityActivationPolicy ActivationPolicy;
}
```

### EAbilityActivationPolicy

```csharp
public enum EAbilityActivationPolicy
{
    Active,     // Manually activated
    Passive,    // Always active
    Triggered   // Event-based
}
```

---

## Example

```yaml
# Ability_Fireball.asset

Definition:
  Name: "Fireball"
  Description: "Hurls a ball of fire"
  Activation Policy: Active

Tags:
  Asset Tag: Ability.Fireball
  Context Tags: [Ability.Type.Spell, Element.Fire]

Cost: Effect_Fireball_Cost
Cooldown: Effect_Fireball_Cooldown

Starting Level: 1
Max Level: 5

Behaviour:
  Use Implicit Targeting: true
  Stages:
    - Stage Policy: All
      Apply Usage Effects: true
      Tasks:
        - Type: ApplyEffectsAbilityTask
          Effects: [Effect_Fireball_Damage]
```
