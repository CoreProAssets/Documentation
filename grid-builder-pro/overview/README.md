# Overview

This section contains detailed documentation for all building systems in GridBuilderPro.

## Overview

The building systems handle all aspects of the building lifecycle, from initial placement through construction, upgrades, and eventual removal. Each system is designed with a specific responsibility, allowing for clean separation of concerns and easy extensibility.

## System Architecture

```
BuildingManager
     │
     ├── BuildingConstructionSystem
     │    │    Handles: Construction timers, completion callbacks
     │  
     │
     ├── BuildingPreviewSystem
     │    │    Handles: Ghost visualization, material swapping
     │    
     │
     ├── BuildingUpgradeSystem
     │    │    Handles: Building evolution, level transitions
     │    
     │
     ├── BuildingInputSystem
     │    │    Handles: User input processing
     │    
     │
     └── Building Modes
          │
          ├── BuildingModeBuild
          │    │    Handles: Placement, rotation, validation
          │
          ├── BuildingModeSell
          │    │    Handles: Building removal, resource refunds
          │
          └── BuildingModeRepair
               │    Handles: Health restoration, repair costs
```

## Documentation Index

### Core Systems

| Topic                                              | Description                                  |
| -------------------------------------------------- | -------------------------------------------- |
| [BuildingManager](../building-manager.md)          | Central coordinator for all building systems |
| [BuildingInputSystem](../building-input-system.md) | User input processing and mode switching     |

### Building Modes

| Topic                                            | Description                      |
| ------------------------------------------------ | -------------------------------- |
| [BuildingModeBuild](../building-mode-build.md)   | Standard building placement mode |
| [BuildingModeSell](../building-mode-sell.md)     | Building sell and removal mode   |
| [BuildingModeRepair](../building-mode-repair.md) | Building repair mode             |

### Support Systems

| Topic                                                            | Description                   |
| ---------------------------------------------------------------- | ----------------------------- |
| [BuildingPreviewSystem](../building-preview-system.md)           | Ghost/preview visualization   |
| [BuildingConstructionSystem](../building-construction-system.md) | Construction timer management |
| [BuildingUpgradeSystem](../building-upgrade-system.md)           | Building upgrade system       |

## Related Documentation

| Topic                                                   | Description                    |
| ------------------------------------------------------- | ------------------------------ |
| [Architecture Overview](../../architecture/overview.md) | High-level system architecture |
| [Data Components](../../data/overview.md)               | Building data assets           |
| [GridManager](../../architecture/grid-manager.md)       | Grid management system         |
| [Economy System](../../economy/overview.md)             | Resource management            |
| [UI Components](../../ui/overview.md)                   | Building selection UI          |
