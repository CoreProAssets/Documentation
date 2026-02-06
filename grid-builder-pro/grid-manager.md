# Grid manager

## Overview

The `GridManager` class serves as the central authority for all grid-based operations in GridBuilderPro. As a Singleton pattern implementation, it provides global access to grid functionality throughout the game. This class handles grid configuration, coordinate transformation, occupancy management, placement validation, and grid visualization.

The GridManager is marked with `[DefaultExecutionOrder(-100)]` to ensure it initializes before other systems that depend on grid functionality.

## Class Definition

```csharp
namespace CorePro.GridSystem
{
    [DefaultExecutionOrder(-100)]
    public class GridManager : SingletonCorePro<GridManager>
    {
        // Properties and methods
    }
}
```

## Singleton Access

GridManager follows the Singleton pattern, providing global access through a static instance property:

```csharp
// Recommended access method
GridManager gridManager = GridManager.Instance;

// Fallback with null-coalescing
GridManager gridManager = GridManager.Instance ?? FindObjectOfType<GridManager>();
```

{% hint style="warning" %}
**Important**: The GridManager uses `SingletonCorePro<T>` which provides automatic initialization and cleanup. In edit mode, use `ResetSingleton()` to clear stale instances.
{% endhint %}

## Configuration Properties

### Grid Dimensions

| Property           | Type         | Description                                                  |
| ------------------ | ------------ | ------------------------------------------------------------ |
| `cellSize2D`       | `int`        | Size of each grid cell in XZ plane (world units)             |
| `cellHeightSize`   | `float`      | Height of each grid cell in Y axis. If <= 0, uses cellSize2D |
| `gridDimensions3D` | `Vector3Int` | Grid dimensions: (Width X, Height Y levels, Depth Z)         |

```csharp
// Example: Configure grid dimensions
gridManager.gridDimensions3D = new Vector3Int(50, 1, 50);  // 50x50 grid with 1 level
gridManager.cellSize2D = 2;  // Each cell is 2x2 world units
```

### Chunk Configuration

| Property      | Type  | Description                                             |
| ------------- | ----- | ------------------------------------------------------- |
| `chunkSizeXZ` | `int` | Chunk size for XZ plane (larger for horizontal spread)  |
| `chunkSizeY`  | `int` | Chunk size for Y axis (smaller - fewer vertical levels) |

The chunk system divides the grid into smaller regions for improved memory locality and query performance:

```
Grid (50x50) with chunkSizeXZ=8:
├── Chunk [0,0] (8x8 cells)
├── Chunk [1,0] (8x8 cells)
└── ... more chunks
```

### Layer Configuration

| Property            | Type        | Description                                         |
| ------------------- | ----------- | --------------------------------------------------- |
| `groundLayerMask`   | `LayerMask` | Layers considered as valid ground for baking        |
| `obstacleLayerMask` | `LayerMask` | Layers considered as obstacles that block placement |

```csharp
// Example: Configure layers
gridManager.groundLayerMask = LayerMask.GetMask("Ground", "Terrain");
gridManager.obstacleLayerMask = LayerMask.GetMask("Obstacle", "Wall");
```

### Terrain Adaptation

| Property                 | Type    | Description                                      |
| ------------------------ | ------- | ------------------------------------------------ |
| `allowTerrainAdaptation` | `bool`  | Allows building on uneven terrain (rocks, hills) |
| `maxGroundCheckDistance` | `float` | Maximum distance for ground raycasts             |
| `samplerOffsetY`         | `float` | Vertical offset for terrain sampling             |

## Reference Properties

| Property         | Type                 | Description                                               |
| ---------------- | -------------------- | --------------------------------------------------------- |
| `mainCamera`     | `Camera`             | Reference to the main camera (auto-assigned if null)      |
| `gridVisualizer` | `GridVisualizerBase` | Component for grid mesh visualization                     |
| `gridHighlight`  | `GridHighlightBase`  | Component for cell highlighting                           |
| `gridSelect`     | `GridSelectBase`     | Component for selection visualization                     |
| `bakedGridData`  | `BakedGridData`      | ScriptableObject with pre-computed collision/terrain data |

## Grid Properties

### Dimensions and Size

| Property              | Type         | Description                                                            |
| --------------------- | ------------ | ---------------------------------------------------------------------- |
| `CellSize2D`          | `float`      | Read-only cell size in XZ plane                                        |
| `CellHeightSize`      | `float`      | Read-only cell height (returns cellHeightSize if > 0, else cellSize2D) |
| `GridSize3D`          | `Vector3Int` | Read-only 3D footprint for a single cell                               |
| `GridSize2D`          | `Vector2Int` | Read-only 2D footprint for a single cell                               |
| `GridDimensions3D`    | `Vector3Int` | Read-only total grid dimensions                                        |
| `GroundLayerMask`     | `LayerMask`  | Read-only ground layer mask                                            |
| `AlwaysHighlightCell` | `bool`       | Whether to always highlight hovered cell                               |

```csharp
// Read grid properties
float cellArea = gridManager.CellSize2D * gridManager.CellSize2D;
Vector3Int totalCells = gridManager.GridDimensions3D;
int totalCellCount = totalCells.x * totalCells.y * totalCells.z;
```

## Coordinate Transformation Methods

### World to Grid

```csharp
/// <summary>
/// Converts a world space position into integer grid coordinates.
/// </summary>
/// <param name="worldPos">World position to convert.</param>
/// <returns>Grid cell coordinates.</returns>
public Vector3Int GetCell(Vector3 worldPos)
```

**Example Usage:**

```csharp
Vector3 playerPosition = new Vector3(15.5f, 0, 22.3f);
Vector3Int cell = gridManager.GetCell(playerPosition);
// Returns: (7, 0, 11) assuming cellSize2D=2
```

### Grid to World

```csharp
/// <summary>
/// Converts integer grid coordinates back into a world space position (pivot/bottom-left).
/// </summary>
/// <param name="cell">Grid cell coordinates.</param>
/// <returns>World position of the cell's origin (bottom-left).</returns>
public Vector3 CellToWorld(Vector3Int cell)
```

**Example Usage:**

```csharp
Vector3Int cell = new Vector3Int(5, 0, 3);
Vector3 worldPos = gridManager.CellToWorld(cell);
// Returns: (10, 0, 6) assuming cellSize2D=2
```

### Cell Center

```csharp
/// <summary>
/// Returns the center position of a cell in world space.
/// Useful for spawning objects at the center of a cell.
/// </summary>
/// <param name="cell">Cell coordinates (can be Vector3).</param>
/// <returns>World position of cell center.</returns>
public Vector3 GetCellCenter2D(Vector3 cell)
```

**Example Usage:**

```csharp
Vector3Int cell = new Vector3Int(5, 0, 3);
Vector3 center = gridManager.GetCellCenter2D(cell);
// Returns: (11, 0, 7) for cellSize2D=2 (center = origin + half cell)
```

## Occupancy Queries

### Basic Queries

```csharp
/// <summary>
/// Checks if a specific cell coordinate is marked as blocked (baked or runtime).
/// </summary>
/// <param name="cellPos">Cell coordinates to check.</param>
/// <returns>True if cell is blocked, false if free.</returns>
public bool IsCellBlocked(Vector3Int cellPos)
```

### 2D Area Queries

```csharp
/// <summary>
/// Returns true if the specified 2D area (footprint) is not blocked.
/// Use this for most buildings and ground checks.
/// </summary>
/// <param name="coordsXZ">Base coordinates (X, Z).</param>
/// <param name="sizeXZ">Size of the area (width, depth).</param>
/// <param name="yLevel">Y level to check (default: 0).</param>
/// <returns>True if all cells in the area are free.</returns>
public bool IsAreaFree2D(Vector2Int coordsXZ, Vector2Int sizeXZ, int yLevel = 0)
```

**Example Usage:**

```csharp
// Check if 3x2 area starting at (5, 3) is free
Vector2Int baseCoords = new Vector2Int(5, 3);
Vector2Int size = new Vector2Int(3, 2);
bool isFree = gridManager.IsAreaFree2D(baseCoords, size);
```

### 3D Area Queries

```csharp
/// <summary>
/// Returns true if the entire 3D space is free.
/// Use this for flying objects or multi-level structures.
/// </summary>
/// <param name="baseCoords">Base cell coordinates.</param>
/// <param name="size">Size of the volume (width, height, depth).</param>
/// <returns>True if all cells in the volume are free.</returns>
public bool IsAreaFree3D(Vector3Int baseCoords, Vector3Int size)
```

**Example Usage:**

```csharp
// Check if 2x3x2 volume is free (useful for multi-level buildings)
Vector3Int baseCell = new Vector3Int(5, 0, 3);
Vector3Int size = new Vector3Int(2, 3, 2);  // 2 wide, 3 levels high, 2 deep
bool isFree = gridManager.IsAreaFree3D(baseCell, size);
```

### Single Cell Check

```csharp
/// <summary>
/// Shortcut for the most common 1x1 cell check.
/// </summary>
/// <param name="cell">Cell coordinates to check.</param>
/// <returns>True if cell is free.</returns>
public bool IsCellFree(Vector3Int cell)
```

**Example Usage:**

```csharp
bool isFree = gridManager.IsCellFree(new Vector3Int(5, 0, 3));
```

### Detailed Check

```csharp
/// <summary>
/// Diagnostic version - returns detailed info WHY the cube is not free.
/// Use for debugging placement issues.
/// </summary>
/// <param name="baseCell">Base cell coordinates.</param>
/// <param name="size">Size of the area to check.</param>
/// <returns>Detailed result including reason for failure.</returns>
public AreaFreeCheckResult IsAreaFreeDetailed(Vector3Int baseCell, Vector3Int size)
```

**AreaFreeCheckResult Structure:**

```csharp
public struct AreaFreeCheckResult
{
    public bool IsFree;           // Whether area is free
    public string Reason;          // Detailed reason for failure
    public bool IsOutOfBounds;    // Whether check failed due to out-of-bounds
    public Vector3Int FirstBlockedCell;  // First blocked cell found
}
```

### Terrain Flatness Check

```csharp
/// <summary>
/// Checks if all cells in the specified 3D area have the same terrain height.
/// This prevents placing buildings on uneven terrain (e.g., staircases).
/// Optimization: This check is skipped for 1x1 objects.
/// </summary>
/// <param name="baseCell">The base cell coordinate (bottom-left-front).</param>
/// <param name="size">The size of the area to check (width, height, depth).</param>
/// <returns>True if all cells have the same terrain height, false otherwise.</returns>
public bool IsAreaFlat3D(Vector3Int baseCell, Vector3Int size)
```

## Placement Methods

### Place Object

```csharp
/// <summary>
/// Registers an object on the grid and logically blocks the occupied cells.
/// Accounts for 'BlockCellsAbove' to prevent stacking on certain buildings.
/// </summary>
/// <param name="obj">The ICellObject to place.</param>
/// <param name="baseCell">The base cell coordinates where the object will be placed.</param>
public void PlaceObject(ICellObject obj, Vector3Int baseCell)
```

**Example Usage:**

```csharp
gridManager.PlaceObject(buildingInstance, new Vector3Int(5, 0, 3));
```

**Notes:**

* Validates placement before registering
* Handles IBlockCellsAbove interface automatically
* Throws error if placement is invalid

### Remove Object

```csharp
/// <summary>
/// Unregisters an object and clears its blocked cells.
/// Also handles clearing 'BlockedCellsAbove' if they were occupied.
/// </summary>
/// <param name="cellObject">The ICellObject to remove.</param>
public void RemoveObject(ICellObject cellObject)
```

**Example Usage:**

```csharp
gridManager.RemoveObject(buildingInstance);
```

### Blocking Operations

```csharp
/// <summary>
/// Marks a 3D volume as blocked. Use for temporary obstacles or dynamic logic.
/// </summary>
public void BlockArea3D(Vector3Int baseCell, Vector3Int size)

/// <summary>
/// Marks a 3D volume as free.
/// </summary>
public void UnblockArea3D(Vector3Int baseCell, Vector3Int size)
```

## Occupancy Queries for Objects

### Get Object at Cell

```csharp
/// <summary>
/// Returns the object occupying a specific 3D cell coordinate.
/// </summary>
/// <param name="cell">The cell coordinate to check.</param>
/// <returns>The ICellObject or null if the cell is free.</returns>
public ICellObject GetObjectAt(Vector3Int cell)

/// <summary>
/// Returns the object occupying a specific 3D cell coordinate, cast to a specific type.
/// </summary>
/// <typeparam name="T">The expected type of object.</typeparam>
/// <param name="cell">The cell coordinate to check.</param>
/// <returns>The object as type T, or null if the cell is empty or type is wrong.</returns>
public T GetObjectAt<T>(Vector3Int cell) where T : class, ICellObject
```

### Get Cells Occupied by Object

```csharp
/// <summary>
/// Returns a collection of all 3D cell coordinates occupied by a specific object.
/// </summary>
/// <param name="obj">The object to query.</param>
/// <returns>Read-only collection of cell coordinates occupied by the object.</returns>
public IReadOnlyCollection<Vector3Int> GetCellsOccupiedBy(ICellObject obj)
```

### Try Get Object

```csharp
/// <summary>
/// Attempts to find a registered object at a world position.
/// </summary>
/// <typeparam name="T">The expected type of object.</typeparam>
/// <param name="worldPos">World position to check.</param>
/// <param name="result">The found object, or null if not found.</param>
/// <returns>True if an object was found.</returns>
public bool TryGetObjectAt<T>(Vector3 worldPos, out T result) where T : class, ICellObject
```

## Grid Data Access

```csharp
/// <summary>
/// Gets the underlying IGridData interface for advanced queries.
/// </summary>
/// <returns>The grid data interface.</returns>
public IGridData GetGridData()
```

## Terrain Snapping

```csharp
/// <summary>
/// Sophisticated snapping that accounts for terrain height.
/// Uses O(1) dictionary lookups for performance.
/// It checks runtime object height (if stacked) vs baked terrain height and snaps to the highest point.
/// </summary>
/// <param name="worldPos">World position to snap.</param>
/// <returns>Snapped world position accounting for terrain and stacked objects.</returns>
public Vector3 SnapToGridTerrain(Vector3 worldPos)
```

**Example Usage:**

```csharp
// Snap to grid while respecting terrain height
Vector3 mousePos = new Vector3(15.5f, 10f, 22.3f);
Vector3 snappedPos = gridManager.SnapToGridTerrain(mousePos);
// Result: X,Z snapped to grid, Y adjusted to terrain height
```

## Cell Data Queries

```csharp
/// <summary>
/// Returns the CellData for a specific object registered on the grid.
/// </summary>
/// <param name="cellObject">The object to get data for.</param>
/// <returns>CellData containing position and state information.</returns>
public CellData GetCellData(ICellObject cellObject)
```

### Get Cell Data at World Position

```csharp
/// <summary>
/// Returns a complete packet of info (CellData) for a point in world space.
/// Includes snapped position, whether it is free, and what object occupies it.
/// </summary>
/// <param name="worldPos">World position to query.</param>
/// <returns>CellData with all relevant information.</returns>
public CellData GetCellDataAtWorldPos(Vector3 worldPos)
```

**CellData Structure:**

```csharp
public struct CellData
{
    public Vector3Int Cell;          // Grid cell coordinates
    public Vector3 WorldPos;          // World position (snapped if terrain adaptation enabled)
    public Vector3 Center;            // Center of cell in world space
    public bool IsFree;               // Whether cell is free
    public bool HasOccupant;          // Whether cell has an object
    public ICellObject Occupant;      // Reference to occupying object
    public bool IsOnGrid;            // Whether position is within grid bounds
    public bool IsValid;              // Whether data is valid
}
```

## Grid Mask

```csharp
/// <summary>
/// Checks if a coordinate is excluded from the grid based on the assigned Texture2D mask.
/// </summary>
/// <param name="cell">Cell coordinates to check.</param>
/// <returns>True if the cell is masked (excluded from grid).</returns>
public bool IsMasked(Vector2Int cell)

/// <summary>
/// Sets a new grid mask texture.
/// </summary>
/// <param name="newMask">Texture2D mask where white = valid, black = invalid.</param>
public void SetGridMask(Texture2D newMask)

/// <summary>
/// Gets the current grid mask texture.
/// </summary>
/// <returns>The current grid mask, or null if none set.</returns>
public Texture2D GetGridMask()
```

## Visualization Methods

```csharp
/// <summary>
/// Forces the visualizer component to redraw the grid mesh.
/// </summary>
public void RefreshGridVisualization()

/// <summary>
/// Shows the grid visualization.
/// </summary>
public void ShowGrid()

/// <summary>
/// Hides the grid visualization and cell highlighting.
/// </summary>
public void HideGrid()
```

## Highlight Methods

### Single Cell Highlight

```csharp
/// <summary>
/// Advanced highlight that handles both ground visual (Decal/Projector) and object-specific feedback.
/// </summary>
/// <param name="cellData">Cell data for the highlight position.</param>
/// <param name="footprint">Footprint size for the highlight.</param>
/// <param name="isValid">Whether the placement is valid (affects color).</param>
public void HighlightSingle(in CellData cellData, Vector3Int footprint, bool isValid)
```

### Area Highlight

```csharp
/// <summary>
/// Highlights any rectangular area on the grid (e.g., during Drag Select or aiming an AOE skill).
/// </summary>
/// <param name="gridArea">Area to highlight (in grid coordinates).</param>
/// <param name="isValid">Whether the area is valid (affects color).</param>
public void HighlightArea(BoundsInt gridArea, bool isValid)
```

### Clear Highlights

```csharp
/// <summary>
/// Clears all visual highlights from the grid and current occupants.
/// </summary>
public void HideHighlight()
```

## Selection Methods

### Single Selection

```csharp
/// <summary>
/// Selects a single cell and any object on it.
/// </summary>
/// <param name="data">Cell data for the selection.</param>
/// <param name="addToExisting">Whether to add to existing selection.</param>
public void SelectSingle(in CellData data, bool addToExisting = false)
```

### Area Selection

```csharp
/// <summary>
/// Selects all objects and cells in a specific grid area.
/// </summary>
/// <param name="gridArea">Area to select.</param>
/// <param name="addToExisting">Whether to add to existing selection.</param>
public void SelectArea(BoundsInt gridArea, bool addToExisting = false)
```

### Clear Selection

```csharp
/// <summary>
/// Clears all selections and hides the 3D visualizer.
/// </summary>
public void ClearSelection()

/// <summary>
/// Clears the entire selection (alternative method).
/// </summary>
public void HideSelection()
```

## Events

```csharp
/// <summary>
/// Action triggered whenever the occupancy of the grid changes (placement/removal).
/// </summary>
public event Action<Bounds> OnCellsChanged;

/// <summary>
/// Action triggered when the selection changes.
/// </summary>
public event Action<HashSet<ICellObject>> OnSelectionChanged;
```

**Example Usage:**

```csharp
// Subscribe to grid changes
GridManager.Instance.OnCellsChanged += region => {
    Debug.Log($"Grid changed in area: {region}");
};

// Subscribe to selection changes
GridManager.Instance.OnSelectionChanged += selectedObjects => {
    Debug.Log($"Selected {selectedObjects.Count} objects");
};
```

## Editor Methods

### Grid Baking

```csharp
/// <summary>
/// EDITOR ONLY: Scans the scene using Physics Raycasts to find the ground and obstacles.
/// This creates/updates a BakedGridData asset.
/// High performance optimization: this should be run after changing the level geometry.
/// </summary>
[Button("Bake Grid from Scene")]
public void BakeGridFromScene()
```

**Usage in Editor:**

1. Configure ground and obstacle layers
2. Set grid dimensions and cell size
3. Click "Bake Grid from Scene" in the Inspector
4. BakedGridData asset is created/updated automatically

### Layer Configuration

```csharp
/// <summary>
/// Draws a row in the setup window for layer configuration.
/// </summary>
private void DrawLayerConfigRow(string propertyName, string label, int REC_GRID)
```

## Performance Characteristics

| Operation        | Time Complexity | Notes              |
| ---------------- | --------------- | ------------------ |
| `GetCell()`      | O(1)            | Direct calculation |
| `CellToWorld()`  | O(1)            | Direct calculation |
| `IsCellFree()`   | O(1)            | Hash lookup        |
| `IsAreaFree2D()` | O(n)            | n = area cells     |
| `IsAreaFree3D()` | O(n)            | n = volume cells   |
| `GetObjectAt()`  | O(1)            | Hash lookup        |
| `PlaceObject()`  | O(1)            | Hash insertion     |
| `RemoveObject()` | O(1)            | Hash removal       |

## Common Usage Patterns

### Validating Building Placement

```csharp
public bool CanPlaceBuilding(BuildingInstance prefab, Vector3Int baseCell)
{
    Vector3Int footprint = prefab.GetFootprint3D();
    
    // Check bounds
    if (baseCell.x < 0 || baseCell.z < 0) return false;
    if (baseCell.x + footprint.x > gridManager.GridDimensions3D.x) return false;
    if (baseCell.z + footprint.z > gridManager.GridDimensions3D.z) return false;
    
    // Check occupancy
    return gridManager.IsAreaFree3D(baseCell, footprint);
}
```

### Getting All Buildings in Area

```csharp
public List<BuildingInstance> GetBuildingsInArea(BoundsInt area)
{
    List<BuildingInstance> buildings = new List<BuildingInstance>();
    
    for (int x = area.xMin; x < area.xMax; x++)
    {
        for (int z = area.zMin; z < area.zMax; z++)
        {
            Vector3Int cell = new Vector3Int(x, 0, z);
            var building = gridManager.GetObjectAt<BuildingInstance>(cell);
            
            if (building != null && !buildings.Contains(building))
            {
                buildings.Add(building);
            }
        }
    }
    
    return buildings;
}
```

### Converting World to Grid and Back

```csharp
// World → Grid → World
Vector3 worldPos = new Vector3(15.5f, 0, 22.3f);
Vector3Int cell = gridManager.GetCell(worldPos);
Vector3 snappedWorldPos = gridManager.CellToWorld(cell);
// snappedWorldPos ≈ worldPos (snapped to grid)
```

