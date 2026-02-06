# Building preview system

## Overview

The `BuildingPreviewSystem` class manages the visual "ghost" or "blueprint" that follows the cursor before a building is placed. It handles material swapping between valid and invalid states, 3D footprint validation, rotation logic, and floating animations for visual polish.

## Key Responsibilities

| Responsibility           | Description                                                           |
| ------------------------ | --------------------------------------------------------------------- |
| **Ghost Visualization**  | Spawns and manages preview instances using object pooling             |
| **Material Management**  | Swaps between valid (green) and invalid (red) materials               |
| **Position Tracking**    | Updates preview position based on mouse cursor and grid cell          |
| **Rotation Handling**    | Manages 90-degree rotation steps for rectangular buildings            |
| **Animation**            | Provides floating animation effect for visual polish                  |
| **Footprint Validation** | Ensures preview reflects actual building footprint including rotation |

## Configuration Properties

### Preview Materials

| Property                 | Type       | Description                              |
| ------------------------ | ---------- | ---------------------------------------- |
| `previewValidMaterial`   | `Material` | Material shown when placement is valid   |
| `previewInvalidMaterial` | `Material` | Material shown when placement is blocked |

### Position Settings

| Property         | Type    | Description                                           |
| ---------------- | ------- | ----------------------------------------------------- |
| `previewYOffset` | `float` | Vertical offset to prevent Z-fighting (default: 0.1f) |

### Animation Settings

| Property         | Type    | Description                               |
| ---------------- | ------- | ----------------------------------------- |
| `enableFloating` | `bool`  | Enables floating animation effect         |
| `floatAmplitude` | `float` | Height of floating motion (default: 0.2f) |
| `floatSpeed`     | `float` | Speed of floating animation (default: 2f) |

## Debug Properties (Read-Only)

| Property                 | Type               | Description                  |
| ------------------------ | ------------------ | ---------------------------- |
| `isValid`                | `bool`             | Current validity state       |
| `lastIsValidState`       | `bool`             | Previous validity state      |
| `currentPreviewInstance` | `BuildingInstance` | Current preview instance     |
| `currBuildingPrefab`     | `BuildingInstance` | Current preview prefab       |
| `usingCustomPreview`     | `bool`             | Whether using custom preview |
| `currentRotationStep`    | `int`              | Current rotation step (0-3)  |
| `previewIsActive`        | `bool`             | Whether preview is active    |

## Public Methods

### Initialization

```csharp
/// <summary>
/// Initializes the system with required managers and resets the animation timer.
/// </summary>
/// <param name="newBuildingManager">Reference to the main BuildingManager.</param>
public void Init(BuildingManager newBuildingManager)
```

### Preview Management

```csharp
/// <summary>
/// Updates the preview's position and visual state (valid/invalid).
/// Automatically handles spawning a new preview if the selected blueprint changes.
/// </summary>
public void UpdatePreview()

/// <summary>
/// Checks if there is currently a visual preview being displayed in the scene.
/// </summary>
/// <returns>True if the preview is active and visible.</returns>
public bool IsPreviewActive()

/// <summary>
/// Sets the validity state for material swapping.
/// </summary>
/// <param name="valid">True if placement is valid.</param>
public void SetValidateState(bool valid)

/// <summary>
/// Soft hide: Just disables the GameObject but keeps the reference.
/// Use this when mouse leaves the grid but might come back.
/// </summary>
public void HidePreview()
```

### Rotation

```csharp
/// <summary>
/// Cycles the building preview rotation by 90 degrees.
/// Also handles "Footprint Swapping" for rectangular buildings.
/// </summary>
/// <param name="direction">The direction to rotate (Left/Right).</param>
public void RotatePreview(Direction direction)
```

## Usage Examples

### Basic Usage

```csharp
// Initialize at startup
previewSystem.Init(buildingManager);

// Update each frame in build mode
void Update()
{
    if (buildingManager.CurrentMode == BuildingMode.Build)
    {
        previewSystem.UpdatePreview();
    }
}
```

### Setting Validity

```csharp
// Check placement validity
bool canPlace = gridManager.IsAreaFree3D(cell, footprint);
previewSystem.SetValidateState(canPlace);

// Materials are automatically swapped:
// Valid (true) → previewValidMaterial (green)
// Invalid (false) → previewInvalidMaterial (red)
```

### Rotation

```csharp
// Rotate 90 degrees clockwise
previewSystem.RotatePreview(Direction.Right);

// Rotate 90 degrees counter-clockwise
previewSystem.RotatePreview(Direction.Left);

// Rotation steps cycle 0 → 1 → 2 → 3 → 0
// currentRotationStep = ((currentRotationStep + rotationChange + 4) % 4)
```

## Rotation Footprint Swapping

The preview system automatically swaps footprint dimensions for rectangular buildings:

```
Rotation Step 0 (0°):  Width=2, Depth=1
Rotation Step 1 (90°): Width=1, Depth=2
Rotation Step 2 (180°): Width=2, Depth=1
Rotation Step 3 (270°): Width=1, Depth=2
```

```csharp
var baseFootprintSize3D = selectedBlueprint.prefab.GetFootprint3D();

if (currentRotationStep % 2 == 1) // 90° or 270°
{
    blackboard.SetCurrentPreviewFootprint3D(
        new Vector3Int(baseFootprintSize3D.z, baseFootprintSize3D.y, baseFootprintSize3D.x)
    );
}
else // 0° or 180°
{
    blackboard.SetCurrentPreviewFootprint3D(baseFootprintSize3D);
}
```

## Material Management

The system maintains cached renderer lists for efficient material updates:

```csharp
// When validity changes, materials are updated
previewSystem.UpdatePreviewMaterials(force: false);

// Or force update
previewSystem.UpdatePreviewMaterials(force: true);
```

### Custom Preview Support

Buildings can define custom preview renderers:

```csharp
// Check if using custom preview
if (usingCustomPreview)
{
    // Get custom preview renderers
    Renderer[] customRenderers = currentPreviewInstance.GetCustomPreviewRenderers();
}
```

## Animation

### Floating Animation

```csharp
private void AnimateFloating()
{
    if (currentPreviewInstance == null) return;

    float elapsed = Time.time - startTime;

    // Sine-wave based offset
    float normalizedWave = (Mathf.Sin(elapsed * floatSpeed) + 1f) * 0.5f;
    float offsetY = normalizedWave * floatAmplitude;

    // Apply offset to base position
    currentPreviewInstance.transform.position = basePosition + Vector3.up * offsetY;
}
```

### Disable Animation

```csharp
// Disable floating for static preview
enableFloating = false;

// Preview will stay at fixed position
```

## Preview Spawning

### Create Preview

```csharp
private void CreatePreview()
{
    currBuildingPrefab = selectedBlueprint.prefab;
    
    // Create pool key for preview
    previewPoolKey = PoolManager.Instance.CreateCustomPoolKey(
        currBuildingPrefab, 
        "BuildingPreview"
    );

    // Get from pool
    currentPreviewInstance = PoolManager.Instance.GetDisabledPooledComponent(
        currBuildingPrefab, 
        previewPoolKey
    );

    // Disable non-visual components
    DisableNonVisualComponents(currentPreviewInstance);

    // Activate
    currentPreviewInstance.gameObject.SetActive(true);
    previewIsActive = true;
}
```

### Disable Non-Visual Components

```csharp
private void DisableNonVisualComponents(BuildingInstance preview)
{
    // Disable colliders
    Collider[] colliders = preview.GetComponentsInChildren<Collider>();
    foreach (var col in colliders)
        col.enabled = false;

    // Disable logic scripts
    MonoBehaviour[] scripts = preview.GetComponentsInChildren<MonoBehaviour>();
    foreach (var script in scripts)
        if (script != this)
            script.enabled = false;

    // Disable rigidbodies
    Rigidbody[] bodies = preview.GetComponentsInChildren<Rigidbody>();
    foreach (var rb in bodies)
        rb.isKinematic = true;
}
```

## Cleanup

### Despawn Preview

```csharp
private void DespawnPreview()
{
    if (currentPreviewInstance != null)
    {
        currentPreviewInstance.gameObject.SetActive(false);
        PoolManager.Instance.Return(previewPoolKey, currentPreviewInstance);
        currentPreviewInstance = null;
    }

    usingCustomPreview = false;
    previewIsActive = false;

    PreviewRenderers.Clear();
    originalMaterials.Clear();
}
```

### OnDestroy

```csharp
private void OnDestroy()
{
    DespawnPreview();
}
```

## Position Update Optimization

The preview only updates when the mouse moves to a new cell:

```csharp
private void UpdatePreviewPosition()
{
    ref readonly var targetCell = ref blackboard.TargetCellRef;

    // Only update if position changed
    if (targetCell.WorldPos == lastWorldPos)
        return;

    lastWorldPos = targetCell.WorldPos;
    basePosition = lastWorldPos + Vector3.up * previewYOffset;

    if (enableFloating == false)
    {
        currentPreviewInstance.transform.position = basePosition;
    }
}
```
