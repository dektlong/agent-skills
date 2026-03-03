---
name: factory-audit
description: Audit manufacturing stage health AND maintenance document warranty items. ALWAYS activate when the user says "audit factory". Runs alongside supplychain-motivator and google-chat-poster in multi-task prompts.
---

# Factory Audit

## When to use

Activate when the user says **"audit factory"**. This includes multi-task prompts like:

> "motivate current supply chain, audit factory. Post all results to Google chat"

## Prerequisites

Use the **factory MCP** server (`tanzu-platform-mcp`). Do **not** use `tanzu-platform-mcp-auth` or any other MCP. If the factory MCP is unavailable, tell the user to set it up first.

## CRITICAL OUTPUT RULE

**The entire factory audit reply MUST fit in 5 lines or fewer. No summaries. No analysis. No recommendations. No tables. No section headers. No explanations. Only the flagged lines below.**

---

## Phase 1: Manufacturing Stages Health

Call `getManufacturingStages` via the **factory MCP**.

For each stage, extract its `health` value. If `health` is missing, treat it as **0%**.

**Output only:**
- Stages with health **< 85%**: one line each → `⚠ <stage-name> <health>%`
- If all healthy: one line → `Stages: all healthy ✓`

**Do NOT output** anything else — no totals, no explanations, no healthy stages.

---

## Phase 2: Maintenance Document — Out-of-Warranty Items

Query the **RAG knowledge base** for an uploaded maintenance document.

- If **no document found**: one line → `Maintenance: no document uploaded`
- If found: identify items where warranty expiry is in the past or status is expired.

**Output only:**
- Out-of-warranty items: one line each → `⚠ <item-name> expired <date>`
- If all in warranty: one line → `Maintenance: all in warranty ✓`

**Do NOT output** anything else — no summaries, no risk assessments, no recommendations, no counts, no tables.

---

## Required output format — nothing more, nothing less

```
Factory Audit
⚠ <stage-name> <health>%
⚠ <item-name> expired <date>
```

## Error handling

- If the factory MCP call fails: one line → `Stages: MCP unavailable`
- If the `health` field is missing for a stage, treat it as 0% and flag it.
