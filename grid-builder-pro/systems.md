# Systems

## Introduction

The building systems in GridBuilderPro handle all aspects of the building lifecycle, from initial placement through construction, upgrades, and eventual removal. Each system is designed with a specific responsibility, allowing for clean separation of concerns and easy extensibility.

The building systems layer sits between the high-level BuildingManager and the low-level GridManager, implementing gameplay-specific logic that coordinates grid operations with economy, UI, and visual feedback systems.

## System Architecture

```
BuildingManager
     │
     ├── BuildingConstructionSystem
     │    │    Handles: Construction timers, completion callbacks
     │    └── Uses: GlobalTimerHub, PoolManager
     │
     ├── BuildingPreviewSystem
     │    │    Handles: Ghost visualization, material swapping
     │    └── Uses: PoolManager, GridManager
     │
     ├── BuildingUpgradeSystem
     │    │    Handles: Building evolution, level transitions
     │    └── Uses: Economy, PoolManager
     │
     └── BuildingModeBuild
          │    Handles: Placement, rotation, validation
          └── Uses: PreviewSystem, GridManager, Economy
```

## BuildingConstructionSystem

The `BuildingConstructionSystem` manages the construction phase of buildings. It handles time-based construction, integrates with the global timer system, and spawns visual feedback for construction progress.

### Core Responsibilities

* **Timer Management**: Creates and manages construction timers using `GlobalTimerHub`
* **Visual Feedback**: Spawns `BuildingInstanceTimer` components above buildings
* **Callback Handling**: Triggers completion callbacks when construction finishes
* **Resource Integration**: Works with the economy system to validate and deduct construction costs

### Key Methods

| Method                                  | Description                                   |
| --------------------------------------- | --------------------------------------------- |
| `StartConstruction(building, duration)` | Initiates construction process for a building |

### Usage Example

```csharp
// Start construction for a building with 5-second build time
buildingConstructionSystem.StartConstruction(buildingInstance, 5f);

// Construction will:
// 1. Create a timer in GlobalTimerHub
// 2. Spawn world-space timer UI above building
// 3. Call building.OnConstructionStarted() immediately
// 4. Call building.OnConstructionCompleted() when finished
```

### Integration Points

The system integrates with:

* **GlobalTimerHub**: For time tracking and callbacks
* **PoolManager**: For timer UI pooling
* **BuildingInstance**: For construction state management
* **Economy**: For resource validation before construction starts

## BuildingPreviewSystem

The [`BuildingPreviewSystem`](/broken/pages/e2ccee8ab05deff366dc53674b395fdadc14040a) provides the visual "ghost" or "blueprint" that follows the cursor during building placement. It handles all aspects of preview visualization, including material swapping, animation, and rotation.

### Core Responsibilities

* **Ghost Visualization**: Spawns and manages preview instances using object pooling
* **Material Management**: Swaps between valid (green) and invalid (red) materials
* **Position Tracking**: Updates preview position based on mouse cursor and grid cell
* **Rotation Handling**: Manages 90-degree rotation steps for rectangular buildings
* **Animation**: Provides floating animation effect for visual polish
* **Footprint Validation**: Ensures preview reflects actual building footprint including rotation

### Configuration Properties

| Property                 | Type     | Description                              |
| ------------------------ | -------- | ---------------------------------------- |
| `previewValidMaterial`   | Material | Material shown when placement is valid   |
| `previewInvalidMaterial` | Material | Material shown when placement is blocked |
| `previewYOffset`         | float    | Vertical offset to prevent Z-fighting    |
| `enableFloating`         | bool     | Enables floating animation effect        |
| `floatAmplitude`         | float    | Height of floating motion                |
| `floatSpeed`             | float    | Speed of floating animation              |

### Key Methods

| Method                                                                                             | Description                        |
| -------------------------------------------------------------------------------------------------- | ---------------------------------- |
| [`UpdatePreview()`](/broken/pages/e2ccee8ab05deff366dc53674b395fdadc14040a#updatepreview)          | Updates preview position and state |
| [`RotatePreview(direction)`](/broken/pages/e2ccee8ab05deff366dc53674b395fdadc14040a#rotatepreview) | Rotates preview by 90 degrees      |
| [`HidePreview()`](/broken/pages/e2ccee8ab05deff366dc53674b395fdadc14040a#hidepreview)              | Soft-hides the preview             |
| [`IsPreviewActive()`](/broken/pages/e2ccee8ab05deff366dc53674b395fdadc14040a#ispreviewactive)      | Checks if preview is visible       |

### Rotation System

The preview system supports 90-degree rotation steps with automatic footprint swapping for rectangular buildings:

```csharp
// Rotate 90 degrees clockwise
previewSystem.RotatePreview(Direction.Right);

// Rotate 90 degrees counter-clockwise
previewSystem.RotatePreview(Direction.Left);

// Footprint automatically swaps for rectangular buildings:
// 0°  : Width=2, Depth=1
// 90° : Width=1, Depth=2
```

### Material Management

The system maintains renderer lists for efficient material updates:

```csharp
// When validity changes, materials are updated
previewSystem.SetValidateState(isValid);

// Valid placement → previewValidMaterial (green)
// Invalid placement → previewInvalidMaterial (red)
```

## BuildingUpgradeSystem

The [`BuildingUpgradeSystem`](/broken/pages/318bcde5e47488722d5b90966f3ffa48049513c3) handles the evolution of buildings into higher levels. It manages resource validation, building replacement, and construction triggers for upgraded buildings.

### Core Responsibilities

* **Upgrade Validation**: Checks prerequisites (resources, ownership, construction state)
* **Building Replacement**: Swaps old building for new level using object pooling
* **Grid Registration**: Re-registers the new building on the grid
* **Resource Management**: Deducts upgrade costs from economy
* **Visual Effects**: Spawns upgrade completion effects
* **Event Propagation**: Triggers upgrade events for external systems

### Key Methods

| Method                                                                                                 | Description                         |
| ------------------------------------------------------------------------------------------------------ | ----------------------------------- |
| [`CanBeUpgraded(building)`](/broken/pages/318bcde5e47488722d5b90966f3ffa48049513c3#canbeupgraded)      | Checks if upgrade is possible       |
| [`ExecuteUpgrade(oldInstance)`](/broken/pages/318bcde5e47488722d5b90966f3ffa48049513c3#executeupgrade) | Performs the upgrade transformation |

### Upgrade Validation

Before upgrading, the system checks:

```csharp
bool CanBeUpgraded(BuildingInstance building)
{
    // 1. Building must exist
    if (building == null) return false;
    
    // 2. Player must own the building
    if (!buildingManager.IsOwnedByLocalPlayer(building)) return false;
    
    // 3. Building cannot be under construction
    if (building.IsUnderConstruction) return false;
    
    // 4. Next level must exist
    if (building.buildingData.HasUpgrade() == false) return false;
    
    // 5. Player must afford upgrade costs
    if (!Economy.CanSpend(building.buildingData.GetUpgradeCost())) return false;
    
    return true;
}
```

### Upgrade Execution Flow

```
ExecuteUpgrade()
    │
    ├── 1. Validate prerequisites
    ├── 2. Get next level data from BuildingData
    ├── 3. Deduct upgrade resources
    ├── 4. Remove old building from grid
    ├── 5. Spawn new level from pool
    ├── 6. Register new building on grid
    ├── 7. Start construction (if applicable)
    └── 8. Spawn effects and fire events
```

## Building Mode: Build

The [`BuildingModeBuild`](/broken/pages/14e14fc1eec5148c90f2aa2b32a850b41592f3a3) class implements the IBuildingMode interface for standard building placement. It coordinates between the preview system, grid manager, and economy to provide a complete building placement experience.

### Core Responsibilities

* **Blueprint Selection**: Handles building selection from UI
* **Area Validation**: Checks grid availability and terrain flatness
* **Placement Execution**: Instantiates buildings from pools
* **Resource Deduction**: Validates and spends build costs
* **Input Handling**: Processes mouse and keyboard input
* **Cell Selection**: Allows selecting existing buildings

### Key Methods

| Method                                                                                             | Description                     |
| -------------------------------------------------------------------------------------------------- | ------------------------------- |
| [`OnPrimaryAction()`](/broken/pages/14e14fc1eec5148c90f2aa2b32a850b41592f3a3#onprimaryaction)      | Places building or selects cell |
| [`OnSecondaryAction()`](/broken/pages/14e14fc1eec5148c90f2aa2b32a850b41592f3a3#onsecondaryaction)  | Deselects current cell          |
| [`OnCancelAction()`](/broken/pages/14e14fc1eec5148c90f2aa2b32a850b41592f3a3#oncancelaction)        | Cancels building mode           |
| [`RotatePreview(direction)`](/broken/pages/14e14fc1eec5148c90f2aa2b32a850b41592f3a3#rotatepreview) | Rotates preview                 |

### Input Actions

| Action                         | Default Binding | Description                           |
| ------------------------------ | --------------- | ------------------------------------- |
| `rotateClockwiseAction`        | R key           | Rotates preview 90° clockwise         |
| `rotateCounterClockwiseAction` | (optional)      | Rotates preview 90° counter-clockwise |
| `OnPrimaryAction`              | Left click      | Places building or selects cell       |
| `OnSecondaryAction`            | Right click     | Deselects current selection           |
| `OnCancelAction`               | Escape          | Cancels mode                          |

### Placement Validation

Before placing a building, the system validates:

```csharp
private bool IsAreaFree(in CellData cellData)
{
    // 1. Cell must be free (no occupant)
    if (cellData.IsFree == false) return false;
    
    // 2. Check grid occupancy for building footprint
    Vector3Int footprint = selectedBlueprint.prefab.GetFootprint3D();
    if (!gridManager.IsAreaFree3D(cellData.Cell, footprint)) return false;
    
    // 3. Terrain must be flat (for multi-cell buildings)
    if (!gridManager.IsAreaFlat3D(cellData.Cell, footprint)) return false;
    
    return true;
}
```

### Execution Sequence

```
OnPrimaryAction()
    │
    ├── Check if can build
    │   ├── Blueprint selected?
    │   └── Can afford?
    │
    ├── Validate placement area
    │   ├── Cell is free?
    │   ├── Footprint is free?
    │   └── Terrain is flat?
    │
    ├── Deduct resources
    │   └── Show floating cost text
    │
    ├── Spawn building from pool
    │   └── Set position and rotation
    │
    ├── Register on grid
    │   └── Block occupied cells
    │
    ├── Start construction (if applicable)
    └── Spawn effects
```

## Blackboard System

Building systems communicate through a shared [`BuildingSystemBlackboardSO`](/broken/pages/e6b37b0f3ae0ddbf39a0df96641e2b2e2e9f024c) that maintains the current state of building operations. This decouples systems and allows them to react to state changes.

### Blackboard State

| Property                  | Type                                                                         | Description                          |
| ------------------------- | ---------------------------------------------------------------------------- | ------------------------------------ |
| `selectedBlueprint`       | [`BuildingData`](/broken/pages/639ecaec942a14b472ba18c5b3a9ae79ff5a5099)     | Currently selected building to place |
| `selectedBuilding`        | [`BuildingInstance`](/broken/pages/b39c35e7c74bf0a277003a98312566ef48f0a1e9) | Currently selected existing building |
| `TargetCellRef`           | `ref readonly CellData`                                                      | Current cell under cursor            |
| `currentPreviewFootprint` | Vector3Int                                                                   | Footprint including rotation         |
| `previewRotation`         | float                                                                        | Current preview rotation angle       |
| `isPointerOverUI`         | bool                                                                         | Whether cursor is over UI            |

### Event System

The blackboard fires events when state changes:

```csharp
// When selected blueprint changes
blackboard.OnSelectedBlueprintChanged += OnBlueprintChanged;

// When selected building changes
blackboard.OnSelectedBuildingChanged += OnBuildingChanged;
```

## Object Pooling Integration

All building systems use the [`PoolManager`](/broken/pages/7596d9a2f8ec449683998799b459b892efeb047d) for efficient object management:

```csharp
// Get a pooled building instance
BuildingInstance instance = PoolManager.Instance.GetDisabledPooledComponent(prefab);

// Return to pool when removing
PoolManager.Instance.Return(poolKey, buildingInstance);

// Create preview-specific pool key
previewPoolKey = PoolManager.Instance.CreateCustomPoolKey(prefab, "BuildingPreview");
```

### Benefits

* **Zero Allocation**: No GameObject.Instantiate/Destroy during gameplay
* **Performance**: Faster spawning from pre-warmed pools
* **Memory**: Bounded memory usage regardless of building frequency

## Performance Considerations

### Hot Path Optimizations

1. **Cached Lists**: Preview renderers and materials are cached to avoid allocations
2. **Struct-Based Data**: CellData uses structs for stack allocation
3. **Reference Types**: Uses `ref readonly` for cell data to avoid copying
4. **Conditional Updates**: Only updates when position or state actually changes

### Memory Management

```csharp
// ❌ Avoid - causes allocation
foreach (var item in collection) { }

// ✅ Recommended - uses cached lists
for (int i = 0; i < PreviewRenderers.Count; i++)
{
    var renderer = PreviewRenderers[i];
}
```

## Related Documentation

| Topic                                                                                | Description                          |
| ------------------------------------------------------------------------------------ | ------------------------------------ |
| [BuildingConstructionSystem](/broken/pages/56d63f87a401284e244ba6d9238f56d3b762de9e) | Detailed construction system API     |
| [BuildingPreviewSystem](/broken/pages/e2ccee8ab05deff366dc53674b395fdadc14040a)      | Detailed preview system API          |
| [BuildingUpgradeSystem](/broken/pages/318bcde5e47488722d5b90966f3ffa48049513c3)      | Detailed upgrade system API          |
| [BuildingModeBuild](/broken/pages/14e14fc1eec5148c90f2aa2b32a850b41592f3a3)          | Detailed build mode API              |
| [BuildingManager](/broken/pages/8b97063d12b987064d2ff24ca6e12888dbaea5d5)            | High-level building coordination     |
| [Object Pooling](/broken/pages/7596d9a2f8ec449683998799b459b892efeb047d)             | PoolManager integration              |
| [Economy Integration](/broken/pages/e17da26cd36b9b6f66319cb6724f00cb10ab833c)        | Resource management                  |
| [UI Components](/broken/pages/51a6e3d35c5f08bf5cfbc9c1dc4625da931f04b5)              | Building selection and management UI |
