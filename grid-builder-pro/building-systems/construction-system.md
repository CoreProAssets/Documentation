# Construction system

<figure><img src="../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

## Overview

The `BuildingConstructionSystem` class manages the construction phase of buildings in GridBuilderPro. It handles time-based construction, integrates with the global timer system, and spawns visual feedback for construction progress. This system is responsible for creating timers, managing callbacks, and coordinating between the economy system and building instances.

## Key Responsibilities

| Responsibility           | Description                                                             |
| ------------------------ | ----------------------------------------------------------------------- |
| **Timer Management**     | Creates and manages construction timers using `GlobalTimerHub`          |
| **Visual Feedback**      | Spawns `BuildingInstanceTimer` components above buildings               |
| **Callback Handling**    | Triggers completion callbacks when construction finishes                |
| **Resource Integration** | Works with the economy system to validate and deduct construction costs |

## Configuration Properties

| Property                | Type                    | Description                                                 |
| ----------------------- | ----------------------- | ----------------------------------------------------------- |
| `buildingInstanceTimer` | `BuildingInstanceTimer` | Prefab for world-space timer UI                             |
| `timerOffset`           | `Vector3`               | Vertical offset for timer display (default: `(0f, 1f, 0f)`) |

## Public Methods

### Start Construction

```csharp
/// <summary>
/// Initiates the construction process for a specific building.
/// Spawns a world-space timer UI and registers a logical timer in the GlobalTimerHub.
/// </summary>
/// <param name="building">The building being constructed.</param>
/// <param name="duration">Total time in seconds for construction.</param>
public void StartConstruction(BuildingInstance building, float duration)
```

### Usage Example

```csharp
// Start construction for a building with 5-second build time
constructionSystem.StartConstruction(buildingInstance, 5f);

// Construction will:
// 1. Create a timer in GlobalTimerHub
// 2. Spawn world-space timer UI above building (if timer prefab assigned)
// 3. Call building.OnConstructionStarted() immediately
// 4. Call building.OnConstructionCompleted() when finished
```

## Construction Flow

```
StartConstruction(building, duration)
    │
    ├── 1. Validate parameters
    │   ├── Check building != null
    │   └── Check duration > 0
    │
    ├── 2. Create logical timer in GlobalTimerHub
    │   ├── duration: construction duration
    │   ├── useUnscaledTime: false (uses game time)
    │   ├── channel: Gameplay
    │   ├── complete: building (callback target)
    │   ├── updateListener: building
    │   └── token: building.GetInstanceID()
    │
    ├── 3. Spawn world-space timer UI (if configured)
    │   ├── Get pooled BuildingInstanceTimer
    │   ├── Position at building center + offset
    │   ├── Setup timer duration
    │   └── Create timer with timerAdapter as listener
    │
    ├── 4. Set construction state
    │   ├── building.SetConstructionDuration(duration)
    │   └── building.OnConstructionStarted()
    │
    └── 5. Wait for completion
        └── GlobalTimerHub calls building.OnConstructionCompleted()
```

## Integration Points

The system integrates with:

| System               | Purpose                           |
| -------------------- | --------------------------------- |
| **GlobalTimerHub**   | For time tracking and callbacks   |
| **PoolManager**      | For timer UI pooling              |
| **BuildingInstance** | For construction state management |

### GlobalTimerHub Integration

```csharp
// Create timer in GlobalTimerHub
GlobalTimerHub.Instance.Create(
    duration: duration,
    useUnscaledTime: false,
    channel: GlobalTimerHub.TimerChannel.Gameplay,
    complete: building,           // Callback when complete
    updateListener: building,      // Update callback
    token: building.GetInstanceID(),
    owner: building
);
```

### BuildingInstance Callbacks

```csharp
// Called immediately when construction starts
building.OnConstructionStarted();

// Called when timer completes
building.OnConstructionCompleted();
```

## Building Construction States

```csharp
// BuildingInstance states
public enum ConstructionState
{
    NotStarted,      // Initial state
    InProgress,      // Construction in progress
    Completed,       // Construction finished
    Cancelled        // Construction was cancelled
}
```

## BuildingInstance Timer Configuration

The `BuildingInstanceTimer` prefab provides world-space UI for construction progress:

```csharp
// Spawn timer above building
var timerAdapter = PoolManager.Instance.GetPooledComponent(buildingInstanceTimer);
timerAdapter.transform.position = building.GetCenterPoint() + timerOffset;
timerAdapter.Setup(duration);
```

### Timer Adapter Setup

```csharp
public class BuildingInstanceTimer : MonoBehaviour
{
    public void Setup(float duration)
    {
        // Initialize timer with duration
        // Show progress bar or countdown
    }
}
```

## Configuration in BuildingData

Construction settings are defined in `BuildingData`:

| Property              | Description                                  |
| --------------------- | -------------------------------------------- |
| `constructionTime`    | Time in seconds to construct the building    |
| `effectsOnStartBuild` | Visual effects spawned at construction start |

## Events

The construction system triggers callbacks through the timer system:

```csharp
// Timer completion callback
void OnTimerComplete(object owner)
{
    BuildingInstance building = owner as BuildingInstance;
    building.OnConstructionCompleted();
}

// Timer update callback
void OnTimerUpdate(float progress, object owner)
{
    // Update progress UI
}
```

## Usage Examples

### Basic Construction

```csharp
// Check if building is under construction
bool isUnderConstruction = building.IsUnderConstruction;

// Get remaining construction time
float remainingTime = building.GetRemainingConstructionTime();

// Get construction progress (0-1)
float progress = building.GetConstructionProgress();
```

### Cancel Construction

```csharp
// Cancel construction (returns resources)
building.CancelConstruction();

// Or via GlobalTimerHub
GlobalTimerHub.Instance.CancelTimer(building.GetInstanceID());
```

### Skip Construction (Debug/Cheat)

```csharp
// Complete construction immediately
building.CompleteConstructionImmediately();
```

## Performance Considerations

* **Zero Allocation**: Timers are managed through the GlobalTimerHub without allocations
* **Pooling**: Timer UI instances are pooled for reuse
* **Batched Updates**: Multiple construction timers can be updated efficiently
