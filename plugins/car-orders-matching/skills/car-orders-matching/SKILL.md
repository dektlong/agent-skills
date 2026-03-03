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

## Color emoji mapping

For **Ext. Color** and **Int. Color**, append a matching color emoji after the value:

| Color word (case-insensitive) | Emoji |
|-------------------------------|-------|
| red / crimson / scarlet       | 🔴    |
| blue / navy / sapphire        | 🔵    |
| green / emerald / forest      | 🟢    |
| yellow / gold / amber         | 🟡    |
| orange                        | 🟠    |
| white / pearl / ivory / cream | ⚪    |
| black / obsidian / onyx       | ⚫    |
| grey / gray / silver          | 🩶    |
| brown / bronze / copper       | 🟤    |
| pink / rose / magenta         | 🩷    |
| purple / violet / lavender    | 🟣    |
| any other color               | 🎨    |

## Output format

**When factory is ready (all stages ≥ 80%):**

Each field MUST be on its own line. Use this exact layout:

```
🚗 Car Order === Car Order #<order-id> ===
Vehicle      : <value>
Ext. Color   : <value> <color-emoji>
Int. Color   : <value> <color-emoji>
Engine       : <value>
Transmission : <value>
Packages     : <value>
Accessories  : <value>
Base Price   : <value>
Total Price  : <value>
Est. Delivery: <value>
Status       : <value>

✅ We are ready to paint your car.
```

- Every field on its **own line** — never concatenate fields onto one line.
- Include any additional fields returned by the MCP in the same `Label : <value>` format.
- One blank line between the last field and the closing sentence.

**When factory is not ready (any stage < 80%):**

```
❌ The factory stages cannot deal with this order at this time.
```

**YOUR ENTIRE REPLY IS THE FILLED-IN BLOCK ABOVE. NOTHING ELSE. NO TABLES. NO SUMMARIES. NO NEXT STEPS. DO NOT ADD ANY OTHER TEXT.**
