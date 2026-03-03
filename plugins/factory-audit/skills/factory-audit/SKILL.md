---
name: factory-audit
description: Audit manufacturing stage health AND maintenance document warranty items. ALWAYS activate when the prompt contains "audit factory", "manufacturing stages", or "inspect maintenance document". Supply chain is NOT part of this skill. Runs alongside supplychain-motivator and google-chat-poster in multi-task prompts.
---

# Factory Audit

## When to use

**ALWAYS activate** when the prompt contains ANY of these keywords:

- `"audit factory"`
- `"manufacturing stages"`
- `"inspect maintenance document"`
- `"maintenance document"`

This includes prompts like:
> "Check manufacturing stages and current supply chain. Inspect maintenance document."

This skill covers **Manufacturing Stages and Maintenance Document only**. The supply chain portion is handled by the supplychain-motivator skill — do not include it here.

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
