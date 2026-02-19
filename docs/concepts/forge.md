# The Forge

**The Forge** is PlayForge's custom editor suite for creating, viewing, analyzing, and configuring assets. Access the Forge via **Tools → PlayForge → Forge**.

## Overview

The Forge provides four main tabs:

| Tab | Purpose |
|-----|---------|
| **Create** | Create new assets with templates |
| **View** | Browse and search all PlayForge assets |
| **Analysis** | Compare power balance and complexity |
| **Settings** | Configure paths, templates, and preferences |

## Create Tab

Quickly create new PlayForge assets:

```yaml
Asset Type: Ability
Name: "Fireball"
Template: ★ Combat Ability (default)
Path: Assets/PlayForge/Abilities/
```

### Templates

Templates pre-configure new assets with common settings:

- **★ Default Template** — Used automatically when creating assets
- **Tagged Templates** — Organized by category (Offensive, Defensive, etc.)

Configure templates in **Settings → Assets → [Asset Type] → Templates**.

### Recent Assets

Shows recently created assets for quick access. Configure row count in Settings.

## View Tab

Browse all PlayForge assets with filtering and search.

### Asset Type Filters

| Icon | Type |
|------|------|
| ⚡ | Ability |
| ✦ | Effect |
| ◈ | Attribute |
| ▣ | AttributeSet |
| ◎ | Entity |
| ♦ | Item |

### View Modes

- **List View** — Flat list of all assets
- **Grouped View** — Organized by folder or type
- **Search** — Filtered by name

### Actions

| Action | Result |
|--------|--------|
| Single-click | Select asset |
| Double-click | Open Visualizer (configurable) |
| Right-click | Context menu |

## Analysis Tab

Evaluate balance and complexity across your project.

### Effect Analysis

| Metric | Description |
|--------|-------------|
| **Power Score** | Calculated impact (magnitude × duration × frequency) |
| **Percentile** | Comparison to other effects |
| **Duration** | Instant, Durational, or Infinite |
| **Stacks** | Maximum stack count |

### Ability Analysis

| Metric | Description |
|--------|-------------|
| **Effect Power** | Sum of all effect power scores |
| **Effect Count** | Number of applied effects |
| **Cost Score** | Resource cost evaluation |
| **Cooldown** | Time between activations |
| **Complexity** | Stages, tasks, and rules |

### Warnings

The analyzer flags potential issues:

| Warning | Condition |
|---------|-----------|
| ⚠️ Overpowered | Above 95th percentile |
| ⚠️ Underpowered | Below 5th percentile |
| ⚠️ High Complexity | Too many stages/tasks |
| ⚠️ No Impact | Negligible power score |

### Presets

Quick analysis configurations:

- **Balance Check** — Focus on power distribution
- **Complexity Audit** — Identify complex abilities
- **Threat Assessment** — Evaluate damage potential

## Settings Tab

### General

| Setting | Description |
|---------|-------------|
| **Remember Type Filter** | Restore last filter on reopen |
| **Double-click Visualize** | Open visualizer vs select |
| **Auto-refresh** | Refresh cache on window focus |
| **Recently Created Rows** | Number of recent assets (0-50) |

### Assets

Per-asset-type configuration:

**Paths:**
```yaml
Abilities:    Assets/Data/Abilities/
Effects:      Assets/Data/Effects/
Attributes:   Assets/Data/Attributes/
```

**Templates:**
```yaml
Templates:
  - ★ Combat Ability (default)
  - [Offensive] Damage Spell
  - [Defensive] Shield Buff
```

**Type-Specific:**

| Asset | Settings |
|-------|----------|
| Ability | Default cost/cooldown templates |

### Tag Registry

- **Refresh Tag Registry** — Rescan project for Tags
- View statistics (unique tags, contexts, last scan)

## Level Providers

Certain assets can link to level providers in order to derive level bounds. This has 2 effects:
1. Linked level providers override any starting and max level fields
2. Scaler level bounds become locked to these derived values (optional)

## Visualizer

The **Visualizer** provides detailed visual breakdown of any asset:

- Asset hierarchy and relationships
- Effect chains and dependencies
- Tag requirements and grants
- Level scaling preview

Open via:

- Double-click in View tab
- **◉** button in any PlayForge inspector
- **Window → PlayForge → Visualizer**

## Inspector Integration

All PlayForge inspectors include a header with quick actions:

| Button | Action                     |
|--------|----------------------------|
| **?**  | Open documentation         |
| **↻**  | Refresh inspector          |
| **↓**  | Import from template/asset |
| **◉**  | Open Visualizer            |
| **⊞**  | Open in Forge Manager      |