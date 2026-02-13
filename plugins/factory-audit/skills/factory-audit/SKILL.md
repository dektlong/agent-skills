---
name: factory-audit
description: Finds Cloud Foundry apps with "factory" in the name in org dekt-group-org (all spaces), flags apps whose memory exceeds 1GB, and recommends reducing application memory. Use when the user says "audit factory apps", asks to audit factory apps memory, find factory apps over 1GB, or run a factory memory audit.
---

# Factory Audit

## When to use

This skill runs when the user says **"audit factory apps"** or asks to audit factory apps (memory), find factory apps over 1GB, or check factory app memory in dekt-group-org.

## Purpose

This skill performs a **memory audit** on factory-named apps **in org `dekt-group-org` only**:

1. **Scope**: Organization **`dekt-group-org`** and **all spaces** within that org (no other orgs).
2. **Filter**: Only apps whose name contains `"factory"` (case-insensitive).
3. **Rule**: Flag any app with memory **over 1GB (1024M)** and recommend reducing application memory.

**Out of scope:** Routes, services, non-factory apps, instance counts, staleness. Do not list or analyze apps that do not have "factory" in the name.

## Prerequisites

Verify the Cloud Foundry MCP server is available. If not configured, tell the user to set it up first.

## Audit Workflow

**CRITICAL:** The audit must include **every** app in org `dekt-group-org`. That means querying **every** space in that org (no spaces skipped) and getting the application list for **each** of those spaces. Missing a space = missing apps = incomplete audit.

### Step 1: Enumerate ALL spaces and get ALL apps (do not skip any)

- **Scope is org `dekt-group-org` only.** You must include every space in this org and every app in each of those spaces.
- **Mandatory workflow:**
  1. Call the CF MCP tool to **list spaces** with `organization: "dekt-group-org"`. Note the full list of space names returned.
  2. **For every space** in that list (do not skip any, do not sample), call the CF MCP tool to **list applications** with `organization: "dekt-group-org"` and `space: "<that space name>"`.
  3. Collect apps from all spaces into one combined set. You must have requested the app list for **each** space in dekt-group-org; if you skipped a space, the audit is incomplete.
- **Do not** assume only certain spaces have factory apps; **do not** skip spaces; **do not** use a subset of spaces. Every space in the org must be queried so that all apps in dekt-group-org are considered.

### Step 2: Filter to factory apps only

After retrieving apps for each space:

- **Keep only** apps where the name contains `"factory"` (case-insensitive).
- Discard all other apps; do not list or mention them by name in the report.
- For each factory app, get app details (e.g. `applicationDetails`) including **memory allocation** (in MB). **Always pass** `organization: "dekt-group-org"` and `space: "<the app's space>"` when calling application details so the correct app is retrieved (app names can repeat across spaces).

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

1. **Scope**: "Org dekt-group-org, all spaces. Only apps with 'factory' in the name."
2. **Spaces inspected**: List the number and names of spaces in dekt-group-org that were queried (e.g. "Spaces inspected: 5 (space-a, space-b, ...)"). This confirms every space was included.
3. **Summary**: Total factory apps audited; count exceeding 1GB; count compliant (≤1GB).
4. **Apps exceeding 1GB** (if any): For each, list:
   - App name
   - Org / space
   - Allocated memory (e.g. 2048M)
   - **Recommendation: Reduce the application memory** (to 1GB or less).
5. **Compliant factory apps** (optional): Count or short list.

**Verification before responding:**

- [ ] I called the apps list tool **once per space** in dekt-group-org (no spaces skipped).
- [ ] I used explicit `organization: "dekt-group-org"` and `space: "<name>"` on every list/detail call (no default target).
- [ ] Only org `dekt-group-org` and all its spaces were considered.
- [ ] Only factory-named apps were evaluated and listed.
- [ ] Every app over 1GB is named with its actual memory and includes the recommendation to reduce memory.
- [ ] No non-factory apps appear in the report (except possibly a single total count of "X total apps in scope").

## Example output

```
Factory Audit (memory cap 1GB)
Scope: Org dekt-group-org, all spaces. Only apps with 'factory' in the name.

Spaces inspected: 5 (space-a, space-b, space-c, space-d, space-e)

Summary:
- Factory apps audited: 8
- Exceeding 1GB: 2
- Compliant (≤1GB): 6

Apps exceeding 1GB (recommend reducing application memory):
- my-factory-api (dekt-group-org / space-x): 2048M — Recommendation: Reduce the application memory to 1GB or less.
- factory-processor (dekt-group-org / space-y): 1536M — Recommendation: Reduce the application memory to 1GB or less.

Compliant: 6 factory apps at or under 1GB.
```

## Error handling

- If an MCP call fails, report the error clearly.
- If memory is missing for an app, note "memory unknown" for that app and still list it if it might exceed 1GB once known.
