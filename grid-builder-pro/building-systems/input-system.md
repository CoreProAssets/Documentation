# Building input system

## Overview

The `BuildingInputSystem` class handles user input processing for building operations in GridBuilderPro. It provides an abstraction layer between raw input sources (keyboard, mouse, touch) and the building mode system. This allows for flexible input handling that can be easily extended or replaced without modifying core building logic.

## Key Responsibilities

| Responsibility         | Description                             |
| ---------------------- | --------------------------------------- |
| **Input Abstraction**  | Provides unified input interface        |
| **Mode Switching**     | Handles mode change input               |
| **Action Mapping**     | Maps input to building actions          |
| **Input State**        | Tracks current input state              |
| **Editor Integration** | Supports both old and new input systems |

## Input Sources

The system supports multiple input sources:

| Source                 | Description                                  |
| ---------------------- | -------------------------------------------- |
| **Unity Input System** | New InputSystem package (recommended)        |
| **Legacy Input**       | Input.GetButton, Input.GetKeyDown (fallback) |
| **Custom Input**       | Custom input providers (extensible)          |

## Configuration Properties

| Property             | Type      | Description                            |
| -------------------- | --------- | -------------------------------------- |
| `useNewInputSystem`  | `bool`    | Use Unity's new Input System           |
| `primaryActionKey`   | `KeyCode` | Primary action key (default: Mouse0)   |
| `secondaryActionKey` | `KeyCode` | Secondary action key (default: Mouse1) |
| `cancelKey`          | `KeyCode` | Cancel action key (default: Escape)    |
| `rotateKey`          | `KeyCode` | Rotate building key (default: R)       |

## Input Bindings

### Default Bindings

| Action                       | Default Binding   | Description                  |
| ---------------------------- | ----------------- | ---------------------------- |
| **Primary Action**           | Left Mouse Click  | Place building / Select cell |
| **Secondary Action**         | Right Mouse Click | Deselect                     |
| **Cancel**                   | Escape            | Cancel mode / Deselect       |
| **Rotate Clockwise**         | R                 | Rotate 90° clockwise         |
| **Rotate Counter-Clockwise** | Q                 | Rotate 90° counter-clockwise |
| **Mode: Build**              | 1                 | Switch to build mode         |
| **Mode: Sell**               | 2                 | Switch to sell mode          |
| **Mode: Repair**             | 3                 | Switch to repair mode        |

## Public Methods

### Mode Management

```csharp
/// <summary>
/// Changes the current building mode and reconfigures input accordingly.
/// </summary>
/// <param name="newMode">The mode to switch to.</param>
public void ChangeMode(BuildingMode newMode)
```

### Input Processing

```csharp
/// <summary>
/// Processes input for the current frame. Called from Update.
/// </summary>
public void ProcessInput()

/// <summary>
/// Checks if primary action was triggered this frame.
/// </summary>
/// <returns>True if primary action was triggered.</returns>
public bool WasPrimaryActionPressed()

/// <summary>
/// Checks if secondary action was triggered this frame.
/// </summary>
/// <returns>True if secondary action was triggered.</returns>
public bool WasSecondaryActionPressed()

/// <summary>
/// Checks if cancel action was triggered this frame.
/// </summary>
/// <returns>True if cancel action was triggered.</returns>
public bool WasCancelPressed()

/// <summary>
/// Checks if rotate action was triggered this frame.
/// </summary>
/// <returns>True if rotate action was triggered.</returns>
public bool WasRotatePressed()
```

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

### Mode Switching

```csharp
// Switch to build mode
inputSystem.ChangeMode(BuildingMode.Build);

// This reconfigures input bindings for build mode
// - Enables rotation input
// - Enables placement validation
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

### Touch Gestures

| Gesture        | Action                    |
| -------------- | ------------------------- |
| **Single Tap** | Place building / Select   |
| **Double Tap** | Rotate                    |
| **Long Press** | Show context menu         |
| **Drag**       | Rotate (after long press) |

## Custom Input Providers

The system supports custom input providers:

```csharp
public interface IInputProvider
{
    bool GetPrimaryAction();
    bool GetSecondaryAction();
    bool GetCancel();
    bool GetRotate();
    Vector3 GetPointerPosition();
    bool IsPointerOverUI();
}

/// <summary>
/// Register a custom input provider.
/// </summary>
/// <param name="provider">The input provider to use.</param>
public void SetInputProvider(IInputProvider provider)
```

### Example Custom Provider

```csharp
public class GamepadInputProvider : IInputProvider
{
    public bool GetPrimaryAction()
    {
        return Input.GetButtonDown("Fire1");
    }
    
    public bool GetSecondaryAction()
    {
        return Input.GetButtonDown("Fire2");
    }
    
    // ... implement other methods
}

// Register custom provider
inputSystem.SetInputProvider(new GamepadInputProvider());
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

| Topic                                                                                | Description                          |
| ------------------------------------------------------------------------------------ | ------------------------------------ |
| [Building Systems Overview](/broken/pages/f016c8ff2609f90a652203ead003158f64b9924f)  | High-level building systems overview |
| [BuildingModeBuild](/broken/pages/0307eb33f2b7315a407c03a03719f7b36e291bfe)          | Build mode with input handling       |
| [GridManager](/broken/pages/64bc8ef8453a682225b79129bec572a06e37d611)                | Grid coordinate system               |
| [Unity Input System](https://docs.unity3d.com/Packages/com.unity.inputsystem@latest) | Input System documentation           |

| Topic                                                                                | Description                          |
| ------------------------------------------------------------------------------------ | ------------------------------------ |
| [Building Systems Overview](/broken/pages/f016c8ff2609f90a652203ead003158f64b9924f)  | High-level building systems overview |
| [BuildingModeBuild](/broken/pages/0307eb33f2b7315a407c03a03719f7b36e291bfe)          | Build mode with input handling       |
| [GridManager](/broken/pages/64bc8ef8453a682225b79129bec572a06e37d611)                | Grid coordinate system               |
| [Unity Input System](https://docs.unity3d.com/Packages/com.unity.inputsystem@latest) | Input System documentation           |
