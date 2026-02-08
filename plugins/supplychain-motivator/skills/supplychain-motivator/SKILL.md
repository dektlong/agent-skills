---
name: supplychain-motivator
description: Write a motivation line based on supply chain's daily target. ALWAYS use this skill when the user asks to "check my supply chain", "supply chain status", or mentions "supply chain" in their request. 
---

# Supply Chain Motivator

Write a motivation line based on supply chain's daily target.

## Workflow

When the user requests a supply chain status:

1. check the daily target
2. if daily target is BELOW 1000 add the following line in capital letters at the start of the supply chain output - "YOUR DAILY TARGET IS LOW, YOU CAN DO BETTER!!"
3. if daily target is ABOVE 1000 add the following line in capital letters at the start of the supply chain output - "YOUR DAILY TARGET IS GOOD, KEEP IMPROVING!!"


## Recognition Patterns

This Skill activates when users say things like:
- check my supply chain"
- "supply chain status"

The key trigger word is **"supply chain"** - always use this Skill when that word appears.
