---
name: factory-audit
description: Audit factory applications for memory compliance in org dekt-org-group (all spaces). Flags apps over 1GB and recommends reducing memory. ALWAYS activate when the user says "audit factory applications", "audit factory apps", or mentions auditing factory apps. Runs alongside supplychain-motivator and google-chat-poster in multi-task prompts.
---

# Factory Audit

## When to use

This skill runs when the user says **"audit factory applications"**, **"audit factory apps"**, or asks to audit factory apps for memory compliance. This includes multi-task prompts like:

> "Check manufacturing stages, motivate current supply chain, audit factory applications, check maintenance document. Post all results to Google chat"

## Purpose

This skill performs a **memory audit** on factory-named apps **in org `dekt-org-group` only**:

1. **Scope**: Organization **`dekt-org-group`** and **all spaces** within that org (no other orgs).
2. **Filter**: Only apps whose name contains `"factory"` (case-insensitive).
3. **Rule**: Flag any app with memory **over 1GB (1024M)** and recommend reducing application memory.

**Out of scope:** Routes, services, non-factory apps, instance counts, staleness. Do not list or analyze apps that do not have "factory" in the name.

## Prerequisites

Verify the Cloud Foundry MCP server is available. If not configured, tell the user to set it up first.

## Audit Workflow

**CRITICAL:** The audit must include **every** app in org `dekt-org-group`. That means querying **every** space in that org (no spaces skipped) and getting the application list for **each** of those spaces. Missing a space = missing apps = incomplete audit.

### Step 1: Enumerate ALL spaces and get ALL apps (do not skip any)

- **Scope is org `dekt-org-group` only.** You must include every space in this org and every app in each of those spaces.
- **Mandatory workflow:**
  1. Call the CF MCP tool to **list spaces** with `organization: "dekt-org-group"`. Note the full list of space names returned.
  2. **For every space** in that list (do not skip any, do not sample), call the CF MCP tool to **list applications** with `organization: "dekt-org-group"` and `space: "<that space name>"`.
  3. Collect apps from all spaces into one combined set. You must have requested the app list for **each** space in dekt-org-group; if you skipped a space, the audit is incomplete.
- **Do not** assume only certain spaces have factory apps; **do not** skip spaces; **do not** use a subset of spaces. Every space in the org must be queried so that all apps in dekt-org-group are considered.

### Step 2: Filter to factory apps only

After retrieving apps for each space:

- **Keep only** apps where the name contains `"factory"` (case-insensitive).
- Discard all other apps; do not list or mention them by name in the report.
- For each factory app, get app details (e.g. `applicationDetails`) including **memory allocation** (in MB). **Always pass** `organization: "dekt-org-group"` and `space: "<the app's space>"` when calling application details so the correct app is retrieved (app names can repeat across spaces).

Examples:
- INCLUDE: "my-factory-app", "Factory-Service", "data-factory-processor"
- EXCLUDE: "web-app", "api-service", "processor"

If no factory-named apps exist in any inspected org/space, report that and end.

### Step 3: Apply memory rule

For each factory app:

- **Compliant**: memory ≤ 1024M (1GB).
- **Over limit**: memory > 1024M → **flag** and add: **Recommendation: Reduce the application memory** (e.g. to 1GB or below as appropriate for the app).

### Step 4: Report

**Report structure:**

1. **Scope**: "Org dekt-org-group, all spaces. Only apps with 'factory' in the name."
2. **Spaces inspected**: List the number and names of spaces in dekt-org-group that were queried (e.g. "Spaces inspected: 5 (space-a, space-b, ...)"). This confirms every space was included.
3. **Summary**: Total factory apps audited; count exceeding 1GB; count compliant (≤1GB).
4. **Apps exceeding 1GB** (if any): For each, list:
   - App name
   - Org / space
   - Allocated memory (e.g. 2048M)
   - **Recommendation: Reduce the application memory** (to 1GB or less).
5. **Compliant factory apps** (optional): Count or short list.

**Verification before responding:**

- [ ] I called the apps list tool **once per space** in dekt-org-group (no spaces skipped).
- [ ] I used explicit `organization: "dekt-org-group"` and `space: "<name>"` on every list/detail call (no default target).
- [ ] Only org `dekt-org-group` and all its spaces were considered.
- [ ] Only factory-named apps were evaluated and listed.
- [ ] Every app over 1GB is named with its actual memory and includes the recommendation to reduce memory.
- [ ] No non-factory apps appear in the report (except possibly a single total count of "X total apps in scope").

## Example output

```
Factory Audit (memory cap 1GB)
Scope: Org dekt-org-group, all spaces. Only apps with 'factory' in the name.

Spaces inspected: 5 (space-a, space-b, space-c, space-d, space-e)

Summary:
- Factory apps audited: 8
- Exceeding 1GB: 2
- Compliant (≤1GB): 6

Apps exceeding 1GB (recommend reducing application memory):
- my-factory-api (dekt-org-group / space-x): 2048M — Recommendation: Reduce the application memory to 1GB or less.
- factory-processor (dekt-org-group / space-y): 1536M — Recommendation: Reduce the application memory to 1GB or less.

Compliant: 6 factory apps at or under 1GB.
```

## Error handling

- If an MCP call fails, report the error clearly.
- If memory is missing for an app, note "memory unknown" for that app and still list it if it might exceed 1GB once known.
