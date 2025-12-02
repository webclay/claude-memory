# Common AI Mistakes - DO NOT REPEAT

**Purpose:** Track recurring mistakes so Claude avoids them. Update this file whenever you catch Claude making a repeated error.

---

## Installation & Dependencies
- Installing packages without asking first
- Using `npm` when project uses `pnpm` (or vice versa)
- Adding shadcn/UI components without user approval
- Installing outdated package versions

## Scope & Changes
- Refactoring unrelated code while fixing a bug
- "Improving" code that wasn't asked to be changed
- Adding features that weren't requested
- Changing file structure without discussion
- Modifying configuration files unnecessarily

## Patterns & Consistency
- Using different spacing than existing components
- Creating new component variants instead of using existing ones
- Not checking existing code before creating new patterns
- Using different naming conventions than the codebase
- Ignoring the design system defined in projectbrief.md

## Code Quality
- Leaving console.log statements in code
- Using `any` type instead of proper TypeScript types
- Not running linter before suggesting commit
- Creating overly complex solutions for simple problems
- Not handling error cases

## Communication
- Making assumptions instead of asking clarifying questions
- Not warning about side effects before making changes
- Proceeding with large changes without confirmation

---

**How to use this file:**

1. When Claude makes a mistake, add it to the appropriate category
2. Be specific - describe what went wrong
3. Claude will check this file before starting work on new features
4. Remove items that are no longer relevant
