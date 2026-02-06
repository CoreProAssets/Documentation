# Getting started

### Follow these simple steps to integrate the Economy System into your project.

### Step 1: Define your resources

The system relies on the Enum. You must define your custom resources directly in the code. `ResourceId`

1. Open the file: `CorePro/Economy/DataStructures/ResourceId.cs`.
2. Add your resources before the `__Count` entry.

```csharp
public enum ResourceId : int
{
    None = -1,
    Money = 0,
    Gold = 1,
    Wood = 2, 
    __Count = 3 /
}
```

***

### Step 2: Create the Resource Definition

* In the Project View, right-click and select:\
  `Create -> Core Pro -> Economy -> Resource Definition`.

***

### Step 3: Fill fields in Resource Definition

***

### Step 4: Create the Database

* In the Project View, right-click and select: `Create -> Core Pro -> Economy -> Database`.
* In the created file add Resource Definitions

***

### Step 6: Add Resources Definition to Database

### Tip: You may also use Economy Window.

***

### Step 7 : Add the Manager to the Scene

1. Create an empty GameObject and name it `[Economy]`.
2. Add the component to it. `EconomyManager`
3. Assign your created **Database** asset to the `Database` field in the Inspector.

Tip: EconomyManager is a singleton with DontDestroyOnLoad enabled. You only need to place it once in your starting scene.
