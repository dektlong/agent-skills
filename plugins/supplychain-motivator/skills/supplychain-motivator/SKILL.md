---
name: supplychain-motivator
description: Check supply chain status and report the daily target with a motivation line. ALWAYS activate this skill when the words "supply chain" appear anywhere in the user's request, even in multi-task prompts. Trigger phrases include "supply chain status", "check supply chain", "check manufacturing stages and supply chain status". This skill handles the supply chain portion of the request and runs alongside other skills.
---

# Supply Chain Motivator

This skill handles the **supply chain status** portion of any user request. It checks the daily target and adds a motivation line.

## CRITICAL: When to activate

**ALWAYS activate this skill when the words "supply chain" appear ANYWHERE in the user's request.**

This is true even when the request contains multiple tasks. For example, in a prompt like:

> "Check manufacturing stages and supply chain status. Audit factory applications. Check for maintenance issues. Post results to Google chat."

This skill handles the **"supply chain status"** part. Other skills handle the other parts (factory audit, Google chat, etc.). **Do not skip this skill just because other tasks are also present.**

## Workflow

When activated:

1. Check the daily target
2. If daily target is BELOW 1000 → add a motivation phrase with "you can do better" sentiment
3. If daily target is ABOVE 1000 → add a motivation phrase with "keep up the good work" sentiment

## Recognition Patterns

Activate when the request contains **"supply chain"** anywhere, including:
- "check supply chain status"
- "check my supply chain"
- "supply chain status"
- "Check manufacturing stages and supply chain status"
- "Check manufacturing stages and supply chain status. Audit factory applications. Check for maintenance issues. Post results to Google chat."
- Any multi-task prompt where "supply chain" appears in any sentence
