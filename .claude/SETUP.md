# .claude Memory System Setup Instructions

**For:** Claude Code AI Assistant
**Purpose:** Initialize this project's memory system
**Version:** 1.0.0

---

## What You Need to Do

When the user asks you to "setup the memory system", "initialize .claude", or "read SETUP.md and initialize", follow these steps:

### Step 1: Read & Understand Project

1. **Read `projectbrief.md` completely**
   - Identify the project name and purpose
   - Note all features and requirements
   - Understand the tech stack mentioned
   - Identify user roles and permissions
   - Note any special constraints or requirements

2. **Extract Key Information:**
   - Project name: `[extract from projectbrief]`
   - Primary purpose: `[extract from projectbrief]`
   - Main features: `[list from projectbrief]`
   - Tech stack components: `[identify from projectbrief]`

---

### Step 2: Identify Stack Modules

3. **List all files in `.claude/` folder**

4. **Identify which stack modules have been copied:**
   - `02-frontend-*.md` → Frontend framework (Next.js, TanStack, Remix, etc.)
   - `03-api-*.md` → API/backend approach (Server Actions, Convex, tRPC, etc.)
   - `05-database-*.md` + companion file → Database system (Supabase, Convex, Firebase, etc.)
   - `06-deployment-*.md` → Deployment platform (Netlify, Vercel, Cloudflare, etc.)
   - `auth-*.md` → Authentication system (Better Auth, Supabase Auth, Clerk, etc.)
   - `ai-*.md` → AI SDK (Vercel AI SDK, OpenAI, Anthropic, etc.)
   - `email-*.md` → Email service (Resend, SendGrid, etc.)
   - `ui-*.md` → UI component library (Shadcn, Kibo, Motia, etc.)

5. **Create a Tech Stack Summary:**
   ```
   Detected Stack:
   - Frontend: [framework name]
   - API: [API approach]
   - Database: [database system]
   - Deployment: [platform]
   - Auth: [auth system or "Not specified"]
   - AI: [AI SDK or "Not specified"]
   - Email: [email service or "Not specified"]
   - UI: [UI library or "Not specified"]
   ```

---

### Step 3: Update Core Files

**A. Update `00-start-here.md`:**

1. **Replace project name:**
   - Find: `[PROJECT_NAME]` or existing project name
   - Replace with: Actual project name from projectbrief.md

2. **Update QUICK REFERENCE section (bottom of file):**
   ```markdown
   ## QUICK REFERENCE

   - **Package Manager:** [from projectbrief or pnpm default]
   - **Dev Server:** [from projectbrief or localhost:3000 default]
   - **Framework:** [detected frontend framework]
   - **Database:** [detected database]
   - **UI Libraries:** [detected UI libraries]
   - **Styling:** [Tailwind CSS or other from projectbrief]
   - **AI SDK:** [detected AI SDK if present]
   - **Icons:** [from projectbrief or HugeIcons default]
   - **Logging:** [from projectbrief or Winston default]
   - **Testing:** [Jest + React Testing Library or from projectbrief]
   - **Validation:** [Zod or from projectbrief]
   - **Auth:** [detected auth system if present]
   - **Email:** [detected email service if present]
   ```

3. **Update routing sections to reference correct stack modules:**
   - Update section 1 (Frontend) to reference the correct `02-frontend-*.md` file
   - Update section 2 (API) to reference the correct `03-api-*.md` file
   - Update section 4 (Database) to reference the correct `05-database-*.md` file
   - Update section 5 (Deployment) to reference the correct `06-deployment-*.md` file

4. **Update GLOBAL DEVELOPMENT RULES if specified in projectbrief:**
   - Package manager
   - Local development port
   - Logging library

**B. Populate `tasks.md`:**

1. **Read projectbrief.md features and requirements**

2. **Create implementation tasks following this structure:**
   ```markdown
   # Project Implementation Tasks

   **Project:** [Project Name]
   **Last Updated:** [Today's date]

   ---

   ## Task 1: Project Setup & Configuration
   **Status:** pending
   **Priority:** High
   **Dependencies:** None

   ### Checklist:
   - [ ] Initialize project with [framework]
   - [ ] Configure [database]
   - [ ] Set up environment variables
   - [ ] Configure [deployment platform]
   - [ ] Set up development workflow

   ### Test Strategy:
   - [ ] Verify local development environment works
   - [ ] Test database connection
   - [ ] Verify environment variables load correctly

   ---

   ## Task 2: Authentication & Authorization
   **Status:** pending
   **Priority:** High
   **Dependencies:** Task 1

   ### Checklist:
   - [ ] Set up [auth system]
   - [ ] Implement user registration
   - [ ] Implement login/logout
   - [ ] Set up session management
   - [ ] Configure user roles: [list from projectbrief]

   ### Test Strategy:
   - [ ] Test registration flow
   - [ ] Test login/logout
   - [ ] Test session persistence
   - [ ] Test role-based access

   ---

   ## Task 3: Database Schema & Migrations
   **Status:** pending
   **Priority:** High
   **Dependencies:** Task 1

   ### Checklist:
   - [ ] Design database schema
   - [ ] Create tables: [list main tables from projectbrief]
   - [ ] Set up relationships
   - [ ] Configure RLS/security rules
   - [ ] Create indexes for performance

   ### Test Strategy:
   - [ ] Verify schema creation
   - [ ] Test CRUD operations
   - [ ] Test security rules
   - [ ] Test queries performance

   ---

   [Continue with feature-specific tasks from projectbrief.md]

   ## Task N-1: Testing & Quality Assurance
   **Status:** pending
   **Priority:** Medium
   **Dependencies:** All feature tasks

   ### Checklist:
   - [ ] Unit tests for critical functions
   - [ ] Component tests for UI
   - [ ] Integration tests for API
   - [ ] E2E tests for critical flows
   - [ ] Accessibility testing

   ### Test Strategy:
   - [ ] Achieve [X]% code coverage
   - [ ] All tests passing
   - [ ] No accessibility violations

   ---

   ## Task N: Deployment & Launch
   **Status:** pending
   **Priority:** High
   **Dependencies:** Task N-1

   ### Checklist:
   - [ ] Configure production environment
   - [ ] Set up CI/CD pipeline
   - [ ] Deploy to [platform]
   - [ ] Configure domain and SSL
   - [ ] Set up monitoring and logging

   ### Test Strategy:
   - [ ] Production smoke tests
   - [ ] Performance testing
   - [ ] Security audit
   ```

3. **Adapt tasks based on projectbrief.md:**
   - Create one task per major feature mentioned
   - Add sub-tasks (checkboxes) for each feature's components
   - Set dependencies logically
   - Add test strategies for each task

**C. Update `standards.md`:**

1. **Replace `[PROJECT_NAME]` placeholders** with actual project name

2. **Update Entity References section:**
   - Read projectbrief.md for domain entities (e.g., Users, Products, Orders, etc.)
   - Update examples to use actual entity names
   - Add project-specific naming conventions

3. **Update Module/Component Naming Patterns:**
   - If projectbrief mentions specific modules/sections, document naming conventions
   - Example: "Ad Hooks" → "ad-hooks" (kebab-case) → "AdHooks" (PascalCase)

4. **Keep universal coding standards intact:**
   - TypeScript conventions
   - File structure
   - Git workflow
   - Code quality rules

**D. Update `settings.local.json`:**

1. **Replace `[PROJECT_NAME]` in all file paths**

2. **Update command whitelist if needed:**
   - Keep standard git, build, test commands
   - Add project-specific commands if mentioned in projectbrief

3. **Update file path patterns to match project structure:**
   ```json
   {
     "postEditHooks": [
       {
         "pattern": "app/**/*.tsx",
         "command": "npx ultracite fix"
       }
     ]
   }
   ```

---

### Step 4: Verify Integration

1. **Check all cross-references between files work:**
   - `00-start-here.md` routes to correct stack module files
   - `01-rules-definitions.md` lists all present files
   - Stack modules don't reference missing files

2. **Verify no placeholders remain:**
   - Search for `[PROJECT_NAME]`
   - Search for `[PLACEHOLDER]`
   - Search for `[TODO`
   - Replace any found with actual values or mark as user TODO

3. **Confirm file structure:**
   ```
   .claude/
   ├── SETUP.md (this file - can be deleted after setup)
   ├── 00-start-here.md ✓
   ├── 01-rules-definitions.md ✓
   ├── 02-frontend-[stack].md ✓
   ├── 03-api-[stack].md ✓
   ├── 04-testing.md ✓
   ├── 05-database-[stack].md ✓
   ├── [database].md ✓ (e.g., supabase.md, convex.md)
   ├── 06-deployment-[stack].md ✓
   ├── 07-documentation-maintenance.md ✓
   ├── auth-[system].md (optional)
   ├── ai-[sdk].md (optional)
   ├── email-[service].md (optional)
   ├── ui-[library].md (optional)
   ├── CLAUDE.md ✓
   ├── projectbrief.md ✓
   ├── tasks.md ✓
   ├── standards.md ✓
   ├── settings.json ✓
   └── settings.local.json ✓
   ```

---

### Step 5: Confirm Completion

**Report back to user with:**

```markdown
✅ Memory system initialized successfully!

## Detected Configuration

**Project:** [Project Name]
**Purpose:** [Brief summary from projectbrief]

**Tech Stack:**
- Frontend: [detected framework]
- API: [detected API approach]
- Database: [detected database system]
- Deployment: [detected platform]
- Auth: [detected auth system or "Not configured"]
- AI: [detected AI SDK or "Not configured"]
- Email: [detected email service or "Not configured"]
- UI: [detected UI library or "Not configured"]

**Implementation Tasks Created:** [N] tasks

**Key Features Identified:**
1. [Feature 1 from projectbrief]
2. [Feature 2 from projectbrief]
3. [Feature 3 from projectbrief]
...

## Files Updated:
✓ 00-start-here.md - Updated routing and quick reference
✓ tasks.md - Populated with [N] implementation tasks
✓ standards.md - Added project-specific conventions
✓ settings.local.json - Updated project paths

## Next Steps:
1. Review tasks.md to verify implementation plan
2. Start development with any feature request
3. I'll automatically follow the patterns in the stack modules
4. Documentation will stay in sync via 07-documentation-maintenance.md

Ready to start building! 🚀
```

---

## Important Rules

### 1. Never Modify Stack Module Files
- Files like `02-frontend-*.md`, `03-api-*.md`, `05-database-*.md`, `06-deployment-*.md`
- Files like `auth-*.md`, `ai-*.md`, `email-*.md`, `ui-*.md`
- These contain expert patterns - keep them pristine
- Only update the CORE files listed in Step 3 (00, tasks, standards, settings.local)

### 2. Preserve Documentation System Files
- `07-documentation-maintenance.md` - NEVER modify
- `01-rules-definitions.md` - NEVER modify
- `04-testing.md` - NEVER modify
- `CLAUDE.md` - NEVER modify
- `settings.json` - NEVER modify

### 3. Use projectbrief.md as Source of Truth
- All updates derive from projectbrief.md content
- Don't invent features not mentioned
- Ask user for clarification if projectbrief is ambiguous
- If tech stack isn't specified in projectbrief, rely on detected modules

### 4. Handle Missing Information Gracefully
- If projectbrief.md lacks details, use detected module files to infer
- If features are vague, create high-level task structure
- Mark areas needing user input with `[TODO: User to specify - see projectbrief.md]`
- Suggest what information would be helpful to add to projectbrief

### 5. Smart Defaults
- Package manager: `pnpm` (unless projectbrief specifies otherwise)
- Dev port: `localhost:3000` (unless projectbrief specifies otherwise)
- Styling: `Tailwind CSS` (unless projectbrief specifies otherwise)
- Testing: `Jest + React Testing Library` (unless projectbrief specifies otherwise)
- Validation: `Zod` (unless projectbrief specifies otherwise)

---

## Example Initialization Flow

### User Input:
```
"Read SETUP.md and initialize the memory system"
```

### Your Process (Internal):
1. ✓ Read SETUP.md (this file)
2. ✓ Read projectbrief.md
3. ✓ List .claude/ directory files
4. ✓ Identify: TanStack Start + Convex + Better Auth + Vercel AI SDK + Resend
5. ✓ Update 00-start-here.md with tech stack
6. ✓ Create 8 tasks in tasks.md based on projectbrief features
7. ✓ Update standards.md with "TaskMaster" entity patterns
8. ✓ Update settings.local.json with TaskMaster paths
9. ✓ Verify all cross-references work
10. ✓ Confirm no placeholders remain

### Your Output:
```markdown
✅ Memory system initialized successfully!

## Detected Configuration

**Project:** TaskMaster
**Purpose:** AI-powered task management with smart prioritization

**Tech Stack:**
- Frontend: TanStack Start
- API: Convex Functions
- Database: Convex
- Deployment: Vercel
- Auth: Better Auth
- AI: Vercel AI SDK
- Email: Resend
- UI: Motia UI

**Implementation Tasks Created:** 8 tasks
1. Project Setup & Configuration
2. Authentication with Better Auth
3. Convex Database Schema
4. Task Management Core Features
5. AI-Powered Prioritization
6. Email Notifications
7. Testing & Quality Assurance
8. Deployment & Launch

## Files Updated:
✓ 00-start-here.md - Updated routing and quick reference
✓ tasks.md - Populated with 8 implementation tasks
✓ standards.md - Added TaskMaster-specific conventions
✓ settings.local.json - Updated project paths

Ready to start building! 🚀
```

---

## Troubleshooting

### If projectbrief.md is Empty or Missing
```
"⚠️ projectbrief.md is empty or missing key information.

Please fill in at minimum:
- Project name
- Project purpose
- Main features to build
- Preferred tech stack (or I'll use detected modules)

Once updated, I can complete the initialization."
```

### If No Stack Modules Detected
```
"⚠️ No stack modules detected in .claude/ folder.

Expected files like:
- 02-frontend-*.md
- 03-api-*.md
- 05-database-*.md
- 06-deployment-*.md

Please copy your chosen stack modules from claude-memory/stacks/
to the .claude/ folder, then ask me to initialize again."
```

### If Multiple Modules of Same Type Detected
```
"⚠️ Multiple [database/frontend/api] modules detected:
- 05-database-supabase.md
- 05-database-convex.md

Please keep only one and remove the others, then ask me to initialize again."
```

---

## After Initialization

Once setup is complete:
- This SETUP.md file can be deleted (it's only needed once)
- The memory system is now active and ready
- All future development will follow the patterns in the stack modules
- Documentation will automatically stay in sync via 07-documentation-maintenance.md

---

**This file guides Claude through one-time memory system initialization.**
**After setup, normal development begins with Claude reading 00-start-here.md as the entry point.**
