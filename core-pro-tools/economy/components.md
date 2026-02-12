# Components

{% hint style="danger" %}
Page under construction
{% endhint %}

#### Resource Counter UI

Displays the current amount of a specific resource on the screen.

* **Usage:** Attach this to a GameObject with a TextMeshPro component.
* **Setup:** Select the `Resource Id` you want to track and assign the text references.
* **Behavior:** Automatically listens for changes. You do not need to write any code.

#### Resource Income (Generator)

Generates resources over time (e.g., a Gold Mine).

* **Usage:** Add to any object that should produce resources.
* **Logic:** Adds `Amount` every seconds. `Interval`
* **Constraints:** If the resource storage is full (Capacity reached), production stops automatically.
* **Control:** Can be paused/resumed via Inspector or code (e.g., if a building is powered down).

#### Resource Outcome (Upkeep)

Deducts resources over time (e.g., Army salary or Electricity bill).

* **Usage:** Add to any object that consumes resources.
* **Logic:** Spends `Amount` every seconds. `Interval`
* **Pause on Failure:** If the player cannot afford the cost, the component can automatically pause itself and trigger a failure event (e.g., disable the building).

#### Resource Storage

Increases the maximum capacity for a resource.

* **Usage:** Add to storage buildings (e.g., a Warehouse).
* **Logic:** When enabled, it increases the global `Capacity` for the specified resource.
* **Example:** A "Silo" object adds +1000 to the Grain capacity.
