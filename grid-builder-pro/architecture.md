# Architecture

## System Overview

GridBuilderPro follows a modular, layered architecture that separates concerns between grid infrastructure, building mechanics, and user interface. This design enables flexibility, maintainability, and performance optimization by ensuring each system operates independently while communicating through well-defined interfaces.

The architecture is built around three primary layers:

* **Grid Layer** (`CorePro.GridSystem`): Provides the fundamental spatial infrastructure including cell management, occupancy tracking, and grid visualization.
* **Building Layer** (`CorePro.GridBuilderPro`): Implements building-specific logic including placement, selection, upgrades, and lifecycle management.
* **UI Layer** (`CorePro.GridBuilderPro.UI`): Handles user interaction panels, building selection, and contextual information display.

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
│                              GridBuilderPro Core                                │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │                           GridSystem Layer                              │    │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │    │
│  │  │ GridManager  │  │ GridData*    │  │ GridVisual*  │  │ GridHighlight│ │    │
│  │  │ (Singleton)  │  │ (Interface)  │  │ (Interface) │  │ (Interface)   │ │    │
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

The [`GridManager`](/broken/pages/902d748d75a3b0bdcc43d19411ee9e75bb37d2d3) class serves as the central authority for all grid-based operations. As a Singleton pattern implementation, it provides global access to grid functionality throughout the game. The GridManager handles several critical responsibilities:

*   Grid Configuration and Initialization

    During initialization, the GridManager creates a [`GridDataChunked3D`](/broken/pages/ac425541c899eee5b2e36d112fb3582bd4ac1f22) instance that manages the underlying grid data structure. The chunk-based approach divides the grid into smaller, manageable regions, improving memory locality and query performance. Configuration parameters include cell dimensions, grid size, chunk partitioning, and terrain adaptation settings.
*   Coordinate Transformation

    The manager provides bidirectional conversion between world space positions and grid coordinates through methods such as [`GetCell()`](/broken/pages/902d748d75a3b0bdcc43d19411ee9e75bb37d2d3#methods) and [`CellToWorld()`](/broken/pages/902d748d75a3b0bdcc43d19411ee9e75bb37d2d3#methods). These transformations account for the grid's origin point and cell dimensions, enabling precise spatial queries.
*   Occupancy Management

    Through the [`GridObjectRegistry`](/broken/pages/ab291b3cc0732da0a2bc99e0fc400617a6a605b5), the GridManager tracks runtime object placement on the grid. This registry maintains a mapping between cell coordinates and occupying objects, supporting queries such as "what object occupies this cell?" and "what cells does this object occupy?"
*   Placement Validation

    Before objects can be placed on the grid, the manager validates whether the target area is free using methods like [`IsAreaFree2D()`](/broken/pages/902d748d75a3b0bdcc43d19411ee9e75bb37d2d3#occupancy-queries) and [`IsAreaFree3D()`](/broken/pages/902d748d75a3b0bdcc43d19411ee9e75bb37d2d3#occupancy-queries). These methods check both statically baked data (terrain, obstacles) and dynamically registered objects.
*   Event System

    The GridManager exposes events that allow other systems to react to grid changes:

    * [`OnCellsChanged`](/broken/pages/902d748d75a3b0bdcc43d19411ee9e75bb37d2d3#events): Triggered when grid occupancy changes
    * [`OnSelectionChanged`](/broken/pages/902d748d75a3b0bdcc43d19411ee9e75bb37d2d3#events): Triggered when user selection changes

### BuildingManager

The [`BuildingManager`](/broken/pages/39b9b45f38392cbc1b10375392aab3674bf088d4) class acts as the central coordinator for all building-related systems. Unlike the GridManager which handles pure spatial concerns, the BuildingManager implements gameplay logic for building placement, selection, and modification.

*   Mode Management

    The BuildingManager maintains the current building mode (Build, Sell, Repair, Inactive) and delegates operations to specialized systems. When the mode changes, the manager coordinates UI updates, input system reconfiguration, and grid visualization settings.
*   System Coordination

    The BuildingManager instantiates and coordinates several subsystems:

    * [`BuildingConstructionSystem`](/broken/pages/b13285f34e8b541dd351598be6b4dcbef173d982): Handles the construction process, including resource deduction and building instantiation
    * [`BuildingPreviewSystem`](/broken/pages/5383c41db5978b203f1791177d3cc74dfc51cccf): Manages the preview visualization that appears during building placement
    * [`BuildingUpgradeSystem`](/broken/pages/539fff3167d235c7bafd7abb465140708ff87f47): Implements building upgrade mechanics
    * [`BuildingInputSystem`](/broken/pages/0ad1457c01e707fa5d540fafc1c568ca9d3c8560): Processes user input for building operations
*   Building Lifecycle

    Through the [`RemoveBuilding()`](/broken/pages/39b9b45f38392cbc1b10375392aab3674bf088d4#methods) method, the BuildingManager handles the complete removal of buildings from the scene, including grid unregistration and visual cleanup.

## Data Layer

### ScriptableObject Architecture

GridBuilderPro uses ScriptableObjects extensively for data management, providing several advantages:

1. Editor Integration: ScriptableObjects automatically appear in the Unity inspector, enabling convenient editing without custom tools.
2. Asset Pipeline: Data can be saved as separate assets, enabling version control and modular asset management.
3. Runtime Access: Unlike MonoBehaviours, ScriptableObjects can be loaded and accessed without scene dependencies.
4. Event Propagation: ScriptableObjects can serve as lightweight event buses using Unity's \[CreateAssetMenu] attribute.

### Key Data Assets

| Asset                                                                           | Purpose                             | Location                       |
| ------------------------------------------------------------------------------- | ----------------------------------- | ------------------------------ |
| [`BuildingDatabaseSO`](/broken/pages/d19686dd9f75c0bdc878bcf607d29de6f8ba1fd8)  | Catalog of all available buildings  | `GridBuilderPro/Scripts/SO/`   |
| [`BuildingData`](/broken/pages/d66c2772b2abf99c8b0b77a3baa459173459493b)        | Individual building definitions     | `GridBuilderPro/Scripts/Data/` |
| [`BuildingTags`](/broken/pages/2a2d79434cacb32fd45ba9243500fafbc44eb4b5)        | Enumeration of building categories  | `GridBuilderPro/Scripts/Data/` |
| [`GridBuilderDefaults`](/broken/pages/0b281596fb78ed07a1c8fe6f071191c3456d2121) | Default prefab references           | `GridBuilderPro/Scripts/SO/`   |
| [`BakedGridData`](/broken/pages/07db74d699f16ea85cafc9946a01b8bf622782bc)       | Pre-computed collision/terrain data | Generated at edit time         |

### Runtime Data Containers

For runtime performance, the system uses struct-based containers that minimize garbage collection:

* [`CellData`](/broken/pages/9d14546e5fa0c0b65bd8fe11e53ba2683bf32cb9): Represents a single cell's state including occupancy, position, and validity
* [`AreaFreeCheckResult`](/broken/pages/3d45d3b438c26ce3f7a304e93a83bb69aa168169): Detailed result of placement validation queries

## Systems Layer

### Building Modes

Building modes define the operations players can perform. The architecture supports extensible mode creation through the [`IBuildingMode`](/broken/pages/cb9095198cdf52ea4ded853e81f6c1802ae4c5af) interface.

Built-in Modes:

* Build Mode: Validates placement, shows preview, and initiates construction when confirmed
* Sell Mode: Removes buildings and returns resources based on sell value calculations
* Repair Mode: Restores building health with associated costs

Custom Mode Implementation example:

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

The [`BuildingConstructionSystem`](/broken/pages/b13285f34e8b541dd351598be6b4dcbef173d982), [`BuildingPreviewSystem`](/broken/pages/5383c41db5978b203f1791177d3cc74dfc51cccf), and [`BuildingUpgradeSystem`](/broken/pages/539fff3167d235c7bafd7abb465140708ff87f47) handle specific aspects of building management. Each system operates independently but receives coordinated input from the BuildingManager.

### Input Handling

The [`BuildingInputSystem`](/broken/pages/0ad1457c01e707fa5d540fafc1c568ca9d3c8560) translates user input (mouse clicks, keyboard shortcuts) into building operations. This abstraction allows for different input schemes without modifying core building logic.

## Visualization Layer

### Grid Visualization

The grid visualization system uses an interface-based approach, allowing different visual implementations:

* [`GridVisualizerBase`](/broken/pages/b6c1b538de58a3cfd37a10b0d4b8b6c74ba6a9e0): Abstract base class for grid mesh generation
* [`GridHightlightProjector`](/broken/pages/516c952bb55ae70f41bdeb03be654e910c2cdbd9): Handles cell highlighting using decal projectors

### Building Effects

The [`BuildingSpawnMaterializer`](/broken/pages/cd06e2577a70210af3c74b1c1dd35d492870b20d) provides visual feedback during building construction, spawning effects at the building location.

## Editor Tools Layer

GridBuilderPro includes comprehensive editor tooling for project configuration:

| Tool                                                                                 | Purpose                      | Access     |
| ------------------------------------------------------------------------------------ | ---------------------------- | ---------- |
| [`GridBuilderCreator`](/broken/pages/d11e687b6e22f79f13b6049cc080451604f43830)       | Initial project setup wizard | Unity Menu |
| [`GridBuilderSetupWindow`](/broken/pages/2f89fc8a2295c20ce34f07cc36b5caf4c55a5d3b)   | Advanced configuration       | Unity Menu |
| [`BuildingDatabaseSOEditor`](/broken/pages/3a40009d9341715612c2e84aedec593b82e86721) | Building catalog management  | Inspector  |
| [`BuildingDataEditorWindow`](/broken/pages/f715abd4a3689af4da012d3672cd0c27096dc96e) | Individual building editing  | Inspector  |

## Performance Considerations

### Zero GC Design

GridBuilderPro is engineered for zero garbage collection during gameplay through several techniques:

1. Struct-Based Collections: All hot-path data structures use structs rather than classes to avoid heap allocations.
2. Pre-allocated Arrays: Memory is allocated during initialization rather than at runtime.
3. Object Pooling: Building instances and visual effects are pooled rather than instantiated and destroyed.
4. Interface-Based Iteration: Collections expose `IReadOnlyList<T>` interfaces to enable efficient enumeration without allocation.

### Spatial Query Optimization

The chunk-based grid data structure enables efficient spatial queries:

* O(1) Lookups: Individual cell queries use direct array indexing
* Chunk-Based Iteration: Area queries iterate only over affected chunks
* Bitset Storage: Blocked cell data uses compressed bitsets for memory efficiency

### Baked Data

Static grid data (terrain, obstacles) is pre-computed at edit time and serialized to assets, eliminating runtime raycasting and collision checks.

## Extensibility

### Interface-Based Extension Points

| Interface                                                                         | Purpose                 | Extension Example                 |
| --------------------------------------------------------------------------------- | ----------------------- | --------------------------------- |
| [`IGridVisualizer`](/broken/pages/c05fe36a8a4ac4fd134188d533869159fb251d5b)       | Custom grid rendering   | Custom shader-based visualization |
| [`IGridHighlight`](/broken/pages/a9c0fe1db16fb25cb369412332e75d5b42400d70)        | Custom highlighting     | Particle-based effects            |
| [`IBuildingMode`](/broken/pages/cb9095198cdf52ea4ded853e81f6c1802ae4c5af)         | New building operations | Demolish, Combine modes           |
| [`IBuildingActionTarget`](/broken/pages/6eda8394bdb4cc5fea4a16774d54256f5c845163) | Building responses      | Custom destruction effects        |

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

| Topic                                                                     | Description                            |
| ------------------------------------------------------------------------- | -------------------------------------- |
| [GridManager](/broken/pages/902d748d75a3b0bdcc43d19411ee9e75bb37d2d3)     | Complete GridManager API reference     |
| [BuildingManager](/broken/pages/39b9b45f38392cbc1b10375392aab3674bf088d4) | Complete BuildingManager API reference |
| [Grid Data](/broken/pages/ac425541c899eee5b2e36d112fb3582bd4ac1f22)       | Grid data structures and algorithms    |
| [Building Data](/broken/pages/d66c2772b2abf99c8b0b77a3baa459173459493b)   | Building definition assets             |
| [Editor Tools](/broken/pages/7ffffdc0bbb4623ca86761e787ddfaf38a4a0b99)    | Editor integration documentation       |
| [API Reference](/broken/pages/7b721d9e16a03ecf49f9e298386302f15420a0fd)   | Complete API documentation             |
