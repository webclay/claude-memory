Initialize or configure this project with Claude Memory.

## Workflow

1. Check if `.claude/projectbrief.md` exists
2. **If no code exists:** Start interactive questionnaire to create projectbrief
3. **If code exists:** Analyze codebase, detect tech stack, generate projectbrief
4. Generate Feature Dependencies map in projectbrief
5. Install relevant stack modules from `.claude/stacks/`
6. Show setup completion message with available commands

## Full Workflow Details

See `.claude/guides/setup-workflow.md` for the complete setup workflow including:
- ASCII header to display
- Questionnaire format rules
- Tech stack detection logic
- Stack module installation

## On Completion

Show the user:

```
Setup Complete!

Your project is now configured with Claude Memory.

Available Commands:
- /project:status  - See project progress
- /project:next    - Get next task suggestion
- /project:done    - Complete current task
- /project:terminate - Kill dev servers
- /project:help    - See all commands

Next Steps:
1. Review your projectbrief.md
2. Start working - just tell me what to build!
```
