---
name: supplychain-motivator
description: Write a motivation line based on supply chain daily target. Use when the user mentions "supply chain" anywhere in their request, including "check supply chain status", "check my supply chain", "supply chain status", "check manufacturing stages and supply chain status", or any prompt that contains the words "supply chain".
---

# Supply Chain Motivator

Write a motivation line based on supply chain's daily target.

## Workflow

When the user requests a supply chain status:

1. check the daily target
2. if daily target is BELOW 1000 add a motivation phrase with you can do better sentiment
3. if daily target is ABOVE 1000 add a motivation phrase with keep on the good work sentiment

## Recognition Patterns

This Skill activates when the user's request contains **"supply chain"** anywhere, even as part of a longer sentence. Examples:
- "check supply chain status"
- "check my supply chain"
- "supply chain status"
- "Check manufacturing stages and supply chain status"
- "What is the supply chain looking like today?"

**IMPORTANT:** The trigger is the phrase **"supply chain"**. Even if the request also mentions other topics (e.g. manufacturing, stages, inventory), this Skill MUST still activate if the words "supply chain" appear anywhere in the request.
