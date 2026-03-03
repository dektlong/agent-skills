---
name: factory-audit
description: Audit manufacturing stage health AND maintenance document warranty items. ALWAYS activate when the user says "audit factory stages". Runs alongside supplychain-motivator and google-chat-poster in multi-task prompts.
---

# Factory Audit — Manufacturing Stages & Maintenance Audit

## When to use

Activate when the user says **"audit factory stages"**. This includes multi-task prompts like:

> "Check manufacturing stages, motivate current supply chain, audit factory stages. Post all results to Google chat"

## Prerequisites

Use the **factory MCP** server (`tanzu-platform-mcp`). Do **not** use `tanzu-platform-mcp-auth` or any other MCP. If the factory MCP is unavailable, tell the user to set it up first.

## Audit Workflow

Run both phases. Each phase produces its own section in the final report.

---

### Phase 1: Manufacturing Stages Health

#### Step 1-1: Fetch all Manufacturing Stages

Call `getManufacturingStages` via the **factory MCP**. This returns all manufacturing stages and their health data in a single call.

If no stages are returned, note "No manufacturing stages found" and skip to Phase 2.

#### Step 1-2: Evaluate health

For each stage returned, extract its `health` value (as a percentage). If `health` is missing, treat it as **0%**.

#### Step 1-3: Report

Report **only stages with health below 85%**. Do not list healthy stages.

- **Header**: `Manufacturing Stages Health`
- **Summary line**: Total stages inspected; count flagged (< 85%).
- If all stages are healthy, state "All stages healthy ✓" and show no further detail.
- For each flagged stage: stage name, health %, `⚠ BELOW THRESHOLD`

---

### Phase 2: Maintenance Document — Out-of-Warranty Items

#### Step 2-1: Retrieve the maintenance document

Query the **RAG knowledge base** for the uploaded maintenance document. Search for content related to "maintenance", "warranty", "equipment", or "service records".

- If **no document is found**, output: `Maintenance Document: No document uploaded — skipping.` and end Phase 2.

#### Step 2-2: Identify out-of-warranty items

From the retrieved document, extract all items that have an explicit warranty expiry date or warranty status. Flag any item where:

- The warranty expiry date is **in the past** (before today's date), **or**
- The warranty status is marked as expired / out-of-warranty.

Ignore items with no warranty information.

#### Step 2-3: Report

Report **only out-of-warranty items**. Do not list items still under warranty.

- **Header**: `Maintenance Document — Out-of-Warranty Items`
- **Summary line**: Total items found in document; count out of warranty.
- If all items are in warranty, state "All items within warranty ✓" and show no further detail.
- For each out-of-warranty item: item name, expiry date (if available), `⚠ OUT OF WARRANTY`

---

## Combined report format

```
Factory Audit
─────────────────────────────────────────
Manufacturing Stages Health
Stages inspected: 6 | Flagged (<85%): 2

⚠ Stages below 85%:
  - assembly-stage  50%  ⚠ BELOW THRESHOLD
  - paint-stage     50%  ⚠ BELOW THRESHOLD

─────────────────────────────────────────
Maintenance Document — Out-of-Warranty Items
Items in document: 8 | Out of warranty: 2

⚠ Out-of-warranty items:
  - Conveyor Belt Motor   expired 2024-06-01  ⚠ OUT OF WARRANTY
  - Hydraulic Press Seal  expired 2025-01-15  ⚠ OUT OF WARRANTY
```

## Error handling

- If the factory MCP call fails, report the error and skip Phase 1.
- If the `health` field is missing for a stage, treat it as 0% and flag it.
- If the RAG document is partially readable, process what is available and note any gaps.
