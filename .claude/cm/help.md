Show all available Claude Memory commands.

## Available Commands

| Command | Description |
|---------|-------------|
| `cm setup` | Initialize or configure the project. Detects existing code, generates projectbrief, installs stack modules. |
| `cm status` | Show current task progress, recent commits, and uncommitted changes. |
| `cm next` | Get a suggestion for the next task based on dependencies and priorities. |
| `cm done` | Mark the current in-progress task as complete and update tasks.md. |
| `cm terminate` | Kill all running Node.js and dev server processes for a clean restart. |
| `cm blockers` | List all blocked tasks and what's blocking them. |
| `cm update` | Check for Claude Memory updates and install them. |
| `cm help` | Show this help message. |

## Tips

- Type `cm` followed by a command name (e.g., `cm status`)
- Commands read from `.claude/tasks.md` and `.claude/projectbrief.md`
- All commands can be shared with your team via git
