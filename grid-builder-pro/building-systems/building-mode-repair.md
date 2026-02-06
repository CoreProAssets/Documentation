# Building mode repair

## Overview

The `BuildingModeRepair` class implements the `IBuildingMode` interface for repairing damaged buildings in GridBuilderPro. This mode allows players to restore buildings to full health by paying repair costs. The mode provides visual feedback for damaged buildings and handles the complete repair transaction including resource deduction.



## Key Responsibilities

| Responsibility         | Description                                     |
| ---------------------- | ----------------------------------------------- |
| **Damage Detection**   | Identifies buildings that need repair           |
| **Repair Calculation** | Calculates repair costs based on missing health |
| **Resource Deduction** | Deducts repair costs from player economy        |
| **Health Restoration** | Restores building health to maximum             |
| **Visual Feedback**    | Highlights damaged buildings                    |

## Configuration Properties

| Property                | Type  | Description                          |
| ----------------------- | ----- | ------------------------------------ |
| `repairHighlightColor`  | Color | Color for damaged building highlight |
| `showRepairCostPreview` | bool  | Show repair cost on hover            |
| `requireFullRepair`     | bool  | Require full repair or allow partial |

## Repair Calculation

The repair system calculates costs based on missing health:

```csharp
public ResourceAmount[] CalculateRepairCost(BuildingInstance building)
{
    BuildingData data = building.GetBuildingData();
    
    // Get full repair cost
    ResourceAmount[] fullCost = data.GetRepairCost();
    
    // Calculate missing health percentage
    float missingHealthPercent = 1f - (building.CurrentHealth / building.MaxHealth);
    
    // Calculate proportional cost
    ResourceAmount[] proportionalCost = new ResourceAmount[fullCost.Length];
    for (int i = 0; i < fullCost.Length; i++)
    {
        proportionalCost[i] = new ResourceAmount(
            fullCost[i].Id,
            Mathf.CeilToInt(fullCost[i].Amount * missingHealthPercent)
        );
    }
    
    return proportionalCost;
}
```

### Repair Cost Formula

```
Repair Cost = Full Repair Cost × (Missing Health / Max Health)
```

Example:

* Full Repair Cost: 100 Gold
* Building Health: 70/100 (70%)
* Missing Health: 30%
* Repair Cost: 30 Gold

## Public Methods

### Mode Interface Methods

```csharp
/// <summary>
/// Called when entering repair mode. Highlights damaged buildings.
/// </summary>
public void OnEnterMode()

/// <summary>
/// Called when exiting repair mode. Clears highlights.
/// </summary>
public void OnExitMode()

/// <summary>
/// Called each frame while in repair mode.
/// </summary>
public void OnUpdate()

/// <summary>
/// Primary action (click). Repairs the selected building.
/// </summary>
public void OnPrimaryAction()

/// <summary>
/// Secondary action. Cancels selection.
/// </summary>
public void OnSecondaryAction()

/// <summary>
/// Cancel action. Exits repair mode.
/// </summary>
public void OnCancelAction()
```

## Usage Examples

### Entering Repair Mode

```csharp
// Switch to repair mode
buildingManager.SetMode(BuildingMode.Repair);

// Damaged buildings are now highlighted
```

### Repairing a Building

```csharp
// User clicks on a damaged building
repairMode.OnPrimaryAction();

// This triggers:
// 1. Check if building is damaged
// 2. Calculate repair cost
// 3. Validate player can afford
// 4. Deduct resources
// 5. Restore building health
// 6. Show completion effects
```

### Get Repair Preview

```csharp
// Get preview of repair cost
ResourceAmount[] cost = repairMode.CalculateRepairCost(building);

// Display in UI
uiManager.ShowRepairCost(cost, building.CurrentHealth, building.MaxHealth);
```

## Validation

Before repairing, the system validates:

```csharp
public bool CanRepair(BuildingInstance building)
{
    // 1. Building must exist
    if (building == null) return false;
    
    // 2. Player must own the building
    if (!buildingManager.IsOwnedByLocalPlayer(building)) return false;
    
    // 3. Building must have health system
    if (!building.HasHealthSystem) return false;
    
    // 4. Building must be damaged
    if (building.CurrentHealth >= building.MaxHealth) return false;
    
    // 5. Player must afford repair
    if (!Economy.CanSpend(CalculateRepairCost(building))) return false;
    
    return true;
}
```

## Events

```csharp
/// <summary>
/// Fired when a building is repaired.
/// </summary>
public event Action<BuildingInstance, float> OnBuildingRepaired;

/// <summary>
/// Fired when repair attempt fails.
/// </summary>
public event Action<BuildingInstance, string> OnRepairFailed;

/// <summary>
/// Fired when repair cost changes (hover).
/// </summary>
public event Action<ResourceAmount[]> OnRepairCostPreviewChanged;
```

## Execution Flow

```
OnPrimaryAction()
    │
    ├── Check if building is selected
    ├── Validate repair conditions
    │   ├── Player ownership?
    │   ├── Building damaged?
    │   └── Can afford?
    │
    ├── Calculate repair cost
    │   ├── Get full repair cost
    │   └── Calculate proportional amount
    │
    ├── Deduct resources
    │   └── Show floating cost text
    │
    ├── Restore health
    │   ├── Set to full health
    │   └── Update health bar
    │
    ├── Spawn effects
    │   └── Show repair completion
    │
    └── Fire OnBuildingRepaired event
```

## Visual Feedback

### Highlight System

When hovering over buildings in repair mode:

| State                   | Visual          |
| ----------------------- | --------------- |
| Fully Repaired          | No highlight    |
| Damaged (can afford)    | Green highlight |
| Damaged (cannot afford) | Red highlight   |

### Health Bar

```csharp
// Update health bar for selected building
building.ShowHealthBar(true);
building.UpdateHealthBar(currentHealth, maxHealth);
```

## BuildingData Repair Settings

| Property         | Description                        |
| ---------------- | ---------------------------------- |
| `repairFullCost` | Resources required for full repair |

## Partial vs Full Repair

### Full Repair

```csharp
// Restore to maximum health
building.RestoreHealth(building.MaxHealth);
```

### Partial Repair

```csharp
// Restore by a fixed amount
building.AddHealth(50f);

// Or restore to a specific amount
building.SetHealth(targetHealth);
```
