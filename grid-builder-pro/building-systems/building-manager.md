# Building manager

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

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

### Set Mode

```csharp
/// <summary>
/// Sets the current building mode.
/// </summary>
/// <param name="newMode">The mode to switch to.</param>
public void SetMode(BuildingMode newMode)
```

### Toggle Mode

```csharp
/// <summary>
/// Toggles between a specific mode and Inactive.
/// </summary>
/// <param name="mode">The mode to toggle.</param>
public void ToggleMode(BuildingMode mode)
```

### Remove Building

```csharp
/// <summary>
/// Removes a building from the grid and scene.
/// </summary>
/// <param name="building">The building to remove.</param>
/// <returns>True if removal was successful.</returns>
public bool RemoveBuilding(BuildingInstance building)
```

### Mode Change Handling

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

### Player Ownership

```csharp
/// <summary>
/// Checks if a building is owned by the local player.
/// </summary>
/// <param name="building">The building to check.</param>
/// <returns>True if owned by local player.</returns>
public bool IsOwnedByLocalPlayer(BuildingInstance building)
{
    if (building == null) 
        return false;
        
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

### Events

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

### Blackboard Integration

The BuildingManager works with `BuildingSystemBlackboardSO` for shared state:

| Blackboard Property | Usage                                |
| ------------------- | ------------------------------------ |
| `selectedBlueprint` | Currently selected building to place |
| `selectedBuilding`  | Currently selected existing building |
| `TargetCellRef`     | Current cell under cursor            |

## Performance Considerations

* **Singleton Access**: Use cached references when possible
* **Event Subscription**: Unsubscribe in OnDestroy to avoid leaks
* **Mode Switching**: Mode changes are lightweight operations

