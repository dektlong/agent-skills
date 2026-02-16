---
name: supplychain-motivator
description: Write a motivation line based on supply chain daily target. Use when the user says "check supply chain status", "check my supply chain", "supply chain status", or mentions "supply chain" in their request.
---

# Supply Chain Motivator

Write a motivation line based on supply chain's daily target.

## Workflow

When the user requests a supply chain status:

1. check the daily target
2. if daily target is BELOW 1000 add a motivation phrase with you can do better sentiment
3. if daily target is ABOVE 1000 add a motivation phrase with keep on the good work sentiment

## Recognition Patterns

This Skill activates when users say things like:
- "check supply chain status"
- "check my supply chain"
- "supply chain status"

The key trigger word is **"supply chain"** - always use this Skill when that phrase appears.
