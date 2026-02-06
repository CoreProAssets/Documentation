# Building manager

## Overview

The `BuildingManager` class serves as the central coordinator for all building-related systems in GridBuilderPro. It acts as a facade that delegates operations to specialized subsystems including construction, preview, upgrade, and building modes. This singleton manages the overall state of building operations and provides a unified API for external systems.

## Key Responsibilities

| Responsibility          | Description                                  |
| ----------------------- | -------------------------------------------- |
| **System Coordination** | Delegates to specialized building systems    |
| **Mode Management**     | Manages building modes (Build, Sell, Repair) |
| **State Management**    | Tracks current building state                |
| **Event Propagation**   | Fires events for external systems            |
| **Building Lifecycle**  | Handles building removal and cleanup         |

## Singleton Access

```csharp
// Recommended access method
BuildingManager manager = BuildingManager.Instance;

// Or via property
BuildingManager manager = this.buildingManager;
```

## Configuration Properties

### Game Rules

| Property             | Type          | Description                      |
| -------------------- | ------------- | -------------------------------- |
| `localPlayerFaction` | `FactionType` | Faction type of the local player |

### State

| Property      | Type           | Description                    |
| ------------- | -------------- | ------------------------------ |
| `currentMode` | `BuildingMode` | Currently active building mode |

### References

| Property              | Type                         | Description                  |
| --------------------- | ---------------------------- | ---------------------------- |
| `buildingInputSystem` | `BuildingInputSystem`        | Input processing system      |
| `previewSystem`       | `BuildingPreviewSystem`      | Preview visualization system |
| `constructionSystem`  | `BuildingConstructionSystem` | Construction timer system    |
| `upgradeSystem`       | `BuildingUpgradeSystem`      | Building upgrade system      |

### Modes

| Property     | Type                 | Description         |
| ------------ | -------------------- | ------------------- |
| `modeBuild`  | `BuildingModeBuild`  | Build mode handler  |
| `modeSell`   | `BuildingModeSell`   | Sell mode handler   |
| `modeRepair` | `BuildingModeRepair` | Repair mode handler |

## Building Modes

The BuildingManager supports the following modes:

```csharp
public enum BuildingMode
{
    None,       // No mode active
    Inactive,   // Mode inactive but ready
    Build,      // Building placement mode
    Sell,       // Building sell mode
    Upgrade,    // Building upgrade mode
    Repair      // Building repair mode
}
```

## Public API - System Access

### Get System References

```csharp
/// <summary>
/// Gets reference to BuildingModeBuild.
/// </summary>
public BuildingModeBuild GetBuildMode() => modeBuild;

/// <summary>
/// Gets reference to BuildingModeSell.
/// </summary>
public BuildingModeSell GetSellMode() => modeSell;

/// <summary>
/// Gets reference to BuildingPreviewSystem.
/// </summary>
public BuildingPreviewSystem GetPreviewSystem() => previewSystem;

/// <summary>
/// Gets reference to BuildingConstructionSystem.
/// </summary>
public BuildingConstructionSystem GetConstructionSystem() => constructionSystem;

/// <summary>
/// Gets reference to BuildingUpgradeSystem.
/// </summary>
public BuildingUpgradeSystem GetUpgradeSystem() => upgradeSystem;

/// <summary>
/// Gets reference to GridManager.
/// </summary>
public GridManager GetGridManager() => gridManager;

/// <summary>
/// Gets reference to BuildingManagerUI.
/// </summary>
public BuildingManagerUI GetBuildingUIManager() => buildingManagerUI;
```

## Mode Management

### Set Mode

```csharp
/// <summary>
/// Sets the current building mode.
/// </summary>
/// <param name="newMode">The mode to switch to.</param>
public void SetMode(BuildingMode newMode)
```

**Mode Behavior:**

| Mode       | Grid Visibility | UI Visibility | Input System |
| ---------- | --------------- | ------------- | ------------ |
| `None`     | Hidden          | Hidden        | Disabled     |
| `Inactive` | Hidden          | Hidden        | Disabled     |
| `Build`    | Visible         | Building Menu | Build Mode   |
| `Sell`     | Visible         | Sell UI       | Sell Mode    |
| `Upgrade`  | Visible         | Upgrade UI    | Upgrade Mode |
| `Repair`   | Visible         | Repair UI     | Repair Mode  |

### Toggle Mode

```csharp
/// <summary>
/// Toggles between a specific mode and Inactive.
/// </summary>
/// <param name="mode">The mode to toggle.</param>
public void ToggleMode(BuildingMode mode)
```

### Quick Mode Setters

```csharp
/// <summary>
/// Sets build mode.
/// </summary>
[Button("Set Build Mode")]
public void SetBuildMode()

/// <summary>
/// Sets sell mode.
/// </summary>
[Button("Set Sell Mode")]
public void SetSellMode()

/// <summary>
/// Disables all modes.
/// </summary>
[Button("Disable All Modes")]
public void DisableAllModes()
```

## Building Lifecycle

### Remove Building

```csharp
/// <summary>
/// Removes a building from the grid and scene.
/// </summary>
/// <param name="building">The building to remove.</param>
/// <returns>True if removal was successful.</returns>
public bool RemoveBuilding(BuildingInstance building)
```

**Removal Flow:**

```
RemoveBuilding(building)
    │
    ├── 1. Check if building is selected
    │   └── Deselect if needed
    │
    ├── 2. Get base cell from registry
    │   └── Vector3Int baseCell = gridManager.GetCellData(building).Cell
    │
    ├── 3. Calculate unblock area
    │   └── Consider BlockCellsAbove interface
    │
    ├── 4. Unblock cells
    │   └── gridManager.UnblockArea3D()
    │
    ├── 5. Remove from registry
    │   └── gridManager.RemoveObject()
    │
    ├── 6. Update connections
    │   └── BuildingConnectionSystem.UpdateConnectionsAround()
    │
    └── 7. Deactivate
        └── building.gameObject.SetActive(false)
```

## Mode Change Handling

```csharp
/// <summary>
/// Disables building mode and clears selection.
/// </summary>
public void HandleDisableBuildingMode()

/// <summary>
/// Cancels current selection but stays in current mode.
/// </summary>
public void HandleCancelSelection()
```

## Player Ownership

```csharp
/// <summary>
/// Checks if a building is owned by the local player.
/// </summary>
/// <param name="building">The building to check.</param>
/// <returns>True if owned by local player.</returns>
public bool IsOwnedByLocalPlayer(BuildingInstance building)
{
    if (building == null) return false;
    return building.Faction == localPlayerFaction;
}

/// <summary>
/// Sets the local player faction (for AI/multiplayer).
/// </summary>
/// <param name="faction">The faction to set.</param>
public void SetLocalPlayerFaction(FactionType faction)
{
    localPlayerFaction = faction;
}
```

## Events

```csharp
/// <summary>
/// Fired when the building mode changes.
/// </summary>
public event Action<BuildingMode> OnModeChanged;
```

### Subscribing to Events

```csharp
// Subscribe to mode changes
BuildingManager.Instance.OnModeChanged += OnModeChanged;

void OnModeChanged(BuildingMode newMode)
{
    Debug.Log($"Switched to mode: {newMode}");
}
```

## Blackboard Integration

The BuildingManager works with `BuildingSystemBlackboardSO` for shared state:

| Blackboard Property | Usage                                |
| ------------------- | ------------------------------------ |
| `selectedBlueprint` | Currently selected building to place |
| `selectedBuilding`  | Currently selected existing building |
| `TargetCellRef`     | Current cell under cursor            |

## Usage Examples

### Basic Building Flow

```csharp
// Switch to build mode
BuildingManager.Instance.SetMode(BuildingMode.Build);

// Select a building from UI
blackboard.SetSelectedBlueprintData(selectedBuildingData);

// User clicks to place
// Building is placed, construction starts (if applicable)

// Switch to select mode
BuildingManager.Instance.SetMode(BuildingMode.Inactive);

// Click on building to select
// Show building info panel

// Switch to sell mode
BuildingManager.Instance.SetMode(BuildingMode.Sell);

// Click on selected building to sell
// Resources refunded, building removed
```

### Checking State

```csharp
// Check current mode
BuildingMode currentMode = BuildingManager.Instance.CurrentMode;

// Check if in build mode
if (currentMode == BuildingMode.Build)
{
    // Handle build mode specific logic
}

// Get building system
var previewSystem = BuildingManager.Instance.GetPreviewSystem();
var constructionSystem = BuildingManager.Instance.GetConstructionSystem();
```

## Editor Support

### Inspector Buttons

The BuildingManager provides editor buttons for quick testing:

| Button                | Action                 |
| --------------------- | ---------------------- |
| **Set Build Mode**    | Switches to build mode |
| **Set Sell Mode**     | Switches to sell mode  |
| **Disable All Modes** | Clears all modes       |

## Performance Considerations

* **Singleton Access**: Use cached references when possible
* **Event Subscription**: Unsubscribe in OnDestroy to avoid leaks
* **Mode Switching**: Mode changes are lightweight operations

