# prefs editor pro

`Tools -> CorePro -> Prefs Editor Pro`

### 1. View Modes (Tabs)

The window is divided into three tabs to separate different types of data:

* **PlayerPrefs:** Shows data saved by your game (e.g., Score, Settings, Save Slots).
* **EditorPrefs:** Shows settings saved by the Unity Editor itself (e.g., Layouts, Last Opened Folder).
* **EditorCoreProPrefs:** A specialized filter that only shows Editor keys starting with "CorePro". Useful for debugging your own tool settings.

### 2. Search & Filter

Managing hundreds of keys can be messy. Use the toolbar to find exactly what you need.

* **Search Bar:** Filters the list by Key name in real-time.
* **Hide System Prefs:** When enabled, it hides internal Unity keys (garbage) to keep the list clean.
* **Exclusion List:** You can customize which keys to hide. Default is `unity, com.unity`.

### 3. Editing Values

You don't need to delete and recreate keys to change values.

* **Direct Editing:** Simply click on any value field (String, Int, or Float) in the list and type a new number or text.
* **Type Safety:** The editor automatically detects if a value is an Integer, Float, or String.

### 4. Managing Keys

* **Add New:** Opens a popup to create a custom Key/Value pair manually.
* **Delete Selected:** You can select multiple rows (using `Shift+Click` or `Ctrl/Cmd+Click`) and delete them all at once.
* **Delete All (Visible):** Deletes **only** the items currently visible in the list.
  * _Safety Feature:_ If you use the Search bar to filter for "Score", clicking "Delete All" will ONLY delete keys containing "Score", leaving everything else safe.

***

## Workflow: Debugging a Save System

**Scenario:** You want to reset the player's gold to test the "Not Enough Money" UI.

1. Open **Prefs Editor Pro**.
2. Select the **PlayerPrefs** tab.
3. Type `Gold` in the **Search** bar.
4. Locate the key (e.g., `Player_Gold_Amount`).
5. Change the value from `1000` to `0`.
6. The change is saved instantly. Play your game to test.

Note: This tool reads directly from the System Registry (on Windows). If your game modifies a value while this window is open, click Refresh to see the latest data.
