# Economy manager

This area is used for setting up the component before or during the game.

* **Dont Destroy On Load:** If checked, the Economy Manager persists between scene changes.
*   **Economy Database:** Reference to the .

    * _Inline Editing:_ You can expand this field to view and edit resource definitions (Name, Icon, Initial Capacity) directly within the Manager, without searching for the asset file.

    `EconomyDatabaseSO`
* **Override Init Balances:** A list where you can define specific starting amounts for this scene (e.g., start Level 5 with 1000 Gold), overriding the database defaults.

***

#### 2. Current Resources (Raw State)

Located in the middle, this foldout list shows the **raw data** for every resource registered in the system.

* **Usage:** Expand this list to inspect detailed flags and values for debugging.
* **Data Points:**
  * **Balance / Capacity:** Exact integer values.
  * **Total Earned:** Lifetime accumulation stats.
  * **Is Empty / Is Full:** Read-only checkboxes showing the current logical state of the resource.

***

#### 3. Live Resource Monitor (Bottom Dashboard)

A custom graphical dashboard designed for quick testing during **Play Mode**.

* **Debug Operation Amount:** Controls the value used by the `+` and `-` buttons (default: 10).
* **Resource Cards:** Each active resource is displayed as a panel:
  * **Icon & Name:** Visual identification.
  * **Progress Bar:** Blue bar visualizing `Current Balance / Capacity`. If capacity is unlimited, it shows `∞`.
  * **Quick Actions:** Use the **\[+10]** and **\[-10]** buttons to instantly add or spend resources to test UI reactions or game balance.

\
<br>
