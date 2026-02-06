---
description: >-
  The CorePro Economy System is a centralized solution for managing all game
  currencies and resources (e.g., Gold, Wood, Mana, Score). It is designed to be
  lightweight, easy to use, and memory-efficient
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
    visible: true
  metadata:
    visible: true
---

# Economy System

* **Centralized Manager:** A single point of truth for all resource balances.
* **Zero-Allocation:** Built on integer math (`int`) and Enums to avoid garbage collection spikes.
* **Event-Driven:** UI updates automatically only when values change—no polling in `Update()`.
* **Capacity System:** Native support for storage limits (e.g., maximum 100 Wood).
* **Static API:** Access the system from anywhere in your code via `Economy.Add()`.
