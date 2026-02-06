# GridBuilderPro Documentation Summary

This document provides an overview of all documentation files available for GridBuilderPro.

## Getting Started

| File | Description |
|------|-------------|
| [Introduction](introduction.md) | Overview of GridBuilderPro, key features, requirements, and core concepts |
| [Quick Start](quick-start.md) | Step-by-step guide to set up your first GridBuilderPro project |

## Architecture

| File | Description |
|------|-------------|
| [Architecture Overview](architecture/overview.md) | High-level system architecture and component relationships |
| [GridManager](architecture/grid-manager.md) | Complete API reference for GridManager - central grid authority |

## Data Components

| File | Description |
|------|-------------|
| [Data Overview](data/overview.md) | Overview of ScriptableObject-based data architecture |
| [BuildingData](data/building-data.md) | Individual building definitions |
| [BuildingDatabaseSO](data/building-database.md) | Building catalog and runtime queries |
| [BuildingTags](data/building-tags.md) | Category enumeration with flags attribute |
| [BuildingCategory](data/building-category.md) | UI organization for buildings |
| [GridBuilderDefaults](data/grid-builder-defaults.md) | Global default settings and prefabs |

## Building Systems

| File | Description |
|------|-------------|
| [Systems Overview](systems/overview.md) | Index and architecture of all building systems |
| [BuildingManager](systems/building-manager.md) | Central coordinator for building operations |
| [BuildingModeBuild](systems/building-mode-build.md) | Building placement mode |
| [BuildingModeSell](systems/building-mode-sell.md) | Building sell and removal mode |
| [BuildingModeRepair](systems/building-mode-repair.md) | Building repair mode |
| [BuildingPreviewSystem](systems/building-preview-system.md) | Ghost/preview visualization |
| [BuildingConstructionSystem](systems/building-construction-system.md) | Construction timer management |
| [BuildingUpgradeSystem](systems/building-upgrade-system.md) | Building upgrade system |
| [BuildingInputSystem](systems/building-input-system.md) | User input processing |

## Editor Tools

| File | Description |
|------|-------------|
| [Editor Overview](editor/overview.md) | Overview of editor tools and utilities |
| [GridBuilderCreator](editor/creator.md) | Initial project setup wizard |
| [GridBuilderSetupWindow](editor/setup-window.md) | Advanced configuration window |
| [BuildingDatabaseSOEditor](editor/database-editor.md) | Building catalog management |

## API Reference

| File | Description |
|------|-------------|
| [API Overview](api/overview.md) | Complete API documentation |

---

## File Structure

```
docs/
├── introduction.md
├── quick-start.md
├── SUMMARY.md
├── architecture/
│   ├── overview.md
│   └── grid-manager.md
├── data/
│   ├── overview.md
│   ├── building-data.md
│   ├── building-database.md
│   ├── building-tags.md
│   ├── building-category.md
│   └── grid-builder-defaults.md
├── systems/
│   ├── overview.md
│   ├── building-manager.md
│   ├── building-mode-build.md
│   ├── building-mode-sell.md
│   ├── building-mode-repair.md
│   ├── building-preview-system.md
│   ├── building-construction-system.md
│   ├── building-upgrade-system.md
│   └── building-input-system.md
└── editor/
    └── overview.md
```

---

## Quick Links

- **Start Here**: [Introduction](introduction.md)
- **First Project**: [Quick Start](quick-start.md)
- **Core Systems**: [Architecture Overview](architecture/overview.md)
- **Grid Management**: [GridManager](architecture/grid-manager.md)
- **Building Data**: [Data Overview](data/overview.md)
- **Building Operations**: [Systems Overview](systems/overview.md)

---

*Last updated: 2024*
