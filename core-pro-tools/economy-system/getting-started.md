---
layout:
  width: wide
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: false
  metadata:
    visible: true
---

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

* In the Project View, right-click and select: \
  `Create -> Core Pro -> Economy -> Resource Definition`.

<div align="left"><figure><img src="../../.gitbook/assets/Economy Create Resource Definition (1).png" alt=""><figcaption></figcaption></figure></div>

***

### Step 3: Fill fields in Resource Definition

<div align="center"><figure><img src="../../.gitbook/assets/Economy Create Resource Definition 2.png" alt=""><figcaption></figcaption></figure></div>

***

### Step 4: Create the Database

* In the Project View, right-click and select: `Create -> Core Pro -> Economy -> Database`.
* In the created file add Resource Definitions

<div align="left"><figure><img src="../../.gitbook/assets/EconomyDatabase Create.png" alt=""><figcaption></figcaption></figure></div>

***

### Step 6: Add Resources Definition to Database

<div align="left"><figure><img src="../../.gitbook/assets/Economy Add Resources to Database.png" alt=""><figcaption></figcaption></figure></div>

### Tip: You may also use Economy Window.

<div align="left"><figure><img src="../../.gitbook/assets/Economy Economy Window 1.png" alt=""><figcaption></figcaption></figure></div>

<div align="left"><figure><img src="../../.gitbook/assets/Economy Window 2.png" alt=""><figcaption></figcaption></figure></div>

***

### Step 7 : Add the Manager to the Scene

1. Create an empty GameObject and name it `[Economy]`.
2. Add the component to it. `EconomyManager`
3. Assign your created **Database** asset to the `Database` field in the Inspector.

Tip: EconomyManager is a singleton with DontDestroyOnLoad enabled. You only need to place it once in your starting scene.

<div align="left"><figure><img src="../../.gitbook/assets/Economy Manager On Scene.png" alt=""><figcaption></figcaption></figure></div>
