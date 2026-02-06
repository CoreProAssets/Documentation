# Data

## Introduction

GridBuilderPro uses a comprehensive data architecture based on ScriptableObjects to define buildings, categories, and system defaults. This approach provides several key advantages:

* **Editor Integration**: All data assets appear in the Unity Inspector, enabling convenient editing without custom tools
* **Asset Management**: Data can be version-controlled, shared between projects, and organized in folders
* **Runtime Performance**: Pre-serialized data avoids runtime parsing or calculation overhead
* **Clean Separation**: Data is separate from behavior, following the composition over inheritance principle

The data layer consists of several interconnected asset types that work together to define the building catalog and system configuration.

## Asset Hierarchy

```
GridBuilderDefaults (Global Settings)
│
├── BuildingDatabaseSO (Building Catalog)
│   └── BuildingData[] (Individual Buildings)
│       ├── BuildingTags (Category Flags)
│       ├── BuildingType (Functional Classification)
│       └── BuildingInstance Prefab (Runtime Object)
│
└── BuildingCategory[] (Optional UI Organization)
```

## BuildingData

The [`BuildingData`](/broken/pages/3b87c48fcaee9051ec83ec6ee70761c11b72c973) asset is the fundamental definition of a building in the system. Each building in your game should have a corresponding BuildingData asset that defines its properties, costs, and behaviors.

### Core Properties

| Property       | Type                                                                         | Description                                        |
| -------------- | ---------------------------------------------------------------------------- | -------------------------------------------------- |
| `buildingTags` | [`BuildingTags`](/broken/pages/48e194224aea0517cf90ffcbddf150230ad974c1)     | Category flags used for filtering and organization |
| `buildingType` | [`BuildingType`](/broken/pages/65498fc919693755d3ddb60c0b18d931fce3cea3)     | Functional classification of the building          |
| `buildingName` | string                                                                       | Display name shown in UI                           |
| `prefab`       | [`BuildingInstance`](/broken/pages/abf42fe67b949276f4f158d5a4b5f35c30714482) | Prefab instantiated when building is placed        |
| `icon`         | Sprite                                                                       | Thumbnail displayed in building selection UI       |
| `description`  | string                                                                       | Detailed description for UI panels                 |
| `sortWeight`   | int                                                                          | Ordering priority (lower values appear first)      |

### Build Properties

| Property              | Type                   | Description                                  |
| --------------------- | ---------------------- | -------------------------------------------- |
| `buildCost`           | `ResourceAmount[]`     | Array of resources required to build         |
| `constructionTime`    | float                  | Time in seconds for construction to complete |
| `effectsOnStartBuild` | `ConstructionEffect[]` | Visual effects spawned at construction start |

### Economy Properties

| Property               | Type               | Description                                         |
| ---------------------- | ------------------ | --------------------------------------------------- |
| `sellRefundPercentage` | float (0-1)        | Portion of build cost returned when selling         |
| `customSellRewards`    | `ResourceAmount[]` | Optional custom sell rewards (overrides percentage) |
| `repairFullCost`       | `ResourceAmount[]` | Resources required for full repair                  |

### Upgrade Properties

| Property        | Type                                                                     | Description                           |
| --------------- | ------------------------------------------------------------------------ | ------------------------------------- |
| `level`         | int                                                                      | Current upgrade level of the building |
| `nextLevelData` | [`BuildingData`](/broken/pages/3b87c48fcaee9051ec83ec6ee70761c11b72c973) | Reference to next level's data asset  |

### Connection Properties

| Property         | Type                                                                               | Description                                     |
| ---------------- | ---------------------------------------------------------------------------------- | ----------------------------------------------- |
| `connectionData` | [`BuildingConnectionData`](/broken/pages/e2e76dac783cee49f8d5d423aa5bf64f672354f0) | Optional configuration for neighbor connections |

### Usage Example

```csharp
// Accessing building data properties
BuildingData data = buildingInstance.GetComponent<BuildingInstance>().GetBuildingData();

// Check if player can afford to build
bool canAfford = data.CanSpendToBuild();

// Get formatted cost description for UI
string costText = data.GetCostDescription();

// Check if building can be upgraded
bool hasUpgrade = data.HasUpgrade();
```

## BuildingDatabaseSO

The [`BuildingDatabaseSO`](/broken/pages/1619048e7d6a9d21f5431ccb024e48cac4fb0c02) asset serves as the central catalog of all available buildings in your project. It provides efficient runtime queries for filtering buildings by category and type.

### Key Features

* **Automatic Category Baking**: Extracts unique categories from all buildings at edit time
* **Runtime Lookup Optimization**: Creates cached dictionaries for zero-alloc queries
* **Sorting**: Automatically sorts buildings by `sortWeight` during initialization
* **Editor Integration**: Provides buttons for syncing and validating the database

### Runtime API

```csharp
// Initialize the database (call once at startup)
BuildingDatabaseSO.Instance.Initialize();

// Get all buildings in a category
IReadOnlyList<BuildingData> defensiveBuildings = 
    BuildingDatabaseSO.Instance.GetBuildingsByCategory(BuildingTags.Defence);

// Get buildings filtered by both category and type
IReadOnlyList<BuildingData> defensiveTowers = 
    BuildingDatabaseSO.Instance.GetBuildingsByCategoryAndType(
        BuildingTags.Defence, 
        BuildingType.Tower
    );

// Get buildings by type only
IReadOnlyList<BuildingData> allTowers = 
    BuildingDatabaseSO.Instance.GetBuildingsByType(BuildingType.Tower);
```

### Editor Management

The database provides several editor utilities accessed through the Inspector:

| Button                      | Action                                                  |
| --------------------------- | ------------------------------------------------------- |
| **Auto-Find All Buildings** | Scans project for all BuildingData assets and adds them |
| **Bake Categories Only**    | Recalculates category list without modifying buildings  |
| **Validate And Clean**      | Removes null entries from the building list             |

## BuildingTags

The [`BuildingTags`](/broken/pages/48e194224aea0517cf90ffcbddf150230ad974c1) enum uses the `[Flags]` attribute to provide a bitwise category system. This allows buildings to belong to multiple categories simultaneously.

### Predefined Categories

```csharp
[Flags]
public enum BuildingTags
{
    None        = 0,
    Base        = 1 << 0,    // 1 - Main base structures
    Defence     = 1 << 1,    // 2 - Defensive structures
    Factory     = 1 << 2,    // 4 - Production facilities
    Storage     = 1 << 3,    // 8 - Resource storage
    Decoration  = 1 << 4,    // 16 - Cosmetic structures
    Power       = 1 << 5,    // 32 - Power generation
    Logistics   = 1 << 6,    // 64 - Transportation
    All         = ~0         // All bits set
}
```

### Using Tags

```csharp
// Assign multiple categories to a building
BuildingTags tags = BuildingTags.Defence | BuildingTags.Base;

// Check if building belongs to a category
bool isDefensive = data.HasCategory(BuildingTags.Defence);

// Enumerate all set flags
foreach (BuildingTags tag in Enum.GetValues(typeof(BuildingTags)))
{
    if ((data.buildingTags & tag) != 0)
    {
        Debug.Log($"Building belongs to category: {tag}");
    }
}
```

### Custom Categories

To add custom categories, extend the enum:

```csharp
[Flags]
public enum BuildingTags
{
    // ... existing categories ...
    
    // Custom categories
    Research   = 1 << 7,    // 128 - Research facilities
    Residential = 1 << 8,   // 256 - Housing
    Medical    = 1 << 9,    // 512 - Healthcare
}
```

**Note**: Always use power-of-two values (1, 2, 4, 8...) to ensure proper bitwise operations.

## BuildingCategory

The [`BuildingCategory`](/broken/pages/6a0b20d7a562fbd5c50b04f2479ceaab2014ede3) asset provides optional UI organization for buildings. Unlike `BuildingTags` which are for gameplay logic, categories are primarily for organizing the building selection UI.

### Properties

| Property       | Type                                                                     | Description                          |
| -------------- | ------------------------------------------------------------------------ | ------------------------------------ |
| `buildingType` | [`BuildingType`](/broken/pages/65498fc919693755d3ddb60c0b18d931fce3cea3) | Type classification                  |
| `CategoryName` | string                                                                   | Display name for the category tab    |
| `buildings`    | `List<BuildingData>`                                                     | Buildings belonging to this category |

### Usage

```csharp
// Create category asset in Editor
// Assets > Grid Building > Building Category

// Assign buildings to the category
category.buildings.AddRange(newBuildingDataList);
```

## GridBuilderDefaults

The [`GridBuilderDefaults`](/broken/pages/b560b41fe777871f498345dde6df670172df1136) asset contains global settings and default prefabs used by editor tools to set up building instances automatically.

### Scene References

| Property             | Type                                                                          | Description                        |
| -------------------- | ----------------------------------------------------------------------------- | ---------------------------------- |
| `buildingManager`    | [`BuildingManager`](/broken/pages/bb7cb781524ae1092e9d2a7efed0d3868a9df706)   | Default building manager reference |
| `defaultGridManager` | [`GridManager`](/broken/pages/52774b93e5740ab56f87249ba2271b72d3bd78f7)       | Default grid manager reference     |
| `economyManager`     | [`EconomyManager`](/broken/pages/66bfda17bb16dd17077a324e070630d24ccf2983)    | Economy system manager             |
| `buildingManagerUI`  | [`BuildingManagerUI`](/broken/pages/2f18329905f1fce6bfbb6170c1425e6e6a309ff6) | UI manager reference               |

### Prefab References

| Property                    | Type       | Description                             |
| --------------------------- | ---------- | --------------------------------------- |
| `buildingSelectablePrefab`  | GameObject | Prefab for clickable building selection |
| `groundPrefab`              | GameObject | Default ground tile prefab              |
| `blockAreaPrefab`           | GameObject | Prefab for blocked area visualization   |
| `defaultConstructionVisual` | Transform  | Visual effect for construction          |

### Layer and Tag Settings

| Property             | Type      | Description                       |
| -------------------- | --------- | --------------------------------- |
| `groundLayer`        | LayerMask | Layer for ground tiles            |
| `blockedLayer`       | LayerMask | Layer for blocked areas           |
| `gridObjectLayer`    | LayerMask | Layer for building objects        |
| `defaultBuildingTag` | string    | Tag applied to building instances |

### API Methods

```csharp
// Get the default selectable prefab
GameObject selectablePrefab = GridBuilderDefaults.Instance.GetBuildingSelectablePrefab();

// Get default construction visual
Transform constructionVisual = GridBuilderDefaults.Instance.GetConstructionVisualPrefab();

// Get default building layer
int buildingLayer = GridBuilderDefaults.Instance.GetDefaultBuildingLayer();
```

### Editor Validation

The defaults asset provides validation tools:

| Button                   | Action                                           |
| ------------------------ | ------------------------------------------------ |
| **Validate All Prefabs** | Checks that all prefabs have required components |
| **Create Missing Tag**   | Checks if the default building tag exists        |

## Data Flow

### Edit Time Flow

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│  BuildingData   │────▶│ BuildingDatabase │     │  GridBuilder    │
│  (Definition)    │     │  SO (Catalog)    │     │  Defaults (CFG) │
└─────────────────┘     └──────────────────┘     └─────────────────┘
         │                       │                        │
         ▼                       ▼                        ▼
    Inspector            Editor Queries            Editor Setup Tools
    Editing              Category Baking            Prefab Assignment
```

### Runtime Flow

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│  BuildingData   │────▶│ BuildingDatabase │     │  BuildingManager│
│  (Loaded Asset) │     │  SO (Cached)     │     │  (Runtime)      │
└─────────────────┘     └──────────────────┘     └─────────────────┘
         │                       │                        │
         ▼                       ▼                        ▼
    Cost Lookup            Query Buildings           Instantiate Prefabs
    Upgrade Check          Category Filter          Apply Settings
```

## Best Practices

### Creating New Buildings

1. **Create BuildingData Asset**: Right-click in Project view, select `Grid Building > Building Data`
2. **Configure Properties**: Set name, tags, type, and assign prefab
3. **Set Build Costs**: Add ResourceAmount entries for required resources
4. **Add to Database**: Use the "Auto-Find All Buildings" button or manually add to BuildingDatabaseSO
5. **Test in Play Mode**: Verify construction, selection, and economy interactions

### Performance Considerations

* **Batch Updates**: When modifying multiple buildings, use the database's sync methods
* **Avoid Runtime Modification**: BuildingData is designed for edit-time configuration
* **Use Cached Lookups**: Access buildings through BuildingDatabaseSO for zero-allocation queries

### Organizing Large Projects

For projects with many buildings:

```
Assets/
└── GridBuilding/
    ├── Data/
    │   ├── BuildingDatabaseSO.asset
    │   ├── GridBuilderDefaults.asset
    │   ├── Categories/
    │   │   ├── Defence/
    │   │   │   └── TowerData.asset
    │   │   └── Production/
    │   │       └── FactoryData.asset
    │   └── Buildings/
    │       ├── BuildingData_01.asset
    │       └── ...
    └── Prefabs/
        └── Buildings/
            ├── Tower_Prefab.prefab
            └── Factory_Prefab.prefab
```

## Related Documentation

| Topic                                                                               | Description                         |
| ----------------------------------------------------------------------------------- | ----------------------------------- |
| [BuildingData](/broken/pages/3b87c48fcaee9051ec83ec6ee70761c11b72c973)              | Detailed BuildingData API reference |
| [BuildingDatabaseSO](/broken/pages/1619048e7d6a9d21f5431ccb024e48cac4fb0c02)        | Database management and queries     |
| [BuildingTags](/broken/pages/48e194224aea0517cf90ffcbddf150230ad974c1)              | Category enumeration reference      |
| [BuildingCategory](/broken/pages/6a0b20d7a562fbd5c50b04f2479ceaab2014ede3)          | UI category organization            |
| [GridBuilderDefaults](/broken/pages/b560b41fe777871f498345dde6df670172df1136)       | Default settings configuration      |
| [Economy System](/broken/pages/66bfda17bb16dd17077a324e070630d24ccf2983)            | Resource and economy integration    |
| [Runtime Building Instance](/broken/pages/abf42fe67b949276f4f158d5a4b5f35c30714482) | Building instance component         |
