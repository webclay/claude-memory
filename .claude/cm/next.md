Suggest the next task to work on based on dependencies and priorities.

## Steps

1. Read `.claude/tasks.md`
2. Read Feature Dependencies from `.claude/projectbrief.md`
3. Find tasks that are:
   - Status: pending
   - All dependencies completed
4. Sort by priority (high → medium → low)
5. Present top recommendation with reasoning
6. Offer 2 alternatives if available

## Output Format

```
NEXT TASK RECOMMENDATION

Recommended: [Task name]

Why this task?
- All dependencies completed ([list them])
- Priority: [high/medium/low]
- Estimated scope: [small/medium/large]
- Aligns with recent work on [related area]

Alternative Options:
1. [Task X] - [brief reason]
2. [Task Y] - [brief reason]

Would you like to start "[Recommended task]"?
```
