# Introduction

## Overview

**GridBuilderPro** is a comprehensive, performance-optimized grid-based building system for Unity. Designed specifically for mobile platforms, this system provides a robust foundation for creating games that require grid-based building mechanics, such as city builders, tower defense games, RTS titles, and base construction simulations.

The system is built with a strict focus on **Zero Garbage Collection (GC)** during runtime, ensuring optimal CPU performance and minimal battery consumption on mobile devices. GridBuilderPro leverages Data-Oriented Design principles and utilizes ScriptableObjects for efficient data management, making it an ideal choice for performance-conscious developers.

At its core, GridBuilderPro separates the concerns between the underlying grid infrastructure and the building mechanics. The GridSystem provides a flexible, serialized grid data structure that can be pre-baked and efficiently queried at runtime, while the BuildingManager handles all aspects of building placement, selection, upgrades, and lifecycle management.

![GridBuilderPro Overview](images/overview-diagram.png)

## Key Features

### Performance-First Architecture

GridBuilderPro is engineered from the ground up with performance as a primary concern. Every system has been designed to minimize allocations and avoid common performance pitfalls that plague many Unity projects. The system employs several advanced techniques to achieve optimal performance:

- **Zero GC Runtime**: All hot paths are carefully optimized to generate zero garbage collection during gameplay. This is achieved through the use of struct-based data containers, pre-allocated arrays, and object pooling strategies that eliminate dynamic memory allocation during gameplay.

- **Data-Oriented Design**: The system uses ScriptableObjects as the primary data container, allowing for clean separation between data and behavior. Grid data is stored in a cache-friendly format using struct-based arrays that minimize cache misses during iteration.

- **Baked Grid Data**: Grid information can be pre-computed and serialized at edit time, eliminating the need for expensive runtime calculations. This baked data includes cell occupancy information, building references, and spatial queries.

### Flexible Building System

The building mechanics in GridBuilderPro are designed to be highly flexible and extensible. Whether you need simple tower placement or complex multi-story structures with dependencies, the system can accommodate your requirements:

- **Multi-Size Buildings**: Support for buildings of various sizes, from single cells to large multi-tile structures. The system automatically handles collision detection and placement validation for buildings of any dimensions.

- **Multiple Building Modes**: Built-in support for different building operations including construction, selling, and repair. Each mode can be customized or extended to implement additional behaviors.

- **Building Tags and Categories**: A robust tagging system allows buildings to be organized into categories and filtered by various criteria. This enables sophisticated UI filtering systems and game logic that depends on building types.

- **Upgrade System**: An extensible upgrade system allows buildings to be improved over time. Upgrades can modify building stats, change visual appearance, or unlock new capabilities.

### Editor Integration

GridBuilderPro provides extensive editor tooling to streamline your workflow and reduce development time. All aspects of the system can be configured through Unity's editor, with custom inspectors and specialized windows designed for each component:

- **GridBuilderCreator**: A comprehensive setup wizard that guides you through the initial configuration of the grid system. This tool automatically creates all necessary components and assigns default values based on your project requirements.

- **GridBuilderSetupWindow**: An advanced configuration window that provides fine-grained control over all aspects of the grid system. This includes layer configuration, visual settings, and performance tuning options.

- **BuildingDatabaseSOEditor**: A specialized editor for managing your building database. It provides a clean interface for adding, editing, and organizing buildings, with support for categories, tags, and bulk operations.

- **BuildingDataEditorWindow**: A detailed editor for individual building configurations. This window provides access to all building properties including costs, requirements, behaviors, and visual settings.

### Visual Feedback System

Providing clear visual feedback to players is essential for a good building experience. GridBuilderPro includes a comprehensive visual system that communicates placement validity, selection states, and building status:

- **Real-Time Preview**: Players can see exactly how a building will look before placing it. The preview system shows the building at the cursor position, updating in real-time as the player moves across the grid.

- **Placement Validation**: Visual indicators clearly communicate whether a building can be placed at a given location. Different colors and effects indicate valid placement, invalid placement, and the specific reasons for placement failure.

- **Selection Highlighting**: Selected buildings are highlighted with clear visual feedback. The selection system supports multiple selection modes and can display contextual information about selected objects.

- **Construction Effects**: A materializer system provides satisfying visual feedback when buildings are constructed, destroyed, or upgraded. These effects can be customized to match your game's visual style.

### Extensibility

GridBuilderPro is designed to be extended. Every major system uses interfaces and virtual methods that allow you to customize behavior without modifying the core codebase:

- **Custom Building Actions**: Implement the IBuildingActionTarget interface to create buildings that respond to player actions in custom ways.

- **Building Modes**: Create new building modes by implementing the IBuildingMode interface. This allows you to add new operations beyond the built-in build, sell, and repair modes.

- **Grid Visualizers**: Extend the grid visualization by implementing the IGridVisualizer interface. This enables custom grid overlays, effects, and debug visualizations.

- **Custom Commands**: The command system can be extended to implement new actions that can be performed on buildings. Commands support conditions, costs, and execution logic.

## Requirements

### Unity Version

GridBuilderPro requires **Unity 2021.3 LTS or newer**. The system has been tested with the following Unity versions:

- Unity 2021.3 LTS (recommended)
- Unity 2022.3 LTS
- Unity 6000.0+

### Render Pipeline

The system is designed to work with **Universal Render Pipeline (URP)** and includes specific optimizations for this pipeline. While core functionality should work with other render pipelines, some visual features like decal projector support require URP.

- URP 12.0 or newer (recommended)
- Built-in Render Pipeline (limited features)
- HDRP (limited features)

### Platform Support

GridBuilderPro is designed with mobile platforms as the primary target, but supports all platforms that Unity supports:

- **Primary Target**: iOS, Android
- **Supported**: Windows, macOS, Linux, WebGL
- **Experimental**: Consoles (requires platform-specific considerations)

### Dependencies

GridBuilderPro has minimal external dependencies to ensure easy integration into existing projects:

- **TextMeshPro** (Unity package): Required for all UI components
- **Unity Input System** (optional): Can be used for input handling
- **Addressables** (optional): For advanced asset management scenarios

## Core Concepts

Understanding these core concepts will help you make the most of GridBuilderPro:

### The Grid

The grid is the fundamental spatial structure that underlies the entire system. It divides the game world into discrete cells, each of which can contain information about occupancy, building references, and custom data. The grid is defined by:

- **Cell Size**: The dimensions of each cell in world units. This determines how the grid maps to world space coordinates.
- **Grid Dimensions**: The number of cells in each dimension (width, height, and optionally depth).
- **Origin Point**: The world position of cell (0, 0, 0). This establishes the mapping between grid coordinates and world positions.

The grid can be visualized using the built-in visualizer system, which provides various visualization options including grid lines, cell highlighting, and selection indicators.

### Buildings

Buildings are the player-created objects that populate the grid. Each building is defined by:

- **Building Data**: A ScriptableObject that defines the building's properties including name, description, cost, size, and behaviors.
- **Prefab**: The GameObject that is instantiated when the building is placed in the world.
- **Building Instance**: The runtime instance of a building, which tracks state, health, and instance-specific data.

Buildings can be of various sizes, occupying a single cell or multiple cells depending on their configuration. The system automatically manages collision detection and placement validation for buildings of any size.

### Building Modes

Building modes define the operations that players can perform on buildings. The built-in modes are:

- **Build Mode**: Allows players to place new buildings on the grid. Validates placement, shows previews, and handles construction.
- **Sell Mode**: Allows players to remove buildings and recover resources. Can include selling discounts or additional conditions.
- **Repair Mode**: Allows players to restore damaged buildings to full health. May have costs and conditions.

### Building Tags

Building tags provide a flexible way to categorize and identify buildings. Tags can be used for:

- Filtering buildings in UI panels
- Implementing game logic that depends on building types
- Creating dependencies between buildings
- Implementing achievements or progression systems

Tags are highly flexible and can be created and configured in the editor without coding.

## Quick Links

| Topic | Description |
|-------|-------------|
| [Quick Start](quick-start.md) | Get up and running in 5 minutes |
| [Architecture Overview](architecture/overview.md) | Deep dive into system architecture |
| [GridManager](architecture/grid-manager.md) | Core grid management |
| [BuildingDatabaseSO](data/building-database.md) | Managing your building catalog |
| [API Reference](api/) | Complete API documentation |

## Support

If you encounter issues or have questions:

- **Documentation**: Search this documentation for answers
- **FAQ**: Check common questions and solutions
- **Issues**: Report bugs through the issue tracker

## License

GridBuilderPro is available under a commercial license. See the [License](license.md) page for full details.
