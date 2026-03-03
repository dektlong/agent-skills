---
name: factory-audit
description: Audits manufacturing stage health and maintenance document warranty items. Use when the user mentions "manufacturing stages", "inspect maintenance document", "maintenance document", or "audit factory". Supply chain is out of scope for this skill.
---

# Factory Audit

## When to use

Activate when the user mentions any of: **"manufacturing stages"**, **"maintenance document"**, **"inspect maintenance document"**, **"audit factory"**.

This skill covers **Manufacturing Stages and Maintenance Document only**. Supply chain is handled by the supplychain-motivator skill — do not include it here.

## Prerequisites

Use the **factory MCP** (`dekt-factory-info`) only. Do not use any other MCP.

## Steps

**Step 1 — Manufacturing Stages:** Call `getManufacturingStages` via the factory MCP. From the result, collect only stage names where `health < 85%`.

**Step 2 — Maintenance Document:** Query the RAG knowledge base. From the result, collect only the `(item name, warranty expiry date)` pairs where the expiry date is before today. Ignore everything else in the document — do not read summaries, recommendations, risk sections, or any other content.

## Output — copy this block verbatim, filling in only the flagged lines

```
Factory Audit
⚠ <stage-name> <health>%
⚠ <item-name> expired <date>
```

- One `⚠` line per flagged stage. If none: `Stages: all healthy ✓`
- One `⚠` line per expired item. If no document: `Maintenance: no document`. If none expired: `Maintenance: all in warranty ✓`

**YOUR ENTIRE REPLY IS THE FILLED-IN BLOCK ABOVE. NOTHING ELSE. DO NOT ADD ANY OTHER TEXT.**
