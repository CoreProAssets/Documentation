# icon maker

Defines the quality and location of the generated files.

* **Icon Resolution:** The size of the output texture in pixels (e.g., `256` = 256x256).
* **File Prefix/Suffix:** Added to the prefab name to create the file name.
  * _Example:_ If prefab is `House` and suffix is `_icon`, result is `House_icon.png`.
* **Output Folder:** Path relative to the folder. Leave empty to save in the project root. `Assets`

#### 2. Camera & Look

Controls how the object is rendered.

* **Orthographic Size:** Acts like a "Zoom". Lower value = Closer.
* **Transparent Background:**
  * **Checked:** Saves as a PNG with an alpha channel (perfect for UI).
  * **Unchecked:** Uses the _Camera Background_ color as a solid fill.
* **Alpha Mask:** Optional texture to apply a shape mask (e.g., a circle or rounded square) directly to the icon.

#### 3. Object Placement

Adjusts the position of the spawned model relative to the camera center.

* **Spawn Position/Rotation:** Use this to find the perfect angle (e.g., isometric 45-degree view).
* **Spawn Scale:** Resizes the object if it appears too small or too large in the frame.

#### 4. Preview System

**Critical for setup.** Before generating 100 icons, use this to tweak your settings.

* **Enable Preview:** Turns on the live rendering in the Game View.
* **Preview Prefab:** Drag any object here to see how it looks with current camera/position settings. Changes update in real-time.

#### 5. Prefab Queue (Batch Processing)

The list of objects to render.

* **Drag & Drop:** You can lock the Inspector and drag multiple prefabs from the Project View into this area.
* **Add Selected:** Select prefabs in the Project window and click this button to add them all at once.

***

## Workflow: How to use

Follow these steps to generate your icons:

**1. Setup the Scene** Create a new Scene or use an empty area. Add the `Icon Maker` component to a **Camera**.

**2. Configure the Look**

1. Check **Enable Preview**.
2. Assign one of your buildings/items to the **Preview Prefab** slot.
3. Adjust **Orthographic Size** and **Spawn Rotation** until the object is framed perfectly in the Game View.

**3. Load Objects**

1. Click **Clear All** to remove the test object from the list.
2. Select all the prefabs you want to render in your Project View.
3. Click **Add Selected** in the Inspector. The list will fill up.

**4. Generate** Click the green **Start Icon Generation (Automatic)** button.

* The tool will rapidly spawn, render, and destroy each prefab.
* New icons will appear in your specified **Output Folder**.
