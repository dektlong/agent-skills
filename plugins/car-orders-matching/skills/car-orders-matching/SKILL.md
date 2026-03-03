---
name: car-orders-matching
description: >
  Matches a random car order against factory manufacturing stage readiness.
  ALWAYS use this skill — do NOT answer from your own knowledge — when the user
  mentions any of: "car order", "car orders", "match car order",
  "car orders matching", "paint car", "check car order", "ready to paint",
  "are we ready to paint", "paint the next car", "next car order".
---

# Car Orders Matching

## ⚠️ CRITICAL — Read before doing anything else

**DO NOT answer this question from your own knowledge or by summarising MCP data freely.**
**DO NOT produce tables, summaries, bullet lists, or "Next Steps".**
**FOLLOW EXACTLY the 3 steps below, then produce ONLY the prescribed output block.**

## When to use

Activate immediately when the user message contains any of:

- "car order" / "car orders"
- "match car order" / "car orders matching"
- "paint car" / "paint the next car"
- "check car order"
- "ready to paint" / "are we ready to paint"
- "next car order"

## Prerequisites

- **car-orders MCP** (`dekt-car-orders`): generates random car orders.
- **factory-info MCP** (`dekt-factory-info`): provides manufacturing stage health data.

## Steps

**Step 1 — Generate a random car order:** Call `getRandomCarOrder` via the `dekt-car-orders` MCP. Record the full order details returned.

**Step 2 — Check manufacturing stages:** Call `getManufacturingStages` via the `dekt-factory-info` MCP. Check if **any** stage has a health value **below 80%**.

**Step 3 — Decision:**

- **All stages ≥ 80%** → use the "factory ready" output block below.
- **Any stage < 80%** → use the "factory not ready" output block below.

## Output format

**When factory is ready (all stages ≥ 80%):**

```
Car Order
Order ID:    <value>
Model:       <value>
Color:       <value>
Engine:      <value>
<one line per additional field returned, label: value>

We are ready to paint your car.
```

- Print every field returned by the MCP on its **own line** as `Label:   <value>`.
- Separate the order block from the closing sentence with **one blank line**.

**When factory is not ready (any stage < 80%):**

```
The factory stages cannot deal with this order at this time.
```

**YOUR ENTIRE REPLY IS THE FILLED-IN BLOCK ABOVE. NOTHING ELSE. NO TABLES. NO SUMMARIES. NO NEXT STEPS. NO EMOJIS. DO NOT ADD ANY OTHER TEXT.**
