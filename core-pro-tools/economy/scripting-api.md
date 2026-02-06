# Scripting api

#### Basic Operations

```csharp
using CorePro.Economy;

// Check status
int gold = Economy.GetBalance(ResourceId.Gold);
bool isRich = Economy.CanSpend(ResourceId.Gold, 500);

// Modify balance
Economy.Add(ResourceId.Gold, 100);
Economy.Spend(ResourceId.Gold, 50); // Returns bool (false if insufficient funds)

// Capacity
Economy.SetCapacity(ResourceId.Wood, 200);
bool isFull = Economy.IsFull(ResourceId.Wood);
```

#### Transactions (Builder Pattern)

```csharp
var success = EconomyEx.BeginTransaction()
    .Spend(ResourceId.Gold, 100)
    .Spend(ResourceId.Wood, 50)
    .Gain(ResourceId.Score, 10) // Optional: Grant reward on success
    .Commit();

if (success) 
    Debug.Log("Purchase successful!");
else
    Debug.Log("Not enough resources.");
```

#### Events (Reacting to Changes)

```c#
void OnEnable() 
{
    Economy.OnChanged += HandleResourceChange;
}

void HandleResourceChange(ResourceId id, int delta, int total, int lifetime)
{
    if (id == ResourceId.Gold){
        Debug.Log($"Gold changed by {delta}. Current balance: {total}");
}
```

```c#
void OnEnable() 
{
    Economy.OnChanged += HandleResourceChange;
}

void HandleResourceChange(ResourceId id, int delta, int total, int lifetime)
{
    if (id == ResourceId.Gold)
        Debug.Log($"Gold changed by {delta}. Current balance: {total}");
}
```

#### Economy API

<table><thead><tr><th width="375.3111572265625">Method</th><th width="193.199951171875">Return</th><th>Description</th></tr></thead><tbody><tr><td>GetBalance(ResourceId id)</td><td>int</td><td>Returns the current amount of the specified resource available to spend.</td></tr><tr><td>TotalEarned(ResourceId id)</td><td>int</td><td>Returns the total lifetime amount accumulated (ignores spending). Useful for achievements and stats</td></tr><tr><td>CanSpend(ResourceId id, int amount)</td><td>bool</td><td>Checks whether the player has enough of a single resource. Does not modify the balance.</td></tr><tr><td>CanSpend(ResourceAmount[] costs)</td><td>bool</td><td>Checks whether the player can afford multiple resources at once.</td></tr><tr><td>IsReady</td><td>bool</td><td>Returns <code>true</code> if the Economy system has been properly initialized.</td></tr><tr><td>Add(ResourceId id, int amount)</td><td>void</td><td>Adds resources to the balance. Excess over capacity is discarded.</td></tr><tr><td>Spend(ResourceId id, int amount)</td><td>bool</td><td>wdwH9yIfx8tj</td></tr><tr><td>TrySpend(ResourceAmount[] costs)</td><td>bool</td><td>Attempts to spend multiple resources atomically. Resources are deducted only if all costs can be met.</td></tr><tr><td>AddMany(ResourceAmount[] rewards, int n)</td><td>void</td><td>Adds multiple resources at once. Each amount is multiplied by <code>n</code>.</td></tr><tr><td>Set(ResourceId id, int value)</td><td>void</td><td><strong>Debug/Cheat:</strong> Forces a specific balance value, ignoring capacity limits.</td></tr><tr><td>ResetAll()</td><td>void</td><td><strong>Debug:</strong> Resets all balances and lifetime statistics.</td></tr><tr><td>GetCapacity(ResourceId id)</td><td>int</td><td>Returns the storage limit for the resource. Returns <code>0</code> if unlimited.</td></tr><tr><td>SetCapacity(ResourceId id, int cap)</td><td>int</td><td>Sets a new storage limit for the resource.</td></tr><tr><td>IsFull(ResourceId id)</td><td>bool</td><td>Returns <code>true</code> if <code>Balance >= Capacity</code>.</td></tr><tr><td>IsEmpty(ResourceId id)</td><td>bool</td><td>Returns <code>true</code> if the balance is zero or below.</td></tr><tr><td>OnChanged</td><td>id, delta, current, total</td><td>Fired whenever any resource value changes.</td></tr><tr><td>OnEmptied</td><td>id</td><td>Fired when a resource balance reaches zero.</td></tr><tr><td>OnFilled</td><td>id</td><td>Fired when a resource reaches its maximum capacity.</td></tr><tr><td>OnCapacityChanged</td><td>id, oldCap, newCap</td><td>Fired when a resource’s storage limit changes.</td></tr><tr><td>OnAllSyncedEvt</td><td>void</td><td>Fired when the system initializes or loads data. Useful for full UI refresh.</td></tr><tr><td>PauseAllIncomes()</td><td>int</td><td>Pauses all active <code>ResourceIncome</code> components and returns the number paused.</td></tr><tr><td>ResumeAllIncomes()</td><td>int</td><td>Resumes all paused <code>ResourceIncome</code> components.</td></tr><tr><td>PauseAllOutcomes()</td><td>int</td><td>Pauses all active <code>ResourceOutcome</code> (upkeep) components.</td></tr><tr><td>ResumeAllOutcomes()</td><td>int</td><td>Resumes all paused <code>ResourceOutcome</code> components.</td></tr></tbody></table>
