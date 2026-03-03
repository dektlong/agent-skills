---
name: factory-audit
description: Audit manufacturing stage health in org dekt-org-group. ALWAYS activate when the user says "audit factory stages". Runs alongside supplychain-motivator and google-chat-poster in multi-task prompts.
---

# Factory Audit — Manufacturing Stages Health Audit

## When to use

Activate when the user says **"audit factory stages"**. This includes multi-task prompts like:

> "Check manufacturing stages, motivate current supply chain, audit factory stages. Post all results to Google chat"

## Purpose

Inspect the **health of every Manufacturing Stage** app in org `dekt-org-group`. A Manufacturing Stage is any app whose name contains `"stage"` (case-insensitive). Health is computed as:

```
health % = (running_instances / desired_instances) × 100
```

Any stage with health **below 85%** is flagged in the summary.

## Prerequisites

Use the **factory MCP** server (`tanzu-platform-mcp`). Do **not** use `tanzu-platform-mcp-auth` or any other MCP. If the factory MCP is unavailable, tell the user to set it up first.

## Audit Workflow

### Step 1: Fetch all Manufacturing Stages

Call `getManufacturingStages` via the **factory MCP**. This returns all manufacturing stages and their health data in a single call.

If no stages are returned, report that and end.

### Step 2: Read health for each stage

For each stage returned, extract its `health` value (as a percentage). If `health` is missing, treat it as **0%**.

### Step 3: Produce summary audit report

1. **Header**: `Manufacturing Stages Health Audit — Org: dekt-org-group`
2. **Summary line**: Total stages found; count healthy (≥ 85%); count flagged (< 85%).
3. **Flagged stages** (health < 85%) — for each:
   - Stage name, running / desired instances, health %, `⚠ BELOW THRESHOLD`
4. **Healthy stages** — names and health % only (compact).

**Verification before responding:**
- [ ] `getManufacturingStages` was called via the factory MCP.
- [ ] Every stage returned was evaluated.
- [ ] Every stage with health < 85% is explicitly flagged.

## Example output

```
Manufacturing Stages Health Audit — Org: dekt-org-group
Stages found: 6 | Healthy (≥85%): 4 | Flagged (<85%): 2

⚠ Flagged Stages (health below 85%):
  - assembly-stage  50%  ⚠ BELOW THRESHOLD
  - paint-stage     50%  ⚠ BELOW THRESHOLD

Healthy Stages:
  - intake-stage  100%
  - weld-stage    100%
  - finish-stage  100%
  - inspect-stage 100%
```

## Error handling

- If an MCP call fails, report the error clearly.
- If the `health` field is missing for a stage, note "health unknown" and flag it as requiring manual review.
