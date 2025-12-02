Mark the current in-progress task as complete.

## Steps

1. Read `.claude/tasks.md`
2. Find task with status "in progress"
3. Update status to "complete"
4. Mark all checkboxes as done
5. Check Feature Dependencies in `projectbrief.md` for newly unblocked tasks
6. Suggest next task to work on
7. Ask if user wants to commit the changes

## Output Format

```
TASK COMPLETED

Marked complete: [Task name]

Updated in tasks.md:
- Status: complete
- All checkboxes marked done

Newly Unblocked Tasks:
- [Task X] - was waiting on this task
- [Task Y] - all dependencies now satisfied

Suggested Next Task: [Task name]
Reason: [Why this task is recommended]

Would you like to:
A. Commit these changes
B. Start the suggested task
C. See all available tasks (/project:next)
```
