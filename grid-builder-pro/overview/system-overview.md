# System overview

GridBuilderPro follows a modular, layered architecture that separates concerns between grid infrastructure, building mechanics, and user interface. This design enables flexibility, maintainability, and performance optimization by ensuring each system operates independently while communicating through well-defined interfaces.

The architecture is built around three primary layers:

1. **Grid Layer** (`CorePro.GridSystem`): Provides the fundamental spatial infrastructure including cell management, occupancy tracking, and grid visualization.
2. **Building Layer** (`CorePro.GridBuilderPro`): Implements building-specific logic including placement, selection, upgrades, and lifecycle management.
3. **UI Layer** (`CorePro.GridBuilderPro.UI`): Handles user interaction panels, building selection, and contextual information display.

Each layer is designed to be independent and extensible, allowing developers to modify or replace components without affecting other parts of the system. The use of ScriptableObjects for data management enables clean separation between data and behavior, while the interface-based design facilitates dependency injection and testing.

## High-Level Architecture Diagram

{% code fullWidth="true" %}
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
{% endcode %}

## Core Components



### GridManager

The [`GridManager`](/broken/pages/38bc5fdadc2b02da6537009f8feb38cf90d7e51b) class serves as the central authority for all grid-based operations. As a Singleton pattern implementation, it provides global access to grid functionality throughout the game. The GridManager handles several critical responsibilities:

**Grid Configuration and Initialization**

During initialization, the GridManager creates a [`GridDataChunked3D`](/broken/pages/8bb9e06e8a73b8e5866dfec66f123e366faabefb) instance that manages the underlying grid data structure. The chunk-based approach divides the grid into smaller, manageable regions, improving memory locality and query performance. Configuration parameters include cell dimensions, grid size, chunk partitioning, and terrain adaptation settings.

**Coordinate Transformation**

The manager provides bidirectional conversion between world space positions and grid coordinates through methods such as [`GetCell()`](/broken/pages/38bc5fdadc2b02da6537009f8feb38cf90d7e51b#methods) and [`CellToWorld()`](/broken/pages/38bc5fdadc2b02da6537009f8feb38cf90d7e51b#methods). These transformations account for the grid's origin point and cell dimensions, enabling precise spatial queries.

**Occupancy Management**

Through the [`GridObjectRegistry`](/broken/pages/59da11b90e311b3709b1a8963f4842113a55f760), the GridManager tracks runtime object placement on the grid. This registry maintains a mapping between cell coordinates and occupying objects, supporting queries such as "what object occupies this cell?" and "what cells does this object occupy?"

**Placement Validation**

Before objects can be placed on the grid, the manager validates whether the target area is free using methods like [`IsAreaFree2D()`](/broken/pages/38bc5fdadc2b02da6537009f8feb38cf90d7e51b#occupancy-queries) and [`IsAreaFree3D()`](/broken/pages/38bc5fdadc2b02da6537009f8feb38cf90d7e51b#occupancy-queries). These methods check both statically baked data (terrain, obstacles) and dynamically registered objects.

**Event System**

The GridManager exposes events that allow other systems to react to grid changes:

* [`OnCellsChanged`](/broken/pages/38bc5fdadc2b02da6537009f8feb38cf90d7e51b#events): Triggered when grid occupancy changes
* [`OnSelectionChanged`](/broken/pages/38bc5fdadc2b02da6537009f8feb38cf90d7e51b#events): Triggered when user selection changes

### BuildingManager

The [`BuildingManager`](/broken/pages/67b757e5378a5a2a151274124bebc5a6f1124aa0) class acts as the central coordinator for all building-related systems. Unlike the GridManager which handles pure spatial concerns, the BuildingManager implements gameplay logic for building placement, selection, and modification.

**Mode Management**

The BuildingManager maintains the current building mode (Build, Sell, Repair, Inactive) and delegates operations to specialized systems. When the mode changes, the manager coordinates UI updates, input system reconfiguration, and grid visualization settings.

**System Coordination**

The BuildingManager instantiates and coordinates several subsystems:

* [`BuildingConstructionSystem`](/broken/pages/1bd967f1c7302651aa163feafb78d532562fa45b): Handles the construction process, including resource deduction and building instantiation
* [`BuildingPreviewSystem`](/broken/pages/8c2ddbfe62eacea6986aa49455fba9c23eb41411): Manages the preview visualization that appears during building placement
* [`BuildingUpgradeSystem`](/broken/pages/b02d6074210771d75ea4241173734d66732a8b92): Implements building upgrade mechanics
* [`BuildingInputSystem`](/broken/pages/e681b1634951ae75d1587aec0619b89e6e00e76a): Processes user input for building operations

**Building Lifecycle**

Through the [`RemoveBuilding()`](/broken/pages/67b757e5378a5a2a151274124bebc5a6f1124aa0#methods) method, the BuildingManager handles the complete removal of buildings from the scene, including grid unregistration and visual cleanup.

## Data Layer

### ScriptableObject Architecture

GridBuilderPro uses ScriptableObjects extensively for data management, providing several advantages:

1. **Editor Integration**: ScriptableObjects automatically appear in the Unity inspector, enabling convenient editing without custom tools.
2. **Asset Pipeline**: Data can be saved as separate assets, enabling version control and modular asset management.
3. **Runtime Access**: Unlike MonoBehaviours, ScriptableObjects can be loaded and accessed without scene dependencies.
4. **Event Propagation**: ScriptableObjects can serve as lightweight event buses using Unity's \[CreateAssetMenu] attribute.

### Key Data Assets

| Asset                                                                           | Purpose                             | Location                       |
| ------------------------------------------------------------------------------- | ----------------------------------- | ------------------------------ |
| [`BuildingDatabaseSO`](/broken/pages/a8668d9b9f0912dc9f50f794ff4a046b6f83b231)  | Catalog of all available buildings  | `GridBuilderPro/Scripts/SO/`   |
| [`BuildingData`](/broken/pages/f96c5196fdb41a2138672584f476e1f6a73cf95c)        | Individual building definitions     | `GridBuilderPro/Scripts/Data/` |
| [`BuildingTags`](/broken/pages/515ef7ba6bb084c36601bf5746034efa9ab7d7ed)        | Enumeration of building categories  | `GridBuilderPro/Scripts/Data/` |
| [`GridBuilderDefaults`](/broken/pages/2ce8bc17269d7a9e061f0f118d57455e671d6e47) | Default prefab references           | `GridBuilderPro/Scripts/SO/`   |
| [`BakedGridData`](/broken/pages/663873cf134a8503553cf494f14b7dfc1db9b71b)       | Pre-computed collision/terrain data | Generated at edit time         |

### Runtime Data Containers

For runtime performance, the system uses struct-based containers that minimize garbage collection:

* `CellData`: Represents a single cell's state including occupancy, position, and validity
* `AreaFreeCheckResult`: Detailed result of placement validation queries

## Systems Layer

### Building Modes

Building modes define the operations players can perform. The architecture supports extensible mode creation through the `IBuildingMode` interface:

**Built-in Modes**

* **Build Mode**: Validates placement, shows preview, and initiates construction when confirmed
* **Sell Mode**: Removes buildings and returns resources based on sell value calculations
* **Repair Mode**: Restores building health with associated costs

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

The [`BuildingConstructionSystem`](../building-systems/construction-system.md), [`BuildingPreviewSystem`](../building-systems/preview-system.md), and [`BuildingUpgradeSystem`](../building-systems/upgrade-system.md) handle specific aspects of building management. Each system operates independently but receives coordinated input from the BuildingManager.

### Input Handling

The [`BuildingInputSystem`](../building-systems/input-system.md) translates user input (mouse clicks, keyboard shortcuts) into building operations. This abstraction allows for different input schemes without modifying core building logic.

## Visualization Layer

### Grid Visualization

The grid visualization system uses an interface-based approach, allowing different visual implementations:

* [`GridVisualizerBase`](/broken/pages/7250a8bd1bcfacce28cc742472a221a928a09736): Abstract base class for grid mesh generation
* [`GridHightlightProjector`](/broken/pages/ae68aaf24429152d66d4ba505cc374bc5fbc1c94): Handles cell highlighting using decal projectors

### Building Effects

The [`BuildingSpawnMaterializer`](/broken/pages/463dbebaad439234cc820e56d4a2cfcdbd431fae) provides visual feedback during building construction, spawning effects at the building location.

## Editor Tools Layer

GridBuilderPro includes comprehensive editor tooling for project configuration:

| Tool                                                                                 | Purpose                      | Access     |
| ------------------------------------------------------------------------------------ | ---------------------------- | ---------- |
| [`GridBuilderCreator`](/broken/pages/a081d4b114eed0035281fd2fcd808139424dec90)       | Initial project setup wizard | Unity Menu |
| [`GridBuilderSetupWindow`](/broken/pages/e7fa4e56430a7bfe62a33ef35818028219572431)   | Advanced configuration       | Unity Menu |
| [`BuildingDatabaseSOEditor`](/broken/pages/628c7d517da5d68a505bb6aa31add050610b638d) | Building catalog management  | Inspector  |
| [`BuildingDataEditorWindow`](/broken/pages/2e8aee19cf6e8abffa2221804ae19b6e5ed75bb4) | Individual building editing  | Inspector  |

## Performance Considerations

### Zero GC Design

GridBuilderPro is engineered for zero garbage collection during gameplay through several techniques:

1. **Struct-Based Collections**: All hot-path data structures use structs rather than classes to avoid heap allocations.
2. **Pre-allocated Arrays**: Memory is allocated during initialization rather than at runtime.
3. **Object Pooling**: Building instances and visual effects are pooled rather than instantiated and destroyed.
4. **Interface-Based Iteration**: Collections expose `IReadOnlyList<T>` interfaces to enable efficient enumeration without allocation.

### Spatial Query Optimization

The chunk-based grid data structure enables efficient spatial queries:

* **O(1) Lookups**: Individual cell queries use direct array indexing
* **Chunk-Based Iteration**: Area queries iterate only over affected chunks
* **Bitset Storage**: Blocked cell data uses compressed bitsets for memory efficiency

### Baked Data

Static grid data (terrain, obstacles) is pre-computed at edit time and serialized to assets, eliminating runtime raycasting and collision checks.

## Extensibility

### Interface-Based Extension Points

| Interface                                                                         | Purpose                 | Extension Example                 |
| --------------------------------------------------------------------------------- | ----------------------- | --------------------------------- |
| [`IGridVisualizer`](/broken/pages/0b915016670f3da5f73a501117fcf0bda4b03db4)       | Custom grid rendering   | Custom shader-based visualization |
| [`IGridHighlight`](/broken/pages/779c9e143e435392346290220ac510a2e6eda3d5)        | Custom highlighting     | Particle-based effects            |
| [`IBuildingMode`](/broken/pages/129de2b291b98035c3b461bb99afa46baf18e791)         | New building operations | Demolish, Combine modes           |
| [`IBuildingActionTarget`](/broken/pages/30174466cacd3a4114cf9ad41c439bfbdb545176) | Building responses      | Custom destruction effects        |

