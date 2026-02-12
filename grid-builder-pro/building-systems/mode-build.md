# Mode build

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

## Overview

The `BuildingModeBuild` class implements the `IBuildingMode` interface for standard building placement in GridBuilderPro. It coordinates between the preview system, grid manager, and economy to provide a complete building placement experience. This class handles building selection, area validation, placement execution, resource deduction, and input processing.

## Key Responsibilities

| Responsibility           | Description                                   |
| ------------------------ | --------------------------------------------- |
| **Area Validation**      | Checks grid availability and terrain flatness |
| **Placement Execution**  | Instantiates buildings from pools             |
| **Resource Deduction**   | Validates and spends build costs              |
| **Extra Input Handling** | Processes mouse and keyboard input            |
| **Cell Selection**       | Allows selecting existing buildings           |

## Configuration Properties

### Input Actions

<table><thead><tr><th width="289.3333740234375">Property</th><th width="155.3333740234375">Type</th><th width="184">Default Binding</th><th>Description</th></tr></thead><tbody><tr><td><code>rotateClockwiseAction</code></td><td><code>InputAction</code></td><td>R key</td><td>Rotates preview 90° clockwise</td></tr><tr><td><code>rotateCounterClockwiseAction</code></td><td><code>InputAction</code></td><td>(optional)</td><td>Rotates preview 90° counter-clockwise</td></tr></tbody></table>

### Effects

<table><thead><tr><th width="218.33331298828125">Property</th><th width="313">Type</th><th>Description</th></tr></thead><tbody><tr><td><code>effectsOnStartBuild</code></td><td><code>ConstructionEffect[]</code></td><td>Effects spawned when construction starts</td></tr><tr><td><code>floatingTextStyle</code></td><td><code>FloatingTextStyle</code></td><td>Style for build cost floating text</td></tr></tbody></table>

## Public Methods

### Mode Interface Methods

```csharp
/// <summary>
/// Called when entering build mode. Enables input actions.
/// </summary>
public void OnEnterMode()

/// <summary>
/// Called when exiting build mode. Disables input and clears selection.
/// </summary>
public void OnExitMode()

/// <summary>
/// Called each frame while in build mode. Updates preview and highlight.
/// </summary>
public void OnUpdate()

/// <summary>
/// Primary action (left click). Places building or selects cell.
/// </summary>
public void OnPrimaryAction()

/// <summary>
/// Secondary action (right click). Deselects current cell.
/// </summary>
public void OnSecondaryAction()

/// <summary>
/// Cancel action (escape). Cancels mode and clears selection.
/// </summary>
public void OnCancelAction()
```



### Placement Flow

```csharp
// User clicks to place building
buildingModeBuild.OnPrimaryAction();

// This triggers:
// 1. Validate placement area
// 2. Check resources
// 3. Deduct resources and show floating text
// 4. Spawn building from pool
// 5. Register on grid
// 6. Start construction if applicable
```

### Rotation

```csharp
// Rotate clockwise
buildingModeBuild.RotatePreview(Direction.Right);

// Rotate counter-clockwise (if configured)
buildingModeBuild.RotatePreview(Direction.Left);
```

## Placement Validation

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





## Cell Selection

The build mode also supports selecting existing buildings:

```csharp
public void TrySelectCell()
{
    ref readonly var targetCell = ref blackboard.TargetCellRef;
    
    // Check if valid for selection
    if (targetCell.isOnGrid == false || blackboard.isPointerOverUI)
        return;
    
    if (blackboard.selectedBlueprint)
        return;  // Cannot select when trying to place
    
    // Select cell
    blackboard.SetSelectedCell(targetCell);
    blackboard.TrySetSelectedBuilding();
    
    if (blackboard.HasSelectedBuilding())
        gridManager.SelectSingle(targetCell);
}

public void DeselectCurrCell()
{
    blackboard.SetSelectedCell(CellData.Invalid);
    blackboard.SetSelectedBuilding(null);
    gridManager.HideSelection();
}
```

## Integration with Blackboard

The mode uses `BuildingSystemBlackboardSO` for state management:

| Blackboard Property       | Usage                                |
| ------------------------- | ------------------------------------ |
| `selectedBlueprint`       | Currently selected building to place |
| `TargetCellRef`           | Current cell under cursor            |
| `currentPreviewFootprint` | Footprint including rotation         |
| `previewRotation`         | Current preview rotation angle       |
| `isPointerOverUI`         | Whether cursor is over UI            |
