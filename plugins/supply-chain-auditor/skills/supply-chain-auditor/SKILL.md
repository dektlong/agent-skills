---
name: supply-chain-auditor
description: Audits Cloud Foundry apps with "factory" in the name to ensure memory does not exceed 1028M. Uses CF MCP server to list orgs/spaces/apps, filter to factory-named apps, and report any over the limit. Use when the user asks to audit supply chain, check factory app memory, enforce memory cap on factory apps, or run supply chain audit.
---

# Supply Chain Auditor

## Purpose

This skill performs a **memory-cap audit** on factory-named apps only:

1. **Scope**: Only apps whose name contains "factory" (case-insensitive).
2. **Rule**: Memory must **not exceed 1028M**. Flag any app with memory > 1028M.

**Out of scope:** Routes, services, non-factory apps, instance counts, staleness. Do not list or analyze apps that do not have "factory" in the name.

## Prerequisites

Verify the Cloud Foundry MCP server is available. If not configured, tell the user to set it up first.

## Audit Workflow

### Step 1: Determine scope

- **Default**: Audit all spaces in org `dekt-org-group`.
- If the user specifies another org or space(s), use that scope only.

Use CF MCP tools to:
- Determine org (default `dekt-org-group`) and which spaces to inspect.
- For each `(org, space)`, get the list of applications.

### Step 2: Filter to factory apps only

After retrieving apps for each space:

- **Keep only** apps where the name contains `"factory"` (case-insensitive).
- Discard all other apps; do not list or mention them by name in the report.
- For each factory app, get app details including **memory allocation** (in MB).

Examples:
- INCLUDE: "my-factory-app", "Factory-Service", "data-factory-processor"
- EXCLUDE: "web-app", "api-service", "processor"

If no factory-named apps exist in the inspected scope, report that and end.

### Step 3: Apply memory rule

For each factory app:

- **Compliant**: memory ≤ 1028M (e.g. 512M, 1024M, 1028M).
- **Non-compliant**: memory > 1028M → flag for the report.

### Step 4: Report

**Report structure:**

1. **Scope**: Org(s) and space(s) inspected; note "Only apps with 'factory' in the name."
2. **Summary**: Total factory apps audited; count exceeding 1028M; count compliant.
3. **Apps exceeding 1028M** (if any): For each, list:
   - App name
   - Org / space
   - Allocated memory (e.g. 2048M)
   - Limit: 1028M
4. **Compliant factory apps** (optional): Count or short list.

**Verification before responding:**

- [ ] Only factory-named apps were evaluated and listed.
- [ ] Every app over 1028M is named with its actual memory value.
- [ ] No non-factory apps appear in the report (except possibly a single total count of "X total apps in scope").

## Example output

```
Supply Chain Audit (memory cap 1028M)
Scope: dekt-org-group, all spaces. Only apps with 'factory' in the name.

Summary:
- Factory apps audited: 5
- Exceeding 1028M: 2
- Compliant (≤1028M): 3

Apps exceeding 1028M:
- my-factory-api (dekt-org-group / dekt-chatbot-group): 2048M — limit 1028M
- factory-processor (dekt-org-group / dekt-chatbot-group): 1536M — limit 1028M

Compliant: 3 factory apps at or under 1028M.
```

## Error handling

- If an MCP call fails, report the error clearly.
- If memory is missing for an app, note "memory unknown" for that app and still list it if it might exceed the cap once known.
