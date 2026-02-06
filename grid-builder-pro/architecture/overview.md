    # Architecture Overview

## System Overview

GridBuilderPro follows a modular, layered architecture that separates concerns between grid infrastructure, building mechanics, and user interface. This design enables flexibility, maintainability, and performance optimization by ensuring each system operates independently while communicating through well-defined interfaces.

The architecture is built around three primary layers:

1. **Grid Layer** (`CorePro.GridSystem`): Provides the fundamental spatial infrastructure including cell management, occupancy tracking, and grid visualization.

2. **Building Layer** (`CorePro.GridBuilderPro`): Implements building-specific logic including placement, selection, upgrades, and lifecycle management.

3. **UI Layer** (`CorePro.GridBuilderPro.UI`): Handles user interaction panels, building selection, and contextual information display.

Each layer is designed to be independent and extensible, allowing developers to modify or replace components without affecting other parts of the system. The use of ScriptableObjects for data management enables clean separation between data and behavior, while the interface-based design facilitates dependency injection and testing.

## High-Level Architecture Diagram

```
                                    ┌─────────────────────────────────────┐
                                    │         Unity Engine Core            │
                                    │   (Scene Management, Physics, etc.) │
                                    └─────────────────────────────────────┘
                                                    │
                                                    ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              GridBuilderPro Core                                 │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                           GridSystem Layer                                │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │   │
│  │  │ GridManager  │  │ GridData*    │  │ GridVisual*  │  │ GridHighlight│ │   │
│  │  │ (Singleton)  │  │ (Interface) │  │ (Interface) │  │ (Interface) │ │   │
│  │  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘ │   │
│  │         │                │                   │                   │          │   │
│  │         ▼                ▼                   ▼                   ▼          │   │
│  │  ┌─────────────────────────────────────────────────────────────────────┐   │   │
│  │  │              GridObjectRegistry (Runtime Occupancy Tracking)        │   │   │
│  │  └─────────────────────────────────────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────────────────────────────┘   │
│                                        │                                          │
│                                        ▼                                          │
│  ┌─────────────────────────────────────────────────────────────────────────────┐   │
│  │                           Building Layer                                     │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │   │
│  │  │BuildingManager│ │Construction │  │  Preview     │  │  Upgrade     │      │   │
│  │  │ (Singleton)  │  │  System     │  │  System      │  │  System      │      │   │
│  │  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘      │   │
│  │         │                │                   │                   │              │   │
│  │         ▼                ▼                   ▼                   ▼              │   │
│  │  ┌─────────────────────────────────────────────────────────────────────┐         │   │
│  │  │            Building Modes (Build, Sell, Repair, Upgrade)            │         │   │
│  │  └─────────────────────────────────────────────────────────────────────┘         │   │
│  │                                                                                 │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │   │
│  │  │BuildingData  │  │BuildingData*│  │ BuildingTags│  │   Building   │         │   │
│  │  │   (SO)       │  │   base       │  │ (Enum)      │   Instance   │         │   │
│  │  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘         │   │
│  └─────────────────────────────────────────────────────────────────────────────┘   │
│                                        │                                          │
│                                        ▼                                          │
│  ┌─────────────────────────────────────────────────────────────────────────────┐   │
│  │                             UI Layer                                        │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │   │
│  │  │BuildingMgrUI │ │BlueprintPanel│  │  CostPanel  │  │ CategoryPanel│     │   │
│  │  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘      │   │
│  └─────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
                    ┌─────────────────────────────────────┐
                    │         Editor Tools Layer           │
                    │  (GridBuilderCreator, SetupWindow)   │
                    └─────────────────────────────────────┘

* Indicates interface-based component that can be customized
```

## Core Components

### GridManager

The [`GridManager`](grid-manager.md) class serves as the central authority for all grid-based operations. As a Singleton pattern implementation, it provides global access to grid functionality throughout the game. The GridManager handles several critical responsibilities:

**Grid Configuration and Initialization**

During initialization, the GridManager creates a [`GridDataChunked3D`](grid-data.md) instance that manages the underlying grid data structure. The chunk-based approach divides the grid into smaller, manageable regions, improving memory locality and query performance. Configuration parameters include cell dimensions, grid size, chunk partitioning, and terrain adaptation settings.

**Coordinate Transformation**

The manager provides bidirectional conversion between world space positions and grid coordinates through methods such as [`GetCell()`](grid-manager.md#methods) and [`CellToWorld()`](grid-manager.md#methods). These transformations account for the grid's origin point and cell dimensions, enabling precise spatial queries.

**Occupancy Management**

Through the [`GridObjectRegistry`](grid-object-registry.md), the GridManager tracks runtime object placement on the grid. This registry maintains a mapping between cell coordinates and occupying objects, supporting queries such as "what object occupies this cell?" and "what cells does this object occupy?"

**Placement Validation**

Before objects can be placed on the grid, the manager validates whether the target area is free using methods like [`IsAreaFree2D()`](grid-manager.md#occupancy-queries) and [`IsAreaFree3D()`](grid-manager.md#occupancy-queries). These methods check both statically baked data (terrain, obstacles) and dynamically registered objects.

**Event System**

The GridManager exposes events that allow other systems to react to grid changes:
- [`OnCellsChanged`](grid-manager.md#events): Triggered when grid occupancy changes
- [`OnSelectionChanged`](grid-manager.md#events): Triggered when user selection changes

### BuildingManager

The [`BuildingManager`](building-manager.md) class acts as the central coordinator for all building-related systems. Unlike the GridManager which handles pure spatial concerns, the BuildingManager implements gameplay logic for building placement, selection, and modification.

**Mode Management**

The BuildingManager maintains the current building mode (Build, Sell, Repair, Inactive) and delegates operations to specialized systems. When the mode changes, the manager coordinates UI updates, input system reconfiguration, and grid visualization settings.

**System Coordination**

The BuildingManager instantiates and coordinates several subsystems:
- [`BuildingConstructionSystem`](building-construction.md): Handles the construction process, including resource deduction and building instantiation
- [`BuildingPreviewSystem`](building-preview.md): Manages the preview visualization that appears during building placement
- [`BuildingUpgradeSystem`](building-upgrade.md): Implements building upgrade mechanics
- [`BuildingInputSystem`](input-system.md): Processes user input for building operations

**Building Lifecycle**

Through the [`RemoveBuilding()`](building-manager.md#methods) method, the BuildingManager handles the complete removal of buildings from the scene, including grid unregistration and visual cleanup.

## Data Layer

### ScriptableObject Architecture

GridBuilderPro uses ScriptableObjects extensively for data management, providing several advantages:

1. **Editor Integration**: ScriptableObjects automatically appear in the Unity inspector, enabling convenient editing without custom tools.

2. **Asset Pipeline**: Data can be saved as separate assets, enabling version control and modular asset management.

3. **Runtime Access**: Unlike MonoBehaviours, ScriptableObjects can be loaded and accessed without scene dependencies.

4. **Event Propagation**: ScriptableObjects can serve as lightweight event buses using Unity's [CreateAssetMenu] attribute.

### Key Data Assets

| Asset | Purpose | Location |
|-------|---------|----------|
| [`BuildingDatabaseSO`](building-database.md) | Catalog of all available buildings | `GridBuilderPro/Scripts/SO/` |
| [`BuildingData`](building-data.md) | Individual building definitions | `GridBuilderPro/Scripts/Data/` |
| [`BuildingTags`](building-tags.md) | Enumeration of building categories | `GridBuilderPro/Scripts/Data/` |
| [`GridBuilderDefaults`](grid-builder-defaults.md) | Default prefab references | `GridBuilderPro/Scripts/SO/` |
| [`BakedGridData`](baked-grid-data.md) | Pre-computed collision/terrain data | Generated at edit time |

### Runtime Data Containers

For runtime performance, the system uses struct-based containers that minimize garbage collection:

- [`CellData`](cell-data.md): Represents a single cell's state including occupancy, position, and validity
- [`AreaFreeCheckResult`](area-free-check.md): Detailed result of placement validation queries

## Systems Layer

### Building Modes

Building modes define the operations players can perform. The architecture supports extensible mode creation through the [`IBuildingMode`](../../api/CorePro.GridBuilderPro/IBuildingMode.md) interface:

**Built-in Modes**

- **Build Mode**: Validates placement, shows preview, and initiates construction when confirmed
- **Sell Mode**: Removes buildings and returns resources based on sell value calculations
- **Repair Mode**: Restores building health with associated costs

**Custom Mode Implementation**

```csharp
public class CustomBuildingMode : IBuildingMode
{
    public void Initialize(BuildingManager manager) { }
    public void Execute(Vector3Int cell, BuildingInstance building) { }
    public void Cancel() { }
    public bool CanExecute(BuildingInstance building) => true;
}
```

### Building Systems

The [`BuildingConstructionSystem`](building-construction.md), [`BuildingPreviewSystem`](building-preview.md), and [`BuildingUpgradeSystem`](building-upgrade.md) handle specific aspects of building management. Each system operates independently but receives coordinated input from the BuildingManager.

### Input Handling

The [`BuildingInputSystem`](input-system.md) translates user input (mouse clicks, keyboard shortcuts) into building operations. This abstraction allows for different input schemes without modifying core building logic.

## Visualization Layer

### Grid Visualization

The grid visualization system uses an interface-based approach, allowing different visual implementations:

- [`GridVisualizerBase`](grid-visualizer.md): Abstract base class for grid mesh generation
- [`GridHightlightProjector`](grid-highlight.md): Handles cell highlighting using decal projectors

### Building Effects

The [`BuildingSpawnMaterializer`](building-spawn-materializer.md) provides visual feedback during building construction, spawning effects at the building location.

## Editor Tools Layer

GridBuilderPro includes comprehensive editor tooling for project configuration:

| Tool | Purpose | Access |
|------|---------|--------|
| [`GridBuilderCreator`](editor/creator.md) | Initial project setup wizard | Unity Menu |
| [`GridBuilderSetupWindow`](editor/setup-window.md) | Advanced configuration | Unity Menu |
| [`BuildingDatabaseSOEditor`](editor/database-editor.md) | Building catalog management | Inspector |
| [`BuildingDataEditorWindow`](editor/data-editor-window.md) | Individual building editing | Inspector |

## Performance Considerations

### Zero GC Design

GridBuilderPro is engineered for zero garbage collection during gameplay through several techniques:

1. **Struct-Based Collections**: All hot-path data structures use structs rather than classes to avoid heap allocations.

2. **Pre-allocated Arrays**: Memory is allocated during initialization rather than at runtime.

3. **Object Pooling**: Building instances and visual effects are pooled rather than instantiated and destroyed.

4. **Interface-Based Iteration**: Collections expose `IReadOnlyList<T>` interfaces to enable efficient enumeration without allocation.

### Spatial Query Optimization

The chunk-based grid data structure enables efficient spatial queries:

- **O(1) Lookups**: Individual cell queries use direct array indexing
- **Chunk-Based Iteration**: Area queries iterate only over affected chunks
- **Bitset Storage**: Blocked cell data uses compressed bitsets for memory efficiency

### Baked Data

Static grid data (terrain, obstacles) is pre-computed at edit time and serialized to assets, eliminating runtime raycasting and collision checks.

## Extensibility

### Interface-Based Extension Points

| Interface | Purpose | Extension Example |
|-----------|---------|-------------------|
| [`IGridVisualizer`](../../api/CorePro.GridSystem/IGridVisualizer.md) | Custom grid rendering | Custom shader-based visualization |
| [`IGridHighlight`](../../api/CorePro.GridSystem/IGridHighlight.md) | Custom highlighting | Particle-based effects |
| [`IBuildingMode`](../../api/CorePro.GridBuilderPro/IBuildingMode.md) | New building operations | Demolish, Combine modes |
| [`IBuildingActionTarget`](../../api/CorePro.GridBuilderPro/IBuildingActionTarget.md) | Building responses | Custom destruction effects |

### Event Hooks

Systems expose events that allow external code to react to changes without direct coupling:

```csharp
// Example: React to building placement
GridManager.Instance.OnCellsChanged += region => {
    // Custom logic
};
```

## Dependency Flow

```
                    ┌─────────────────────────────────────────┐
                    │           BuildingManager               │
                    │  (Depends on: GridManager, Systems)     │
                    └─────────────────────────────────────────┘
                                      │
                                      ▼
                    ┌─────────────────────────────────────────┐
                    │           Building Systems              │
                    │  (Construction, Preview, Upgrade)        │
                    └─────────────────────────────────────────┘
                                      │
                                      ▼
                    ┌─────────────────────────────────────────┐
                    │             GridManager                │
                    │  (No dependencies on Building Layer)    │
                    └─────────────────────────────────────────┘
                                      │
                                      ▼
                    ┌─────────────────────────────────────────┐
                    │         GridDataChunked3D              │
                    │         GridObjectRegistry              │
                    └─────────────────────────────────────────┘
```

The dependency flow is unidirectional from higher-level gameplay systems to lower-level infrastructure. The GridManager has no knowledge of building-specific concepts, enabling its reuse in non-building scenarios.

## Related Documentation

| Topic | Description |
|-------|-------------|
| [GridManager](grid-manager.md) | Complete GridManager API reference |
| [BuildingManager](building-manager.md) | Complete BuildingManager API reference |
| [Grid Data](grid-data.md) | Grid data structures and algorithms |
| [Building Data](building-data.md) | Building definition assets |
| [Editor Tools](editor/overview.md) | Editor integration documentation |
| [API Reference](../api/) | Complete API documentation |
