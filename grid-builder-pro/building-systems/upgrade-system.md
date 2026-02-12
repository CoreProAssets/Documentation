# Upgrade system

<figure><img src="../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

## Overview

The `BuildingUpgradeSystem` class handles the evolution of buildings into higher levels in GridBuilderPro. It manages resource validation, building replacement via object pooling, grid re-registration, and construction triggers for upgraded buildings. This system enables a complete upgrade pipeline from validation through execution.

## Key Responsibilities

| Responsibility           | Description                                                     |
| ------------------------ | --------------------------------------------------------------- |
| **Upgrade Validation**   | Checks prerequisites (resources, ownership, construction state) |
| **Building Replacement** | Swaps old building for new level using object pooling           |
| **Grid Registration**    | Re-registers the new building on the grid                       |
| **Resource Management**  | Deducts upgrade costs from economy                              |
| **Visual Effects**       | Spawns upgrade completion effects                               |
| **Event Propagation**    | Triggers upgrade events for external systems                    |

## Configuration Properties

| Property               | Type           | Description                           |
| ---------------------- | -------------- | ------------------------------------- |
| `effectsOnStarUpgrade` | `GameObject[]` | Effects spawned at upgrade location   |
| `enableDebugLogs`      | `bool`         | Enable debug logging (default: false) |

## Public Methods

### Validation

```csharp
/// <summary>
/// Performs a multi-step check to see if an upgrade is currently possible.
/// Checks for: Null references, active construction, max level reached, and resource availability.
/// </summary>
/// <param name="buildingInstance">The building to check.</param>
/// <returns>True if all conditions for an upgrade are met.</returns>
public bool CanBeUpgraded(BuildingInstance buildingInstance)
```

### Execution

```csharp
/// <summary>
/// Executes the actual upgrade process: consumes resources, removes the old building,
/// spawns the new prefab, and registers it on the grid.
/// </summary>
/// <param name="oldInstance">The current building instance to be replaced.</param>
public void ExecuteUpgrade(BuildingInstance oldInstance)
```

## Usage Examples

### Check Upgrade Availability

```csharp
// Check if building can be upgraded
bool canUpgrade = upgradeSystem.CanBeUpgraded(buildingInstance);

if (canUpgrade)
{
    // Show upgrade button in UI
    ui.ShowUpgradeButton();
}
else
{
    // Show why upgrade is not available
    ui.ShowUpgradeLocked();
}
```

### Execute Upgrade

```csharp
// User clicks upgrade button
upgradeSystem.ExecuteUpgrade(buildingInstance);

// This triggers:
// 1. Validate prerequisites
// 2. Get next level data
// 3. Deduct upgrade costs
// 4. Remove old building
// 5. Spawn new level
// 6. Register on grid
// 7. Start construction (if applicable)
// 8. Spawn effects and fire events
```

## Upgrade Validation

```csharp
public bool CanBeUpgraded(BuildingInstance building)
{
    // 1. Building must exist
    if (building == null)
    {
        Debug.LogError("BuildingUpgradeSystem: BuildingInstance is null!");
        return false;
    }
    
    // 2. Player must own the building
    if (!buildingManager.IsOwnedByLocalPlayer(building))
        return false;
    
    // 3. Cannot upgrade building under construction
    if (building.IsUnderConstruction)
        return false;
    
    // 4. Must have BuildingData
    if (building.buildingData == null)
        return false;
    
    // 5. Must have next level defined
    if (HasMaxLevel(building))
        return false;
    
    // 6. Player must afford upgrade costs
    if (!Economy.CanSpend(building.buildingData.buildCost))
        return false;
    
    return true;
}

bool HasMaxLevel(BuildingInstance building)
{
    // Check if next level data is null
    return building.buildingData.nextLevelData == null;
}
```

## Upgrade Execution Flow

```
ExecuteUpgrade(oldInstance)
    │
    ├── 1. Validate prerequisites
    │   └── Exit if validation fails
    │
    ├── 2. Get next level data
    │   ├── BuildingInstance nextLevelPrefab
    │   ├── position = oldInstance.transform.position
    │   └── rotation = oldInstance.GetVisualRotAngle()
    │
    ├── 3. Get base cell before removal
    │   └── Vector3Int baseCell = gridManager.GetCellData(oldInstance).Cell
    │
    ├── 4. Deduct upgrade costs
    │   ├── Get upgrade cost from BuildingData
    │   └── Economy.TrySpend(resourceAmounts)
    │
    ├── 5. Remove old building
    │   ├── Remove from grid registry
    │   └── Return to pool or deactivate
    │
    ├── 6. Spawn new level
    │   ├── Get pooled BuildingInstance
    │   ├── Set position and rotation
    │   └── Initialize new instance
    │
    ├── 7. Register new building
    │   └── gridManager.PlaceObject(newInstance, baseCell)
    │
    ├── 8. Start construction (if applicable)
    │   ├── Get construction time from new building
    │   └── StartConstruction(newInstance, duration)
    │
    ├── 9. Spawn effects
    │   └── PoolManager.Instance.SpawnGameObject()
    │
    └── 10. Fire events
        └── OnBuildingUpgraded?.Invoke(newInstance)
```

## BuildingData Upgrade Configuration

| Property        | Description                            |
| --------------- | -------------------------------------- |
| `level`         | Current upgrade level                  |
| `nextLevelData` | Reference to next level's BuildingData |

## Events

```csharp
/// <summary>
/// Event fired when a building has successfully finished the upgrade transformation.
/// </summary>
public event Action<BuildingInstance> OnBuildingUpgraded;
```

### Subscribing to Events

```csharp
// Subscribe to upgrade completion
upgradeSystem.OnBuildingUpgraded += OnBuildingUpgraded;

void OnBuildingUpgraded(BuildingInstance newBuilding)
{
    Debug.Log($"Upgraded to: {newBuilding.name}");
    // Update UI, play sound, etc.
}
```

## Effects System

```csharp
private void SpawnEffects(BuildingInstance buildingInstance)
{
    foreach (var prefab in effectsOnStarUpgrade)
    {
        if (prefab == null) continue;
        
        PoolManager.Instance.SpawnGameObject(
            prefab, 
            buildingInstance.GetCenterPoint(), 
            Quaternion.identity
        );
    }
}
```

## Resource Handling

### Upgrade Cost Calculation

```csharp
public ResourceAmount[] GetUpgradeCost(BuildingInstance building)
{
    BuildingData nextLevelData = building.buildingData.GetNextLevelBuildingData();
    return nextLevelData.buildCost;
}
```

### Cost Display Format

```csharp
private string FormatRewards(ResourceAmount[] rewards)
{
    if (rewards == null || rewards.Length == 0)
        return "nothing";

    var sb = new System.Text.StringBuilder();
    for (int i = 0; i < rewards.Length; i++)
    {
        if (i > 0) sb.Append(", ");
        sb.Append(rewards[i].Amount).Append(" ").Append(rewards[i].Id);
    }
    return sb.ToString();
}
```

## Integration with Other Systems

### BuildingManager

```csharp
// Get upgrade system reference
BuildingUpgradeSystem upgradeSystem = buildingManager.GetUpgradeSystem();
```

### GridManager

```csharp
// Register new building on grid
gridManager.PlaceObject(newInstance, baseCell);

// Unregister old building
gridManager.RemoveObject(oldInstance);
```

### PoolManager

```csharp
// Spawn new level from pool
BuildingInstance newInstance = PoolManager.Instance.GetPooledComponent(nextLevelPrefab);
```

## Debug Logging

```csharp
if (enableDebugLogs)
{
    Debug.Log($"Upgraded {oldInstance.name}, spent: {FormatRewards(resourceAmounts)}");
    Debug.Log($"Upgraded {oldInstance.name} to {newInstance.name}");
}
```

## Performance Considerations

* **Object Pooling**: New building instances are spawned from pool, not instantiated
* **Zero Allocation**: Resource handling avoids allocations during execution
* **Efficient Grid Operations**: Uses pre-calculated base cell for re-registration
