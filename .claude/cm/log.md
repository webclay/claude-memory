Update the changelog with a summary of work completed this session.

## Steps

1. Read `.claude/changelog.md`
2. Read `.claude/tasks.md` to find recently completed tasks
3. Check git log for recent commits (if available)
4. Ask user: "What would you like to highlight from this session?" (or summarize from context)
5. Create new entry at the top of "Session Log" section with:
   - Today's date and brief title
   - Summary of what was accomplished
   - List of completed tasks, bugs fixed, features added
   - Any notes helpful for future sessions
6. Update "Last Updated" timestamp
7. Save changes to `.claude/changelog.md`

## Output Format

```
CHANGELOG UPDATED

Added entry for: [Date]

[Brief Title]

Summary: [2-3 sentence overview]

Recorded:
- Tasks completed: [list]
- Bugs fixed: [list]
- Features added: [list]

View full changelog: .claude/changelog.md
```

## Important

- Each entry helps future AI sessions understand project history
- Include enough context that someone reading later would understand what was built
- Reference specific files or features when helpful
- Keep summaries concise but informative
