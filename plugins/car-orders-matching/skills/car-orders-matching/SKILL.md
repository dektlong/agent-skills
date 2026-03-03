---
name: car-orders-matching
description: Matches a random car order against factory manufacturing stage readiness. Use when the user mentions "car order", "match car order", "car orders matching", "paint car", "check car order", or "ready to paint". Generates a random car order via the car-orders MCP and validates factory capacity via the factory-info MCP before accepting the order.
---

# Car Orders Matching

## When to use

Activate when the user mentions any of: **"car order"**, **"car orders"**, **"match car order"**, **"car orders matching"**, **"paint car"**, **"check car order"**, **"ready to paint"**.

## Prerequisites

- **car-orders MCP** (`dekt-car-orders`): generates random car orders.
- **factory-info MCP** (`dekt-factory-info`): provides manufacturing stage health data.

## Steps

**Step 1 — Generate a random car order:** Call `getRandomCarOrder` via the `dekt-car-orders` MCP. Record the full order details returned.

**Step 2 — Check manufacturing stages:** Call `getManufacturingStages` via the `dekt-factory-info` MCP. Check if **any** stage has a health value **below 80%**.

**Step 3 — Decision:**

- **All stages ≥ 80%** → Output the car order details followed by: _"We are ready to paint your car."_
- **Any stage < 80%** → Output: _"The factory stages cannot deal with this order at this time."_

## Output format

**When factory is ready (all stages ≥ 80%):**

```
Car Order
<full car order details from Step 1>

We are ready to paint your car.
```

**When factory is not ready (any stage < 80%):**

```
The factory stages cannot deal with this order at this time.
```

**YOUR ENTIRE REPLY IS THE FILLED-IN BLOCK ABOVE. NOTHING ELSE. DO NOT ADD ANY OTHER TEXT.**
