Check for Claude Memory updates and install them.

## Steps

1. Read local version from `.claude/VERSION`
2. Fetch GitHub README to get latest version
3. Compare versions
4. If update available, ask user for update method (automatic or manual)
5. Create backup before updating
6. Apply updates (system files only - never touch user data)
7. Verify update success

## Output Format

```
CLAUDE MEMORY UPDATE

Current Version: [X.X.X]
Latest Version: [Y.Y.Y]

[If update available:]
What's new in vY.Y.Y:
- [Change 1]
- [Change 2]

How would you like to update?
A. Automatic (download from GitHub)
B. Manual (guide me through copying files)

[If already up to date:]
You're already on the latest version!
```

## Important Notes

- **Never update:** projectbrief.md, tasks.md, or user-created files
- **Always update:** System files (CLAUDE.md, guides/, stacks/, templates/)
- **Create backup:** Before any update, backup to `.claude/.backup-vX.X.X/`
- See `.claude/guides/update-system.md` for full implementation details
