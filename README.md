# Overview

This section contains detailed documentation for all building systems in GridBuilderPro.

## Overview

The building systems handle all aspects of the building lifecycle, from initial placement through construction, upgrades, and eventual removal. Each system is designed with a specific responsibility, allowing for clean separation of concerns and easy extensibility.

## System Architecture

<figure><img src=".gitbook/assets/System Schema #1.png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/System Schema #2.png" alt=""><figcaption></figcaption></figure>

### Core Systems

| Topic                                                            | Description                                  |
| ---------------------------------------------------------------- | -------------------------------------------- |
| [BuildingManager](grid-builder-pro/building-manager.md)          | Central coordinator for all building systems |
| [GridManager](architecture/grid-manager.md)                      | Grid data system                             |
| [BuildingInputSystem](grid-builder-pro/building-input-system.md) | User input processing and mode switching     |

### Building Modes

| Topic                                                          | Description                      |
| -------------------------------------------------------------- | -------------------------------- |
| [BuildingModeBuild](grid-builder-pro/building-mode-build.md)   | Standard building placement mode |
| [BuildingModeSell](grid-builder-pro/building-mode-sell.md)     | Building sell and removal mode   |
| [BuildingModeRepair](grid-builder-pro/building-mode-repair.md) | Building repair mode             |

### Support Systems

| Topic                                                                          | Description                   |
| ------------------------------------------------------------------------------ | ----------------------------- |
| [BuildingPreviewSystem](grid-builder-pro/building-preview-system.md)           | Ghost/preview visualization   |
| [BuildingConstructionSystem](grid-builder-pro/building-construction-system.md) | Construction timer management |
| [BuildingUpgradeSystem](grid-builder-pro/building-upgrade-system.md)           | Building upgrade system       |

## Related Documentation

| Topic                                 | Description           |
| ------------------------------------- | --------------------- |
| [Economy System](economy/overview.md) | Resource management   |
| Building UI Manager                   | Building selection UI |
