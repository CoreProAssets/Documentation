# BuildingModeSell

## Overview

The `BuildingModeSell` class implements the `IBuildingMode` interface for selling buildings in GridBuilderPro. This mode allows players to remove buildings from the grid and recover a portion of their build cost as resources. The mode provides visual feedback highlighting buildings that can be sold and handles the complete sell transaction including resource refunds.

## Class Definition

```csharp
namespace CorePro.GridBuilderPro
{
    public class BuildingModeSell : MonoBehaviourCorePro, IBuildingMode
    {
        // Properties and methods
    }
}
```

## Key Responsibilities

| Responsibility | Description |
|---------------|-------------|
| **Building Selection** | Allows selecting buildings to sell |
| **Refund Calculation** | Calculates sell rewards based on BuildingData |
| **Resource Refund** | Returns resources to player economy |
| **Visual Feedback** | Highlights sellable buildings |
| **Building Removal** | Removes buildings from grid and scene |

## Configuration Properties

| Property | Type | Description |
|----------|------|-------------|
| `sellHighlightColor` | Color | Color for sell mode highlight |
| `confirmRequired` | bool | Require confirmation before selling |
| `showRefundPreview` | bool | Show refund amount on hover |

## Sell Calculation

The sell system calculates refunds based on `BuildingData`:

```csharp
public ResourceAmount[] GetSellRewards(BuildingInstance building)
{
    BuildingData data = building.GetBuildingData();
    
    // Check for custom sell rewards
    if (data.HasCustomSellRewards())
        return data.GetSellRewards();
    
    // Calculate from build cost with percentage
    return data.GetSellRewards();
}
```

### Default Sell Formula

```
Refund Amount = Build Cost × Sell Refund Percentage
```

Example:
- Build Cost: 100 Gold, 50 Wood
- Sell Refund Percentage: 0.5 (50%)
- Refund: 50 Gold, 25 Wood

## Public Methods

### Mode Interface Methods

```csharp
/// <summary>
/// Called when entering sell mode. Enables highlight and selection.
/// </summary>
public void OnEnterMode()

/// <summary>
/// Called when exiting sell mode. Clears selection and disables highlight.
/// </summary>
public void OnExitMode()

/// <summary>
/// Called each frame while in sell mode.
/// </summary>
public void OnUpdate()

/// <summary>
/// Primary action (click). Sells the selected building.
/// </summary>
public void OnPrimaryAction()

/// <summary>
/// Secondary action. Cancels selection.
/// </summary>
public void OnSecondaryAction()

/// <summary>
/// Cancel action. Exits sell mode.
/// </summary>
public void OnCancelAction()
```

## Usage Examples

### Entering Sell Mode

```csharp
// Switch to sell mode
buildingManager.SetMode(BuildingMode.Sell);

// Buildings are now highlighted indicating they can be sold
```

### Selling a Building

```csharp
// User clicks on a building
sellMode.OnPrimaryAction();

// This triggers:
// 1. Validate building is selected
// 2. Calculate sell rewards
// 3. Add resources to player economy
// 4. Show floating text with refund
// 5. Remove building from grid
// 6. Return building to pool
```

### Refund Preview

```csharp
// Get preview of sell rewards
ResourceAmount[] preview = sellMode.GetSellRewards(buildingInstance);

// Display in UI
uiManager.ShowRefundPreview(preview);
```

## Validation

Before selling, the system validates:

```csharp
public bool CanSell(BuildingInstance building)
{
    // 1. Building must exist
    if (building == null) return false;
    
    // 2. Player must own the building
    if (!buildingManager.IsOwnedByLocalPlayer(building)) return false;
    
    // 3. Building cannot be under construction
    if (building.IsUnderConstruction) return false;
    
    // 4. Building cannot be locked
    if (building.IsLocked) return false;
    
    return true;
}
```

## Events

```csharp
/// <summary>
/// Fired when a building is sold.
/// </summary>
public event Action<BuildingInstance, ResourceAmount[]> OnBuildingSold;

/// <summary>
/// Fired when sell attempt fails.
/// </summary>
public event Action<BuildingInstance, string> OnSellFailed;
```

## Execution Flow

```
OnPrimaryAction()
    │
    ├── Check if building is selected
    ├── Validate sell conditions
    │   ├── Player ownership?
    │   └── Not under construction?
    │
    ├── Calculate sell rewards
    │   ├── Custom rewards or percentage?
    │   └── Format refund amount
    │
    ├── Add resources to player
    │   └── Show floating refund text
    │
    ├── Remove from grid
    │   └── Unblock occupied cells
    │
    ├── Return to pool
    └── Fire OnBuildingSold event
```

## Visual Feedback

### Highlight System

When hovering over buildings in sell mode:

| State | Visual |
|-------|--------|
| Can Sell | Green highlight |
| Cannot Sell (wrong owner) | Red highlight |
| Under Construction | Yellow highlight |

### Floating Text

```csharp
// Spawn floating text showing refund
FloatingTextManager.Instance.SpawnResourceAmounts(
    rewards: refundAmounts,
    position: building.GetCenterPoint(),
    style: floatingTextStyle,
    isPositive: true  // Green text for refunds
);
```

## Configuration

### BuildingData Sell Settings

| Property | Description |
|----------|-------------|
| `sellRefundPercentage` | Portion of build cost returned (0-1) |
| `customSellRewards` | Optional custom sell rewards |

## Related Documentation

| Topic | Description |
|-------|-------------|
| [Building Systems Overview](overview.md) | High-level building systems overview |
| [BuildingData](../data/building-data.md) | Building definition with sell settings |
| [Economy System](../economy/overview.md) | Resource management |
| [IBuildingMode Interface](../api/CorePro.GridBuilderPro.IBuildingMode.md) | Mode interface definition |
