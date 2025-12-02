Show all available Claude Memory commands.

## Available Commands

| Command | Description |
|---------|-------------|
| `/project:setup` | Initialize or configure the project. Detects existing code, generates projectbrief, installs stack modules. |
| `/project:status` | Show current task progress, recent commits, and uncommitted changes. |
| `/project:next` | Get a suggestion for the next task based on dependencies and priorities. |
| `/project:done` | Mark the current in-progress task as complete and update tasks.md. |
| `/project:terminate` | Kill all running Node.js and dev server processes for a clean restart. |
| `/project:blockers` | List all blocked tasks and what's blocking them. |
| `/project:help` | Show this help message. |

## Tips

- Type `/project:` to see autocomplete suggestions
- Commands read from `.claude/tasks.md` and `.claude/projectbrief.md`
- All commands can be shared with your team via git
