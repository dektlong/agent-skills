---
name: factory-apps-auditor
description: Audit factory apps for compliance with memory allocation standards, instance counts, and deployment staleness. Applies specific audit rules - Java apps must use 1024M memory, non-Java apps must use 512M, identifies multi-instance apps, and flags apps not deployed in 6+ months. Use when the user asks to audit factory apps check factory apps compliance, review factory apps against standards, or evaluate factory apps for configuration issues. Trigger words include "audit", "compliance", "standards", "check configuration".
---

# Factory Apps Auditor

## MANDATORY: Apply ONLY These Three Audit Criteria

This skill performs a **compliance audit** using exactly three criteria. Do NOT add any other analysis:

**The Three Criteria:**
1. **Memory Allocation**: Java apps = 1024M, Non-Java apps = 512M (strict equality)
2. **Instance Count**: Flag any app running >1 instance
3. **Deployment Staleness**: Flag running apps not deployed in 180+ days
   
**DO NOT include in this audit:**
- Any app that does NOT have the word 'factory' in its name (case-insensitive)
  - **HARD RULE:** Do NOT list or mention non-factory apps anywhere in the output (no names, no per-app details). They should only ever appear, at most, as a single total count like "The space contains X total apps" without naming them.
- Routes or orphaned routes
- Service instances or bindings
- Resource optimization suggestions
- Storage/disk analysis
- General space health assessments
- Custom recommendations

**If the user wants general space analysis:** This is the WRONG skill. Use CF MCP tools conversationally without this skill.

**This skill triggers when:** User says "audit" factory apps, OR asks to check compliance/standards.

## Prerequisites

Verify the Cloud Foundry MCP server is available before proceeding. If the user hasn't configured the CF MCP server, inform them they need to set it up first.

## Audit Workflow

### Step 1: Gather Required Parameters
Primary behavior:

- **By default, audit ALL accessible orgs and spaces.**
- If the user explicitly restricts scope (e.g., a specific org, a list of orgs, a specific space, or a list of spaces), only audit within that scope.

Examples of user scope:
- "Audit factory apps in all spaces" → audit all orgs and all spaces you can see.
- "Audit factory apps in org `foo-org`" → audit all spaces in `foo-org` only.
- "Audit factory apps in space `bar-space` in org `foo-org`" → audit only that single space.

Implementation guidance:
- Use the CF MCP tools to:
  - List organizations (for the default all-orgs behavior).
  - For each selected organization, list spaces.
  - For each selected space, list apps.
  - Then apply the factory-name filter and audit criteria below.

### Step 2: Retrieve and Filter Apps

Use the Cloud Foundry MCP server tools to:
- Determine the set of orgs and spaces to inspect based on Step 1.
- For each `(org, space)` pair in that set, get the list of applications.

**Typical tool calls:**
- List all orgs (for the default "all orgs" behavior)
- For each org, list all spaces
- For each space, list all apps in that space using the appropriate CF MCP tool
- **CRITICAL FILTERING STEP:** After retrieving the app list for a space, immediately filter to include ONLY apps that contain the word "factory" in their name (case-insensitive)
  - Example: "my-factory-app", "Factory-Service", "data-factory-processor" → INCLUDE
  - Example: "web-app", "api-service", "processor" → EXCLUDE
- For each **factory app**, retrieve detailed information including:
  - App name
  - Org name
  - Space name
  - State (running, stopped, etc.)
  - Buildpack(s) used
  - Memory allocation
  - Number of instances
  - Last uploaded/deployed timestamp

**IMPORTANT:** If no apps with "factory" in the name are found in ANY inspected space, inform the user that no factory apps exist in the inspected scope and end the audit.

### Step 3: Evaluate Apps Against Audit Criteria

**REMINDER:** Only evaluate apps that passed the "factory" name filter from Step 2.

Analyze each factory app against the following criteria:

#### Memory Allocation Standards

**Java Buildpack Apps:**
- Expected: 1024M (1G) of memory
- Flag if: Memory allocation is not exactly 1024M
- Severity: Configuration inconsistency

**Non-Java Apps (all other buildpacks):**
- Expected: 512M of memory
- Flag if: Memory allocation is not exactly 512M
- Severity: Configuration inconsistency

#### Instance Count

**All Apps:**
- Flag if: App is running more than 1 instance
- Reason: Multi-instance apps may indicate high-availability requirements or potential scaling issues
- Note: This is informational, not necessarily a problem

#### Deployment Staleness

**All Running Apps:**
- Flag if: Last deployed/updated more than 6 months ago (180 days)
- Reason: May be stale, unmaintained, or candidates for review
- Calculate: Compare current date with last uploaded timestamp
- Severity: Potential maintenance risk

### Step 4: Generate Audit Report

**CRITICAL REMINDER - AUDIT SCOPE:**
This audit covers ONLY apps with "factory" in the name and ONLY three criteria (memory, instances, staleness).
DO NOT include: routes, services, resource optimization, or any recommendations.
If you find yourself analyzing routes, services, or making optimization suggestions, STOP - you are outside the scope of this skill.

**CRITICAL REPORTING REQUIREMENTS:**
- MUST include specific app names for EVERY finding
- MUST include actual values (memory, instance count, dates) for every flagged app
- MUST show expected vs. actual values for memory issues
- MUST calculate and show days since deployment for stale apps
- DO NOT provide vague summaries like "2 apps have issues" without listing which apps
- DO NOT list or describe ANY app that does not contain "factory" in its name (case-insensitive) – these apps must be completely excluded from all detailed sections and examples.
- MUST note in the report header that only apps with "factory" in the name were audited

Structure the audit report with the following sections:

**1. Audit Summary**
- Scope: "Only apps with 'factory' in the name"
- Total factory apps audited (count, across all inspected orgs and spaces)
- Apps with memory allocation issues (count + will be listed below)
- Apps running multiple instances (count + will be listed below)
- Apps potentially stale (count + will be listed below)
- Apps fully compliant (count)

**2. Memory Allocation Issues**

For EACH app that doesn't meet memory standards, list:
- App name
- Org and space (e.g., `org-name / space-name`)
- Buildpack type (Java or other)
- Expected memory (1024M for Java, 512M for others)
- Actual memory allocation
- State (running/stopped)

Example format:
```
Memory Allocation Issues (3 apps):

Java Apps - Expected 1024M:
- my-factory-app: 512M (should be 1024M) - RUNNING
- factory-service-2: 2048M (should be 1024M) - STOPPED

Non-Java Apps - Expected 512M:
- data-factory-processor: 1024M (should be 512M) - RUNNING
```

**3. Multi-Instance Apps**

For EACH app running more than 1 instance, list:
- App name
- Org and space (e.g., `org-name / space-name`)
- Current instance count
- Buildpack type
- State

Example format:
```
Multi-Instance Apps (2 apps):

- my-factory-app: 3 instances (Java buildpack) - RUNNING
- factory-worker: 2 instances (Python buildpack) - RUNNING
```

**4. Potentially Stale Apps**

For EACH running app not deployed in 6+ months, list:
- App name
- Org and space (e.g., `org-name / space-name`)
- Last deployed date (YYYY-MM-DD format)
- Days since last deployment (calculated)
- Buildpack type

Example format:
```
Potentially Stale Apps (2 apps):

- legacy-factory-api: Last deployed 2024-03-15 (274 days ago) - Java buildpack
- old-factory-processor: Last deployed 2024-01-10 (339 days ago) - Python buildpack
```

**5. Compliant Apps** (optional, can be summary)

Either list compliant apps OR provide count:
- "3 factory apps are fully compliant with all standards"
- Or list specific app names if count is small

**VERIFICATION CHECKLIST BEFORE RESPONDING:**
- [ ] Did I filter to ONLY apps with "factory" in the name?
- [ ] Did I list specific app names for every issue found?
- [ ] Did I include actual values (memory, instances, dates) for each flagged app?
- [ ] Did I calculate days since deployment for stale apps?
- [ ] Did I show expected vs. actual for memory issues?
- [ ] If I said "2 apps have X issue," did I name both apps?
- [ ] Did I note in the report that only factory apps were audited?

## Example Interactions

**User request:** "audit factory apps"

**Response flow:**
1. Confirm org is "dekt-org-group" and space is "dekt-chatbot-group"
2. Use CF MCP tools to list apps in dekt-org-group/dekt-chatbot-group
3. **Filter to only apps containing "factory" in the name (case-insensitive)**
4. Retrieve details for each factory app
5. Evaluate against all criteria
6. Generate structured audit report with findings

**Example of GOOD audit output:**
```
Factory Apps Audit Report
(dekt-chatbot-group space, dekt-org-group org)
Scope: Only apps with 'factory' in the name

Audit Summary:
- Total factory apps audited: 5
- Memory allocation issues: 2 apps
- Multi-instance apps: 1 app
- Potentially stale apps: 2 apps
- Fully compliant: 0 apps

Memory Allocation Issues (2 apps):

Java Apps - Expected 1024M:
- legacy-factory-api: 512M (should be 1024M) - RUNNING

Non-Java Apps - Expected 512M:
- python-factory-worker: 1024M (should be 512M) - RUNNING

Multi-Instance Apps (1 app):

- critical-factory-service: 3 instances (Java buildpack) - RUNNING

Potentially Stale Apps (2 apps):

- legacy-factory-api: Last deployed 2024-03-15 (274 days ago) - Java buildpack
- old-factory-processor: Last deployed 2024-02-01 (316 days ago) - Python buildpack

Compliant Apps: None - all factory apps have at least one issue.
```

**Example when no factory apps exist:**
```
Factory Apps Audit Report
(dekt-chatbot-group space, dekt-org-group org)

No apps with 'factory' in the name were found in this space. 
The space contains 8 total apps, but none match the factory naming pattern.

Audit cannot proceed - no factory apps to evaluate.
```

**Example of BAD audit output (DO NOT DO THIS):**
```
The audit found several issues. There are 2 apps with memory problems and 2 apps that haven't been updated recently.
```
*This is bad because it doesn't name the apps or provide specific details.*

**Example of VERY BAD audit output - SCOPE VIOLATION (NEVER DO THIS):**
```
Cloud Foundry Space Audit: dekt-chatbot-group (dekt-org-group org)

Applications Summary: 11 total, 5 running, 6 stopped
[Lists ALL apps including non-factory apps]

Service Instances (15)
- User-Provided Services: 12
- Managed Services: 3

Routes (14)
- Orphaned Routes: 3

Recommendations:
1. Clean up orphaned routes
2. Review stopped applications
3. Optimize resource allocation
```
*This is VERY BAD because it:
1. Audits ALL apps instead of filtering for "factory" apps only
2. Analyzes routes and services - OUTSIDE the scope
3. Provides recommendations - NOT part of this audit*

**User request:** "Check my CF space"

**Response flow:**
1. Proceed with audit workflow including the factory name filter

## Important Notes

**Data Requirements:**
- **CRITICAL:** Only apps with "factory" in the name (case-insensitive) should be audited
- Only running apps should be evaluated for deployment staleness
- Stopped apps can be noted separately if relevant
- Memory standards are strict equality checks (not ranges)
- Be precise with buildpack identification - check for "java" in buildpack name (case-insensitive)
- When calculating staleness, use current date and clear date arithmetic

**Reporting Requirements:**
- ALWAYS filter to factory apps first
- ALWAYS note the filtering scope in the report header
- ALWAYS list specific app names for every finding
- NEVER provide summaries without details (e.g., "2 apps have issues" without naming them)
- ALWAYS include actual values: memory amounts, instance counts, deployment dates
- ALWAYS show expected vs. actual for memory issues
- ALWAYS calculate and show days since deployment for stale apps

**Error Handling:**
- If any MCP tool calls fail, report the error clearly to the user
- If buildpack information is null/missing, note this as a configuration issue
- If deployment timestamp is unavailable, note this limitation
- If no factory apps are found, clearly inform the user and end the audit
