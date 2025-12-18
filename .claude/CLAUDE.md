# Claude Memory

A documentation system helping Claude maintain context across sessions.

## Before Any Task

**Always read `projectbrief.md` first** - Contains:
- Tech stack (framework, database, auth, etc.)
- UI library and design system
- Project features and functionality
- Feature dependencies map

This ensures consistent styling, correct technology choices, and awareness of how features connect.

## Essential Files

| Purpose | File |
|---------|------|
| Project overview & design system | `.claude/projectbrief.md` |
| Current tasks & progress | `.claude/tasks.md` |
| Mistakes to avoid | `.claude/mistakes.md` |
| Feature routing | `.claude/00-start-here.md` |
| Tech-specific patterns | `.claude/stacks/[category]/[tech].md` |
| Triggers & automation | `.claude/triggers.md` |

## Core Rules

### Communication Style
**The user is a product engineer, not a programmer.** They cannot read or write code. Therefore:

1. **Explain your reasoning** - When presenting implementation options, explain *why* one approach is better than another. This helps the user learn how developers think and what matters for different tasks.
2. **Always make a recommendation** - After explaining options, clearly state which approach you recommend and why. Don't leave the decision to the user without guidance.
3. **Summarize in plain language** - When explaining what was built or changed, describe the *outcome* (what the user will see/experience), not just the code.
4. **Only ask about product decisions** - Ask the user about features, UX, and business logic. For technical decisions, explain your thinking and then make the call.
5. **No code snippets in explanations** - Unless the user specifically asks to see code, communicate in plain English.


### Pattern Consistency
Before building ANY feature:
1. Read 2-3 existing examples in the codebase
2. Check `projectbrief.md` for UI library and design system
3. Match established patterns exactly
4. If uncertain, ask the user

### Dependencies
**NEVER install packages or components without explicit user approval.** Always ask first before running `npm install`, `pnpm add`, `npx shadcn add`, or similar commands.

### Scope Discipline
**Do only what was asked.** Do not refactor, "improve", or change code unrelated to the current task. However, if your changes would impact other functionality, **tell the user immediately** so it can be discussed before proceeding.

### Complex Workflows
For large tasks with multiple steps (migrations, fixing many errors, complex builds):
1. Create a checklist in `.claude/scratchpad.md` with all items to address
2. Work through each item one by one
3. Verify each fix before checking it off
4. Move to the next item only after the previous is confirmed working

### Code Quality
- Run `npx ultracite fix` before committing
- Use **camelCase** for all TypeScript/JavaScript variables and functions (not database fields)

## Commands

Type `cm` followed by a command name:

| Command | Action |
|---------|--------|
| `cm setup` | Initialize/configure project |
| `cm status` | Show task progress |
| `cm next` | Suggest next task |
| `cm done` | Mark current task complete |
| `cm terminate` | Kill all dev servers |
| `cm blockers` | List blocked tasks |
| `cm update` | Check for updates |
| `cm help` | Show all commands |
