Show current project status and task progress.

## Steps

1. Read `.claude/tasks.md`
2. Find current in-progress task
3. Calculate completion percentage
4. Run `git status` for uncommitted changes
5. Show summary

## Output Format

```
PROJECT STATUS

Current Task: [Task name]
Status: [in progress/pending/blocked]
Progress: [X%] ([Y/Z checkboxes complete])

Recent Activity:
- Last commit: [message] ([time ago])
- Files modified: [count]

Uncommitted Changes:
[List any uncommitted files]

Next Up:
- [Next available task based on dependencies]

Quick Actions:
- cm done - Mark current task complete
- cm next - See task recommendations
```
