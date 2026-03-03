---
name: factory-audit
description: Audit manufacturing stage health AND maintenance document warranty items. ALWAYS activate when the user says "audit factory". Runs alongside supplychain-motivator and google-chat-poster in multi-task prompts.
---

# Factory Audit — Manufacturing Stages & Maintenance Audit

## When to use

Activate when the user says **"audit factory"**. This includes multi-task prompts like:

> "motivate current supply chain, audit factory. Post all results to Google chat"

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

Be **very brief**. One line per flagged stage, nothing else.

- If all stages are healthy: one line — `Stages: all healthy ✓`
- If any are flagged: one line per stage — `⚠ <stage-name> <health>%`

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

Be **very brief**. One line per out-of-warranty item, nothing else.

- If all items are in warranty: one line — `Maintenance: all in warranty ✓`
- If any are out of warranty: one line per item — `⚠ <item-name> expired <date>`

---

## Example output

```
Factory Audit
⚠ assembly-stage 50%
⚠ paint-stage 50%
⚠ Conveyor Belt Motor expired 2024-06-01
⚠ Hydraulic Press Seal expired 2025-01-15
```

## Error handling

- If the factory MCP call fails, report the error and skip Phase 1.
- If the `health` field is missing for a stage, treat it as 0% and flag it.
- If the RAG document is partially readable, process what is available and note any gaps.
