# Building Data

## Introduction

GridBuilderPro uses a comprehensive data architecture based on ScriptableObjects to define buildings, categories, and system defaults. This approach provides several key advantages:

* **Editor Integration**: All data assets appear in the Unity Inspector, enabling convenient editing without custom tools
* **Asset Management**: Data can be version-controlled, shared between projects, and organized in folders
* **Runtime Performance**: Pre-serialized data avoids runtime parsing or calculation overhead
* **Clean Separation**: Data is separate from behavior, following the composition over inheritance principle

The data layer consists of several interconnected asset types that work together to define the building catalog and system configuration.



The `BuildingData` asset is the fundamental definition of a building in the system. Each building in your game should have a corresponding BuildingData asset that defines its properties, costs, and behaviors.

## Building Data Editor

<figure><img src="../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>



### Core Properties

| Property       | Type                                                                         | Description                                        |
| -------------- | ---------------------------------------------------------------------------- | -------------------------------------------------- |
| `buildingTags` | [`BuildingTags`](/broken/pages/515ef7ba6bb084c36601bf5746034efa9ab7d7ed)     | Category flags used for filtering and organization |
| `buildingType` | [`BuildingType`](/broken/pages/7f56da0054c4d6509e4a51dfb927a9fccb1f8b90)     | Functional classification of the building          |
| `buildingName` | string                                                                       | Display name shown in UI                           |
| `prefab`       | [`BuildingInstance`](/broken/pages/86daf2a8cbb74bdc6018279e5bdbed7769e96f11) | Prefab instantiated when building is placed        |
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
| `nextLevelData` | [`BuildingData`](/broken/pages/f96c5196fdb41a2138672584f476e1f6a73cf95c) | Reference to next level's data asset  |

### Connection Properties

| Property         | Type                                                                               | Description                                     |
| ---------------- | ---------------------------------------------------------------------------------- | ----------------------------------------------- |
| `connectionData` | [`BuildingConnectionData`](/broken/pages/deb5ed7755280b13332afeee60fe3450275dd4ce) | Optional configuration for neighbor connections |

### Usage Example

```csharp
// Accessing building data properties
BuildingData data = buildingInstance.GetBuildingData();

// Check if player can afford to build
bool canAfford = data.CanSpendToBuild();

// Get formatted cost description for UI
string costText = data.GetCostDescription();

// Check if building can be upgraded
bool hasUpgrade = data.HasUpgrade();
```

## BuildingDatabaseSO

The [`BuildingDatabaseSO`](/broken/pages/a8668d9b9f0912dc9f50f794ff4a046b6f83b231) asset serves as the central catalog of all available buildings in your project. It provides efficient runtime queries for filtering buildings by category and type.

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

The [`BuildingTags`](/broken/pages/515ef7ba6bb084c36601bf5746034efa9ab7d7ed) enum uses the `[Flags]` attribute to provide a bitwise category system. This allows buildings to belong to multiple categories simultaneously.

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

The [`BuildingCategory`](/broken/pages/8f406731e5ef332b9d6989a584d6039e43d1f923) asset provides optional UI organization for buildings. Unlike `BuildingTags` which are for gameplay logic, categories are primarily for organizing the building selection UI.

### Properties

| Property       | Type                                                                     | Description                          |
| -------------- | ------------------------------------------------------------------------ | ------------------------------------ |
| `buildingType` | [`BuildingType`](/broken/pages/7f56da0054c4d6509e4a51dfb927a9fccb1f8b90) | Type classification                  |
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

The [`GridBuilderDefaults`](/broken/pages/2ce8bc17269d7a9e061f0f118d57455e671d6e47) asset contains global settings and default prefabs used by editor tools to set up building instances automatically.

### Scene References

| Property             | Type                                                                          | Description                        |
| -------------------- | ----------------------------------------------------------------------------- | ---------------------------------- |
| `buildingManager`    | [`BuildingManager`](/broken/pages/8025a526a441d3147f99977c74abb983a49488dd)   | Default building manager reference |
| `defaultGridManager` | [`GridManager`](/broken/pages/9f667de7ecf9c85e0981c1a8a1b787cb4c210cfb)       | Default grid manager reference     |
| `economyManager`     | [`EconomyManager`](/broken/pages/593b1c8a5d6ad9129794a3c0eea6660eea7aea7e)    | Economy system manager             |
| `buildingManagerUI`  | [`BuildingManagerUI`](/broken/pages/125411ec274b1e341c6b9d408c188c61b8f97a4f) | UI manager reference               |

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

