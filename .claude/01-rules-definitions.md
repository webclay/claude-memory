# CopyHatch Development Rules: Overview & Navigation

This file directs AI assistants on how to use the documentation in this project.

## Documentation Structure

The `.claude` folder contains comprehensive development documentation organized as follows:

### **Master Routing File**
- **`00-start-here.md`** - Start here! Master index that routes to all specialist files

### **Core Foundation Files**
These define WHAT we are building and core technical standards:
- `projectbrief.md` - Project requirements and product specification
- `supabase.md` - Supabase-specific configuration and patterns
- `standards.md` - Project-wide coding standards and conventions
- `tasks.md` - Implementation plan and task tracking
- `07-documentation-maintenance.md` - Documentation update protocols and commit workflow

### **Specialist Playbooks**
Expert guides for specific development tasks (referenced from `00-start-here.md`):

1. **`02-frontend-component.md`** - React components, UI, design system
   - Button/Badge/Icon variants and usage
   - Typography, spacing, responsive patterns
   - Data tables, drawers, dialogs, state management
   - Error handling and loading states

2. **`03-api-server-action.md`** - API routes and server logic
   - GET/DELETE endpoint patterns
   - Authentication and authorization
   - Security best practices (Zod validation, RLS)
   - Response structures and error handling

3. **`04-testing.md`** - Testing strategies and patterns
   - React Testing Library usage
   - Mocking patterns
   - Test organization

4. **`05-database-supabase.md`** - Database and SQL
   - SQL style guide
   - RLS policies and migrations
   - Soft deletes, UUID handling, query optimization
   - Language codes and date formatting

5. **`06-deployment-edge-functions.md`** - Deployment and Edge Functions
   - Netlify deployment configuration
   - Supabase Edge Functions (OpenRouter, Firecrawl, ScreenshotOne)
   - Credits system integration
   - PDF export patterns
   - Analytics dashboard patterns
   - Environment variable management

6. **`07-documentation-maintenance.md`** - Documentation Maintenance & Commit Workflow
   - tasks.md update protocol (auto-triggered on commits)
   - Specialist playbook update rules
   - Documentation synchronization between /docs and /.claude
   - Commit workflow integration and automation
   - Documentation health monitoring

## How to Use This Documentation

### For Claude Code:
All files in `.claude` folder are automatically read. Use `00-start-here.md` to understand which specialist file to reference for your current task.

### For Cline/Other AI Assistants:
1. Always keep **Core Foundation Files** active in context
2. Toggle **Specialist Playbooks** as needed based on the task
3. Start with `00-start-here.md` to find the right specialist file

## Priority Rules

**If a request conflicts with documented rules:**
1. The documented rules in `.claude` folder are the **highest authority**
2. Specialist files override general guidance
3. `projectbrief.md` defines the product requirements
4. Always ask for clarification rather than guessing

**Pattern Consistency:**
- All patterns documented in `.claude` files must be followed
- Badge colors, button variants, API structures, etc. are standardized
- New patterns should be added to the appropriate specialist file