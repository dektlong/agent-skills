---
name: factory-audit
description: Finds Cloud Foundry apps with "factory" in the name across all orgs and all spaces, flags apps whose memory exceeds 1GB, and recommends reducing application memory. Use when the user asks to audit factory apps memory, find factory apps over 1GB, check factory app memory across all orgs/spaces, or run a factory memory audit.
---

# Factory Audit

## Purpose

This skill performs a **memory audit** on factory-named apps **across all orgs and all spaces**:

1. **Scope**: All organizations and all spaces (no default org; inspect everything the CF MCP can access).
2. **Filter**: Only apps whose name contains `"factory"` (case-insensitive).
3. **Rule**: Flag any app with memory **over 1GB (1024M)** and recommend reducing application memory.

**Out of scope:** Routes, services, non-factory apps, instance counts, staleness. Do not list or analyze apps that do not have "factory" in the name.

## Prerequisites

Verify the Cloud Foundry MCP server is available. If not configured, tell the user to set it up first.

## Audit Workflow

### Step 1: Enumerate orgs and spaces

- **Scope is all orgs and all spaces.** Do not restrict to a single org unless the user explicitly asks.
- Use CF MCP tools to:
  - List all organizations.
  - For each organization, list all spaces.
  - For each `(org, space)`, get the list of applications (pass org and space to the apps list tool).

### Step 2: Filter to factory apps only

After retrieving apps for each space:

- **Keep only** apps where the name contains `"factory"` (case-insensitive).
- Discard all other apps; do not list or mention them by name in the report.
- For each factory app, get app details (e.g. `applicationDetails`) including **memory allocation** (in MB).

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

1. **Scope**: "All orgs and all spaces. Only apps with 'factory' in the name."
2. **Summary**: Total factory apps audited; count exceeding 1GB; count compliant (≤1GB).
3. **Apps exceeding 1GB** (if any): For each, list:
   - App name
   - Org / space
   - Allocated memory (e.g. 2048M)
   - **Recommendation: Reduce the application memory** (to 1GB or less).
4. **Compliant factory apps** (optional): Count or short list.

**Verification before responding:**

- [ ] All orgs and all spaces were considered (unless user asked for a narrower scope).
- [ ] Only factory-named apps were evaluated and listed.
- [ ] Every app over 1GB is named with its actual memory and includes the recommendation to reduce memory.
- [ ] No non-factory apps appear in the report (except possibly a single total count of "X total apps in scope").

## Example output

```
Factory Audit (memory cap 1GB)
Scope: All orgs and all spaces. Only apps with 'factory' in the name.

Summary:
- Factory apps audited: 8
- Exceeding 1GB: 2
- Compliant (≤1GB): 6

Apps exceeding 1GB (recommend reducing application memory):
- my-factory-api (org-a / space-x): 2048M — Recommendation: Reduce the application memory to 1GB or less.
- factory-processor (org-b / space-y): 1536M — Recommendation: Reduce the application memory to 1GB or less.

Compliant: 6 factory apps at or under 1GB.
```

## Error handling

- If an MCP call fails, report the error clearly.
- If memory is missing for an app, note "memory unknown" for that app and still list it if it might exceed 1GB once known.
