---
name: factory-apps-auditor
description: Audit factory apps for compliance with memory allocation standards, instance counts, and deployment staleness. Applies specific audit rules - Java apps must use 1024M memory, non-Java apps must use 512M, identifies multi-instance apps, and flags apps not deployed in 6+ months. Use when the user asks to audit factory apps check factory apps compliance, review factory apps against standards, or evaluate factory apps for configuration issues. Trigger words include "audit", "compliance", "standards", "check configuration".
---

# Factory Apps Auditor

## ⚠️ CRITICAL FILTER REQUIREMENT - READ FIRST ⚠️

**THIS SKILL AUDITS ONLY APPS WITH "FACTORY" IN THE NAME**

### The Factory Name Filter (MANDATORY)

**STOP AND READ BEFORE PROCEEDING:**

1. **IMMEDIATELY after retrieving apps from any space, you MUST filter to ONLY apps containing "factory" in the name (case-insensitive)**
2. **DO NOT evaluate, list, analyze, or mention ANY app that doesn't have "factory" in its name**
3. **If you find yourself looking at an app called "web-app", "api-service", or anything without "factory" - YOU ARE DOING IT WRONG**

**Implementation Steps:**
```
For each space:
  1. Get list of all apps in space
  2. IMMEDIATELY FILTER: Keep only apps where name.lower().contains("factory")
  3. Discard all other apps - they don't exist for this audit
  4. ONLY THEN evaluate the filtered factory apps
```

**Examples:**
- ✅ INCLUDE: "my-factory-app", "Factory-Service", "data-factory-processor", "FACTORY-api"
- ❌ EXCLUDE: "web-app", "api-service", "processor", "data-pipeline", "my-service"

**If no apps with "factory" in the name exist:** Report this and END the audit immediately.

---

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

- **By default, audit ALL spaces within org `dekt-org-group`.**
- Do NOT inspect any other org unless the user explicitly asks for a different org.
- If the user restricts scope further (e.g., a specific space or list of spaces within an org), only audit within that narrower scope.

Examples of user scope:
- "Audit factory apps in all spaces" → audit all spaces in `dekt-org-group`.
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

### ⚠️ CRITICAL FILTERING STEP - DO THIS IMMEDIATELY ⚠️

**AFTER RETRIEVING THE APP LIST FOR EACH SPACE:**

```python
# Pseudo-code - THIS IS MANDATORY
all_apps = cf_get_apps_for_space(org, space)

# IMMEDIATELY FILTER TO FACTORY APPS ONLY
factory_apps = [app for app in all_apps if "factory" in app.name.lower()]

# DISCARD all_apps - only work with factory_apps from this point forward
# If you reference all_apps again, YOU ARE DOING IT WRONG
```

**Only apps in `factory_apps` should be processed further**

- Example: "my-factory-app", "Factory-Service", "data-factory-processor" → INCLUDE
- Example: "web-app", "api-service", "processor" → EXCLUDE (never mention these)

**After filtering, for each factory app, retrieve detailed information:**
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

**DOUBLE-CHECK:** Before evaluating any app, verify its name contains "factory". If not, SKIP IT.

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
- App name (MUST contain "factory")
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
- App name (MUST contain "factory")
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
- App name (MUST contain "factory")
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
- [ ] Did I verify EVERY app name I'm reporting contains "factory"?
- [ ] Did I list specific app names for every issue found?
- [ ] Did I include actual values (memory, instances, dates) for each flagged app?
- [ ] Did I calculate days since deployment for stale apps?
- [ ] Did I show expected vs. actual for memory issues?
- [ ] If I said "2 apps have X issue," did I name both apps?
- [ ] Did I note in the report that only factory apps were audited?
- [ ] Did I accidentally include any non-factory apps? (If yes, REMOVE THEM)

## Example Interactions

**User request:** "audit factory apps"

**Response flow (default, all orgs and spaces):**
1. Determine whether the user restricted scope; if not, plan to audit **all accessible orgs and spaces**.
2. Use CF MCP tools to:
   - List all orgs
   - For each org, list all spaces
   - For each `(org, space)`, list apps in that space
3. **⚠️ CRITICAL: Within each space, IMMEDIATELY filter to only apps containing "factory" in the name (case-insensitive)**
4. Retrieve details ONLY for factory apps (not all apps)
5. Evaluate ONLY factory apps against all criteria
6. Generate a single structured audit report with findings aggregated across all inspected orgs and spaces
7. **FINAL CHECK: Review your report - if you see app names without "factory" in them, DELETE those entries**

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

**Example when no factory apps exist anywhere in scope:**
```
Factory Apps Audit Report

Scope: Only apps with 'factory' in the name

No apps with 'factory' in the name were found in any inspected org/space.
The inspected scope contains 24 total apps, but none match the factory naming pattern.

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

Apps Audited:
- web-app (512M, 1 instance) ❌ WRONG - no "factory" in name
- api-service (1024M, 2 instances) ❌ WRONG - no "factory" in name
- my-factory-app (512M, 1 instance) ✓ This one is OK
- data-processor (512M, 1 instance) ❌ WRONG - no "factory" in name

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
1. Lists apps without "factory" in the name (web-app, api-service, data-processor)
2. Analyzes routes and services - OUTSIDE the scope
3. Provides recommendations - NOT part of this audit
4. Shows total app counts that include non-factory apps*

**User request:** "Check my CF space"

**Response flow:**
1. Proceed with audit workflow including the factory name filter
2. Remember: Only apps with "factory" in name should appear in any output

## Important Notes

**Data Requirements:**
- **CRITICAL:** Only apps with "factory" in the name (case-insensitive) should be audited
- Filter IMMEDIATELY after retrieving app lists, BEFORE any evaluation
- Only running apps should be evaluated for deployment staleness
- Stopped apps can be noted separately if relevant
- Memory standards are strict equality checks (not ranges)
- Be precise with buildpack identification - check for "java" in buildpack name (case-insensitive)
- When calculating staleness, use current date and clear date arithmetic

**Reporting Requirements:**
- ALWAYS filter to factory apps first, before any evaluation
- ALWAYS note the filtering scope in the report header
- ALWAYS list specific app names for every finding
- EVERY app name listed MUST contain "factory"
- NEVER provide summaries without details (e.g., "2 apps have issues" without naming them)
- ALWAYS include actual values: memory amounts, instance counts, deployment dates
- ALWAYS show expected vs. actual for memory issues
- ALWAYS calculate and show days since deployment for stale apps
- BEFORE submitting report: scan for any app names without "factory" and remove them

**Error Handling:**
- If any MCP tool calls fail, report the error clearly to the user
- If buildpack information is null/missing, note this as a configuration issue
- If deployment timestamp is unavailable, note this limitation
- If no factory apps are found, clearly inform the user and end the audit

## Final Reminder

**IF YOU SEE AN APP NAME IN YOUR OUTPUT THAT DOESN'T CONTAIN "FACTORY", YOU HAVE MADE A CRITICAL ERROR. DELETE IT IMMEDIATELY.**
