# Tag Requirements

Set up conditional ability activation and effect application.

## Basic Requirements

### Source Requirements
What the caster must have/not have:

```yaml
Source Requirements:
  Require Tags: [Status.Alive]
  Avoid Tags: [Status.Stunned, Status.Silenced]
```

### Target Requirements
What the target must have/not have:

```yaml
Target Requirements:
  Require Tags: [Entity.Enemy]
  Avoid Tags: [Status.Invulnerable, Status.Dead]
```

## Common Patterns

### Only Target Enemies

```yaml
Target Requirements:
  Require Tags: [Entity.Enemy]
  Avoid Tags: [Entity.Player, Entity.Neutral]
```

### Only When Alive

```yaml
Source Requirements:
  Require Tags: [Status.Alive]
  Avoid Tags: [Status.Dead]
```

### Prevent While CC'd

```yaml
Source Requirements:
  Avoid Tags: [Status.CrowdControl]
  # Blocks: Status.CrowdControl.Stun, .Root, .Silence, etc.
```

### Can't Stack Debuff

```yaml
Target Requirements:
  Avoid Tags: [Status.Poisoned]  # Won't apply if already poisoned
```

## Hierarchical Matching

Child tags match parent queries:

```yaml
# This blocks ALL crowd control
Avoid Tags: [Status.CrowdControl]

# This only blocks stuns specifically
Avoid Tags: [Status.CrowdControl.Stun]
```

## Ability Validation Rules

For more complex validation:

```yaml
Source Activation Rules:
  - Type: CooldownValidation
  
  - Type: CostValidation
  
  - Type: AttributeValidation
    Attribute: Health
    Comparison: GreaterThan
    Value: 0
    
  - Type: TagValidation
    Require Tags: [Status.Alive]
    Avoid Tags: [Status.Stunned]

Target Activation Rules:
  - Type: TagValidation
    Require Tags: [Entity.Enemy, Status.Alive]
```

## Execute Ability (Low Health Only)

```yaml
# Ability that only works on low-health targets
Target Activation Rules:
  - Type: AttributeValidation
    Attribute: Health
    Comparison: LessThanPercent
    Value: 25  # Below 25% health
```

## Faction System

```yaml
# Player abilities - only hit enemies
Target Requirements:
  Require Tags: [Entity.Enemy]
  Avoid Tags: [Entity.Player]

# Healing - only allies
Target Requirements:
  Require Tags: [Entity.Player]
  Avoid Tags: [Entity.Enemy]
```

## Combo System

```yaml
# Second hit in combo - requires first hit landed
Source Requirements:
  Require Tags: [Combo.FirstHitLanded]
```

```csharp
// Grant combo tag when first hit lands
void OnFirstHitLanded()
{
    _abilitySystem.GrantTag(Tags.Combo.FirstHitLanded);
    
    // Remove after delay
    StartCoroutine(RemoveComboTag(2f));
}
```

## Stance System

```yaml
# Offensive ability - requires aggressive stance
Source Requirements:
  Require Tags: [Stance.Aggressive]
  Avoid Tags: [Stance.Defensive]
```

## Testing Requirements

```csharp
public bool CanActivate(Ability ability, ISource source, ITarget target)
{
    var sourceReqs = ability.Tags.TagRequirements.SourceRequirements;
    var targetReqs = ability.Tags.TagRequirements.TargetRequirements;
    
    var sourceTags = source.GetAppliedTags();
    var targetTags = target.GetAppliedTags();
    
    bool sourceValid = sourceReqs.Validate(sourceTags);
    bool targetValid = targetReqs.Validate(targetTags);
    
    Debug.Log($"Source valid: {sourceValid}, Target valid: {targetValid}");
    
    return sourceValid && targetValid;
}
```
