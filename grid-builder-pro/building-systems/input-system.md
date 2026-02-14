# Input system

## Overview

The `BuildingInputSystem` class handles user input processing for building operations in GridBuilderPro. It provides an abstraction layer between raw input sources (keyboard, mouse, touch) and the building mode system. This allows for flexible input handling that can be easily extended or replaced without modifying core building logic.



## Usage Examples

### Basic Input Processing

```csharp
void Update()
{
    // Process input each frame
    inputSystem.ProcessInput();
    
    // Check for actions
    if (inputSystem.WasPrimaryActionPressed())
    {
        // Execute primary action
        currentMode.OnPrimaryAction();
    }
    
    if (inputSystem.WasSecondaryActionPressed())
    {
        // Execute secondary action
        currentMode.OnSecondaryAction();
    }
    
    if (inputSystem.WasCancelPressed())
    {
        // Execute cancel action
        currentMode.OnCancelAction();
    }
}
```

## Input State

```csharp
/// <summary>
/// Current state of input axes.
/// </summary>
public struct InputState
{
    public bool PrimaryAction;      // Primary action pressed
    public bool SecondaryAction;    // Secondary action pressed
    public bool CancelPressed;       // Cancel pressed
    public bool RotatePressed;       // Rotate pressed
    public Vector2 MousePosition;   // Current mouse position
    public bool IsPointerOverUI;     // Is pointer over UI
}
```

### Accessing Input State

```csharp
InputState state = inputSystem.GetCurrentState();

// Check if pointer is over UI
if (state.IsPointerOverUI)
{
    // Don't process building input
    return;
}
```

## Raycasting

```csharp
/// <summary>
/// Performs a raycast from the camera through the mouse position.
/// </summary>
/// <returns>RaycastHit information or null if no hit.</returns>
public RaycastHit? GetMouseWorldHit()
```

### Example

```csharp
// Get world position from mouse
RaycastHit? hit = inputSystem.GetMouseWorldHit();
if (hit.HasValue)
{
    Vector3 worldPos = hit.Value.point;
    Vector3Int cell = gridManager.GetCell(worldPos);
}
```

## Touch Support

For mobile platforms:

```csharp
/// <summary>
/// Processes touch input for mobile devices.
/// </summary>
public void ProcessTouchInput()

/// <summary>
/// Gets the touch position converted to world coordinates.
/// </summary>
/// <param name="touchIndex">Index of the touch.</param>
/// <returns>World position of the touch.</returns>
public Vector3 GetTouchWorldPosition(int touchIndex)
```

## Input System Configuration

### New Input System

```csharp
// Enable new input system
inputSystem.useNewInputSystem = true;

// Configure InputAction references
public InputAction rotateAction;
public InputAction placeAction;
public InputAction cancelAction;
```

### Input Action Callbacks

```csharp
// Setup InputAction callbacks
rotateAction.performed += ctx => OnRotate();
placeAction.performed += ctx => OnPlace();
cancelAction.performed += ctx => OnCancel();
```

## Integration with Building Modes

```csharp
// Each mode receives processed input
public void OnUpdate()
{
    if (inputSystem.WasPrimaryActionPressed())
    {
        OnPrimaryAction();
    }
    
    if (inputSystem.WasRotatePressed())
    {
        RotatePreview(Direction.Right);
    }
}
```

## Debug Input Display

```csharp
void OnGUI()
{
    // Display current input state
    GUILayout.Label($"Primary: {inputSystem.WasPrimaryActionPressed()}");
    GUILayout.Label($"Secondary: {inputSystem.WasSecondaryActionPressed()}");
    GUILayout.Label($"Cancel: {inputSystem.WasCancelPressed()}");
    GUILayout.Label($"Rotate: {inputSystem.WasRotatePressed()}");
}
```

## Performance Considerations

* **Input Batching**: Multiple inputs processed in single Update call
* **No Allocations**: Raycast results use cached structures
* **Conditional Processing**: Only processes input when in building mode

| Topic                     | Description                          |
| ------------------------- | ------------------------------------ |
| Building Systems Overview | High-level building systems overview |
| BuildingModeBuild         | Build mode with input handling       |
| GridManager               | Grid coordinate system               |
| Unity Input System        | Input System documentation           |

| Topic                     | Description                          |
| ------------------------- | ------------------------------------ |
| Building Systems Overview | High-level building systems overview |
| BuildingModeBuild         | Build mode with input handling       |
| GridManager               | Grid coordinate system               |
| Unity Input System        | Input System documentation           |
