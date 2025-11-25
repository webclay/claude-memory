# Setup Workflow - Unified Initialization System

**Purpose:** Single command to initialize the Claude memory system for any project

**Trigger Keyword:** `setup`

---

## Overview

The `setup` command is the **universal entry point** for initializing the Claude memory system. It intelligently adapts based on what exists in your project:

- ✅ **No project brief?** → Starts interactive questionnaire
- ✅ **Project brief exists?** → Analyzes it and recommends stack modules
- ✅ **Some modules already present?** → Identifies gaps and suggests additions
- ✅ **Fully configured?** → Validates and updates configuration

**One keyword. Zero confusion. Complete automation.**

---

## How It Works

### The Setup Process

When a user types `setup`, Claude follows this workflow:

```
User types "setup"
    ↓
Check for .claude/projectbrief.md
    ↓
    ├─→ NOT FOUND ──────────────────────────────────┐
    │                                                 │
    │   1. Start Interactive Questionnaire           │
    │      - Project type                            │
    │      - Key features                            │
    │      - Database needs                          │
    │      - Team experience                         │
    │      - Scale & timeline                        │
    │      - AI requirements                         │
    │      - Deployment target                       │
    │      - UI approach                             │
    │      - Special requirements                    │
    │                                                 │
    │   2. Generate Two Documents                    │
    │      - Create /docs/prd.md                     │
    │        (Full PRD for humans)                   │
    │      - Create .claude/projectbrief.md          │
    │        (Concise brief for AI)                  │
    │      - Link them together                      │
    │                                                 │
    │   3. Copy Required Stack Modules               │
    │      - Framework module                        │
    │      - Database module                         │
    │      - API module                              │
    │      - Deployment module                       │
    │      - Optional modules (auth, AI, etc.)       │
    │                                                 │
    │   4. Initialize Core System                    │
    │      - Copy .claude-core/ files                │
    │      - Update 00-start-here.md routing         │
    │      - Create tasks.md with initial tasks      │
    │      - Configure standards.md                  │
    │                                                 │
    ├─────────────────────────────────────────────────┘
    │
    └─→ FOUND ──────────────────────────────────────┐
                                                      │
        1. Parse Project Brief                       │
           - Extract tech stack                      │
           - Identify features                       │
           - Understand requirements                 │
                                                      │
        2. Analyze Existing Setup                    │
           - Check for .claude/ directory            │
           - List existing stack modules             │
           - Identify missing modules                │
                                                      │
        3. Make Recommendations                      │
           - "Based on your project brief..."        │
           - Recommend missing modules               │
           - Suggest optional enhancements           │
           - Explain why each is needed              │
                                                      │
        4. Execute Setup (with confirmation)         │
           - Copy missing stack modules              │
           - Update routing                          │
           - Create/update tasks.md                  │
           - Validate cross-references               │
                                                      │
        └─────────────────────────────────────────────┘
```

---

## Scenario 1: No Project Brief Exists

### What Happens

Claude starts an interactive questionnaire to understand your project.

### Example Conversation

**User:** `setup`

**Claude:**
```
╔═══════════════════════════════════════════════════════════════════╗
║                                                                   ║
║   ░█▀▀░█░░░█▀█░█░█░█▀▄░█▀▀░░░█▄█░█▀▀░█▄█░█▀█░█▀▄░█░█             ║
║   ░█░░░█░░░█▀█░█░█░█░█░█▀▀░░░█░█░█▀▀░█░█░█░█░█▀▄░░█░             ║
║   ░▀▀▀░▀▀▀░▀░▀░▀▀▀░▀▀░░▀▀▀░░░▀░▀░▀▀▀░▀░▀░▀▀▀░▀░▀░░▀░             ║
║                                                                   ║
║              🚀  Intelligent Setup & Configuration  🚀            ║
║                                                                   ║
║   One command to initialize your entire development environment   ║
║                                                                   ║
╚═══════════════════════════════════════════════════════════════════╝

Welcome! I'll help you set up your project memory system.

I don't see a project brief yet, so let me ask you a few questions to
understand what you're building.

What type of application are you building?

A. Web application (SaaS, dashboard, e-commerce, etc.)
B. Mobile application (iOS, Android, or both)
C. API/Backend service
D. Full-stack application (web + mobile + API)
E. AI-powered application
F. Chrome extension or browser tool
G. WordPress plugin
H. Other

Just reply with the letter or describe in your own words!
```

**User:** `A web application, it's a project management SaaS`

**Claude:**
```
Perfect! A project management SaaS. That helps me understand your needs.

What are the key features you'll need? (You can select multiple)

- Real-time updates (for team collaboration)
- User authentication (individual users and teams)
- File uploads (documents, attachments)
- Background tasks (notifications, scheduled reports)
- Data visualization (charts, gantt charts, analytics)
- Email notifications
- Payment/subscription management
- AI features (smart suggestions, automation)
- Other features you have in mind?
```

**[... questionnaire continues ...]**

### After Questionnaire Completes

**Claude:**
```
✅ Perfect! I have everything I need.

Based on your answers, here's what I recommend:

📋 PROJECT: TaskMaster Pro (Project Management SaaS)

🎯 RECOMMENDED STACK:
- Framework: Next.js App Router
- Database: Neon + Prisma
- Authentication: Better Auth
- UI Components: Shadcn UI
- Real-time: Supabase Realtime
- Background Jobs: Vercel Workflow DevKit
- AI Integration: Vercel AI SDK
- Email: Resend
- Deployment: Vercel

Now I'll set up everything for you:

1. Creating project documentation:
   ✓ /docs/prd.md (comprehensive product requirements)
   ✓ .claude/projectbrief.md (concise AI context)
   ✓ .claude/tasks.md (detailed implementation tasks)
2. Copying core memory system files
3. Installing required stack modules:
   ✓ framework-nextjs.md
   ✓ database-neon.md
   ✓ database-prisma.md
   ✓ auth-better-auth.md
   ✓ ui-shadcn.md
   ✓ database-supabase.md (for realtime)
   ✓ backend-workflow-devkit.md
   ✓ ai-vercel-ai-sdk.md
   ✓ email-resend.md
4. Configuring routing in 00-start-here.md
5. Creating initial tasks in tasks.md
6. Setting up coding standards

✅ Setup complete!

Your project now has:

📁 /docs/
- prd.md (full product requirements for humans)

📁 .claude/
- projectbrief.md (concise AI context)
- tasks.md (detailed implementation tasks from PRD)
- 00-start-here.md (routing hub)
- standards.md (coding guidelines)
- 9 stack modules for your tech stack

🚀 Ready to start building! Try:
- "Add user authentication"
- "Create a dashboard component"
- "Set up the database schema for projects and tasks"
```

---

## Scenario 2: Project Brief Already Exists

### What Happens

Claude reads the existing project brief, analyzes the requirements, and recommends the appropriate stack modules.

### Example Conversation

**User:** `setup`

**Claude:**
```
╔═══════════════════════════════════════════════════════════════════╗
║                                                                   ║
║   ░█▀▀░█░░░█▀█░█░█░█▀▄░█▀▀░░░█▄█░█▀▀░█▄█░█▀█░█▀▄░█░█             ║
║   ░█░░░█░░░█▀█░█░█░█░█░█▀▀░░░█░█░█▀▀░█░█░█░█░█▀▄░░█░             ║
║   ░▀▀▀░▀▀▀░▀░▀░▀▀▀░▀▀░░▀▀▀░░░▀░▀░▀▀▀░▀░▀░▀▀▀░▀░▀░░▀░             ║
║                                                                   ║
║              🚀  Intelligent Setup & Configuration  🚀            ║
║                                                                   ║
║   One command to initialize your entire development environment   ║
║                                                                   ║
╚═══════════════════════════════════════════════════════════════════╝

Welcome! I'll help you set up your project memory system.

I found your project brief! Let me analyze it...

📋 PROJECT: TaskMaster Pro
Type: SaaS Project Management Platform

📊 CURRENT CONFIGURATION:
✓ Project brief exists (.claude/projectbrief.md)
✓ Tech Stack defined:
  - Framework: Next.js App Router
  - Database: PostgreSQL (not specified which provider)
  - Authentication: Not specified
  - UI: "Modern component library"

📦 EXISTING .claude/ SETUP:
✗ Core system files not found
✗ No stack modules installed

🎯 RECOMMENDED MODULES:

Required Modules (based on your stack):
1. framework-nextjs.md
   Why: Your project uses Next.js App Router

2. database-neon.md + database-prisma.md
   Why: Best serverless PostgreSQL setup for Next.js
   Alternative: database-supabase.md (if you want built-in auth)

3. auth-better-auth.md
   Why: Your project brief mentions "user authentication and teams"

4. ui-shadcn.md
   Why: Modern, accessible component library that fits "modern UI"

Optional Modules (recommended for your features):
5. backend-workflow-devkit.md
   Why: Your brief mentions "scheduled tasks and notifications"

6. ai-vercel-ai-sdk.md
   Why: Your brief mentions "AI-powered task prioritization"

7. email-resend.md
   Why: Your brief mentions "email notifications"

Would you like me to:
A. Install all recommended modules (required + optional)
B. Install only required modules
C. Let me pick which modules to install
D. Show me more details about each module first

What would you prefer?
```

**User:** `A`

**Claude:**
```
Perfect! Installing all recommended modules...

📦 SETUP PROGRESS:

1. Copying core memory system...
   ✓ 00-start-here.md
   ✓ tasks.md
   ✓ standards.md
   ✓ CLAUDE.md
   ✓ SETUP.md

2. Installing stack modules...
   ✓ framework-nextjs.md
   ✓ database-neon.md
   ✓ database-prisma.md
   ✓ auth-better-auth.md
   ✓ ui-shadcn.md
   ✓ backend-workflow-devkit.md
   ✓ ai-vercel-ai-sdk.md
   ✓ email-resend.md

3. Configuring routing...
   ✓ Updated 00-start-here.md with all modules

4. Creating implementation tasks...
   ✓ Database setup (Neon + Prisma)
   ✓ Authentication integration (Better Auth)
   ✓ UI component library setup (Shadcn)
   ✓ Workflow system setup
   ✓ AI integration setup
   ✓ Email service setup

5. Validating configuration...
   ✓ All cross-references valid
   ✓ Project brief linked correctly
   ✓ Module routing complete

✅ SETUP COMPLETE!

Your project is now configured with:
- Complete memory system
- 8 stack modules
- 6 implementation tasks ready

Next steps:
1. Review tasks.md to see the implementation plan
2. Start with: "Set up the database with Neon and Prisma"
3. Or ask me to build any specific feature!

🚀 Ready to build TaskMaster Pro!
```

---

## Scenario 3: Partial Setup Exists

### What Happens

Claude detects what's already configured and suggests what's missing.

### Example Conversation

**User:** `setup`

**Claude:**
```
╔═══════════════════════════════════════════════════════════════════╗
║                                                                   ║
║   ░█▀▀░█░░░█▀█░█░█░█▀▄░█▀▀░░░█▄█░█▀▀░█▄█░█▀█░█▀▄░█░█             ║
║   ░█░░░█░░░█▀█░█░█░█░█░█▀▀░░░█░█░█▀▀░█░█░█░█░█▀▄░░█░             ║
║   ░▀▀▀░▀▀▀░▀░▀░▀▀▀░▀▀░░▀▀▀░░░▀░▀░▀▀▀░▀░▀░▀▀▀░▀░▀░░▀░             ║
║                                                                   ║
║              🚀  Intelligent Setup & Configuration  🚀            ║
║                                                                   ║
║   One command to initialize your entire development environment   ║
║                                                                   ║
╚═══════════════════════════════════════════════════════════════════╝

Welcome! I'll help you set up your project memory system.

I found your project brief and partial .claude/ setup!

📋 PROJECT: TaskMaster Pro
Type: SaaS Project Management Platform

📦 EXISTING SETUP:
✓ Project brief (.claude/projectbrief.md)
✓ Core system files present
✓ 3 stack modules installed:
  - framework-nextjs.md
  - database-prisma.md
  - ui-shadcn.md

🔍 ANALYSIS:

Based on your project brief, I notice you're missing:

1. Database Provider Module
   Current: You have Prisma (ORM) but no database provider
   Recommended: database-neon.md
   Why: Serverless PostgreSQL with branching, perfect for team workflows

2. Authentication Module
   Current: None installed
   Your brief mentions: "User authentication with team workspaces"
   Recommended: auth-better-auth.md
   Why: Full-featured auth with team/workspace support

3. Background Jobs Module
   Current: None installed
   Your brief mentions: "Scheduled notifications and task reminders"
   Recommended: backend-workflow-devkit.md
   Why: Durable workflows for reliable background tasks

4. Email Module
   Current: None installed
   Your brief mentions: "Email notifications"
   Recommended: email-resend.md
   Why: Modern email API with React Email templates

Would you like me to install these missing modules? (yes/no)
```

**User:** `yes`

**Claude:**
```
Installing missing modules...

✓ database-neon.md
✓ auth-better-auth.md
✓ backend-workflow-devkit.md
✓ email-resend.md

✓ Updated 00-start-here.md routing
✓ Created implementation tasks for new modules

✅ Your setup is now complete!

New tasks added to tasks.md:
- Set up Neon database and connect Prisma
- Integrate Better Auth with team workspaces
- Configure Workflow DevKit for background jobs
- Set up Resend for email notifications

🚀 Ready to continue building!
```

---

## Scenario 4: Full Setup Already Complete

### What Happens

Claude validates the existing setup and offers to optimize or update.

### Example Conversation

**User:** `setup`

**Claude:**
```
╔═══════════════════════════════════════════════════════════════════╗
║                                                                   ║
║   ░█▀▀░█░░░█▀█░█░█░█▀▄░█▀▀░░░█▄█░█▀▀░█▄█░█▀█░█▀▄░█░█             ║
║   ░█░░░█░░░█▀█░█░█░█░█░█▀▀░░░█░█░█▀▀░█░█░█░█░█▀▄░░█░             ║
║   ░▀▀▀░▀▀▀░▀░▀░▀▀▀░▀▀░░▀▀▀░░░▀░▀░▀▀▀░▀░▀░▀▀▀░▀░▀░░▀░             ║
║                                                                   ║
║              🚀  Intelligent Setup & Configuration  🚀            ║
║                                                                   ║
║   One command to initialize your entire development environment   ║
║                                                                   ║
╚═══════════════════════════════════════════════════════════════════╝

Welcome! I'll help you set up your project memory system.

✅ Your Claude memory system is already fully configured!

📋 PROJECT: TaskMaster Pro

✓ Project brief complete
✓ Core system files present
✓ All required stack modules installed (7 modules)
✓ Routing configured correctly
✓ Tasks tracked in tasks.md

🔍 VALIDATION RESULTS:
✓ All cross-references valid
✓ All modules up to date
✓ No missing dependencies

💡 OPTIMIZATION SUGGESTIONS:

1. Consider adding ai-openrouter.md
   Why: You're using Vercel AI SDK. OpenRouter can reduce costs
   by routing to cheaper models while maintaining quality.

2. Consider adding scraping-firecrawl.md
   Why: Your brief mentions "import tasks from other tools"
   Firecrawl can help scrape and migrate data.

Would you like to:
A. Add suggested modules
B. Update existing modules to latest versions
C. Review and update tasks.md
D. Everything looks good, I'm ready to build!
```

---

## Implementation Details for Claude

### When User Types "setup"

**CRITICAL: Suppress Initial Analysis Output**

When the user types `setup`, DO NOT output any preliminary analysis messages like:
- "Perfect! I can see this is a new project..."
- "Let me analyze your codebase..."
- "I notice you have..."

Instead, IMMEDIATELY show the ASCII header and then proceed with analysis silently.

**FIRST: Display the ASCII Header and Introduction**

Always display this header FIRST when the user types `setup` - no other text before it:

```
╔═══════════════════════════════════════════════════════════════════╗
║                                                                   ║
║   ░█▀▀░█░░░█▀█░█░█░█▀▄░█▀▀░░░█▄█░█▀▀░█▄█░█▀█░█▀▄░█░█             ║
║   ░█░░░█░░░█▀█░█░█░█░█░█▀▀░░░█░█░█▀▀░█░█░█░█░█▀▄░░█░             ║
║   ░▀▀▀░▀▀▀░▀░▀░▀▀▀░▀▀░░▀▀▀░░░▀░▀░▀▀▀░▀░▀░▀▀▀░▀░▀░░▀░             ║
║                                                                   ║
║              🚀  Intelligent Setup & Configuration  🚀            ║
║                                                                   ║
║   One command to initialize your entire development environment   ║
║                                                                   ║
╚═══════════════════════════════════════════════════════════════════╝

Welcome! I'll help you set up your project memory system.
```

**Note:** The ASCII art spells "CLAUDE MEMORY" in a stylized block font.

**THEN: Analyze Environment Silently and Present Findings**

After showing the header and welcome message, silently analyze the project environment and then present what you found.

**THEN: Detect Environment**

1. **Check Current Environment**
   ```
   Step 1: Does .claude/projectbrief.md exist?
   Step 2: Does package.json exist (codebase present)?
   Step 3: Are there application files (app/, src/, components/, etc.)?
   ```

2. **Route to Appropriate Setup Flow:**

---

### **Flow A: New Project (No Brief, No Code)**

**Detected:** Empty directory or minimal files

**Process:**
- Start tech stack questionnaire (see tech-stack-questionnaire.md and formatting rules below)
- Generate THREE documents based on answers:
  - `/docs/prd.md` - Full Product Requirements Document for humans (includes Implementation Plan)
  - `.claude/projectbrief.md` - Concise project brief for AI
  - `.claude/tasks.md` - Detailed implementation tasks (generated from PRD Implementation Plan)
- Recommend stack modules
- Copy all required files
- Initialize complete system

#### **Questionnaire Formatting Rules**

**CRITICAL:** When asking questions during setup, follow these formatting rules to avoid overwhelming users:

1. **Project Name Detection**
   - DO NOT ask for project name
   - Automatically detect from the parent folder name of `.claude/`
   - Example: If path is `/Users/user/my-project/.claude/`, project name is "my-project"
   - Present it to user: "Project name: **my-project**"

2. **One Question at a Time**
   - NEVER ask multiple questions in a single message
   - Wait for user's answer before asking the next question
   - Exception: Related sub-items within the same question are OK

3. **Question Formatting**
   - Each question number should be on its own line
   - Add a blank line between questions for readability
   - Format: `Question X: [question text]` on one line, then options/details below

4. **Example of CORRECT formatting:**
   ```
   📋 Project Basics

   Project name: **my-saas-app** (detected from folder)

   Question 1: In 2-3 sentences, what does your project do?

   [User answers]

   Question 2: Who is your target audience/users?

   [User answers]

   Question 3: What's the core problem this solves?
   ```

5. **Example of INCORRECT formatting (DO NOT DO THIS):**
   ```
   📋 Project Basics
   Question 1: What is your project name? Question 2: In 2-3 sentences, what does your project do? Question 3: Who is your target audience/users? Question 4: What's the core problem this solves?
   ```

6. **Multi-part Questions**
   - If a question has multiple parts (like selecting features), group them together
   - Use bullet points or checkboxes for clarity
   - Example:
     ```
     Question 5: What are the key features you'll need? (Select all that apply)

     - Real-time updates
     - User authentication
     - File uploads
     - Payment processing
     - Email notifications
     ```

7. **Progressive Disclosure**
   - Start with basic questions
   - Ask follow-up questions based on previous answers
   - Don't ask about features if they're not relevant to the project type

---

### **Flow B: Existing Codebase (No Brief, Code Exists)**

**Detected:** `package.json` exists, application files present, no `.claude/` folder

**Process:**

1. **Analyze Codebase**
   ```
   Scanning for:
   - Framework (Next.js, Remix, etc.)
   - Database (Prisma, Convex, Supabase, etc.)
   - UI libraries (Shadcn, Material-UI, etc.)
   - Auth system (Better Auth, NextAuth, custom)
   - Other integrations (Stripe, Resend, etc.)
   ```

2. **Present Findings**
   ```
   🔍 Detected in your codebase:
   - Framework: Next.js 14 (App Router)
   - Database: Prisma + PostgreSQL
   - Auth: Better Auth
   - UI: Tailwind CSS + Shadcn UI
   - 15 components, 8 API routes, 5 database models
   ```

3. **Ask Clarifying Questions**
   - Project name and description
   - Confirm detected stack (correct any misdetections)
   - Key features (beyond what's visible in code)
   - Deployment target
   - Special requirements

4. **Generate Documentation**
   - Create `/docs/prd.md` (based on existing code + answers)
   - Create `.claude/projectbrief.md` (reflects current implementation)
   - Create `.claude/tasks.md` (includes refactoring + new features)

5. **Install Stack Modules**
   - Copy modules matching detected stack
   - Configure routing in 00-start-here.md
   - Initialize complete system

**See:** [Existing Codebase Setup Guide](./existing-codebase-setup.md)

---

### **Flow C: Partial Setup (Brief Exists, Missing Modules)**

**Detected:** `.claude/projectbrief.md` exists, but incomplete configuration

**Process:**
- Parse project brief for:
  - Project name and description
  - Declared tech stack
  - Listed features and requirements
  - User roles and permissions
  - Special requirements
- Check existing .claude/ setup:
  - Core files present?
  - Which stack modules exist?
  - Is routing configured?
  - Are tasks tracked?
- Identify gaps:
  - Missing required modules
  - Recommended optional modules
  - Configuration updates needed
- Present recommendations
- Install missing pieces (with confirmation)
- Validate complete setup

---

### **Flow D: Complete Setup (Everything Configured)**

**Detected:** `.claude/projectbrief.md` exists, all modules present

**Process:**
- Validate configuration:
  - All referenced modules exist
  - Routing is correct
  - No broken links
- Check for updates:
  - New dependencies in package.json?
  - Stack modules need updating?
- Suggest optimizations:
  - Additional patterns to document
  - Missing best practices
  - Outdated approaches
- Confirm setup is optimal

### Two-Document Strategy

The setup process creates TWO complementary documents:

#### 1. Product Requirements Document (PRD) - `/docs/prd.md`

**Purpose:** Comprehensive product specification for humans (team, stakeholders, developers)

**Contains:**
- Detailed business goals and success metrics
- User personas and user stories
- Complete functional requirements
- Non-functional requirements (performance, security, accessibility)
- UX flows and design principles
- Database schema with full entity descriptions
- API endpoints with detailed specs
- Release plan and milestones
- Risk analysis and mitigation strategies
- Competitive analysis
- Content strategy

**When to use:**
- Onboarding new team members
- Stakeholder presentations
- Product planning and roadmaps
- Feature prioritization discussions
- Understanding the "why" behind decisions

**Template location:** `claude-memory/templates/prd-template.md`

---

#### 2. Project Brief - `.claude/projectbrief.md`

**Purpose:** Concise technical context for AI code generation

**Contains:**
- Tech stack (exact versions and configurations)
- Core features (implementation-focused)
- Simplified database schema (code-ready format)
- API endpoints (implementation specs)
- Key user stories (as acceptance criteria)
- UI components needed
- Implementation priorities (phased approach)
- Code generation guidelines
- Environment variables required
- Common patterns and utilities

**When to use:**
- Claude code generation
- Quick technical reference
- Implementation decisions
- Setting up development environment
- Creating new features

**Template location:** `claude-memory/templates/projectbrief-template.md`

---

#### 3. Tasks File - `.claude/tasks.md`

**Purpose:** Detailed, actionable implementation tasks for daily development

**Contains:**
- Current sprint tasks (in progress)
- Backlog organized by implementation phase
- Completed tasks with timestamps
- Bug tracking
- Future enhancements
- Task management guidelines

**Generated from:** PRD Section 13 (Implementation Plan)

**When to use:**
- Daily task tracking
- Sprint planning
- Progress monitoring
- Identifying blockers
- Breaking down work into subtasks

**Template location:** `claude-memory/templates/tasks-template.md`

**Relationship to PRD:**
- PRD Section 13 provides high-level implementation plan (18 major tasks)
- tasks.md breaks those down into detailed, actionable subtasks
- Each task references the related PRD section
- PRD is strategic, tasks.md is tactical

---

#### Why Three Documents?

**Different Audiences:**
- PRD → Humans (product managers, stakeholders, team)
- Project Brief → AI (code generation context)
- Tasks → Developers (daily implementation work)

**Different Purposes:**
- PRD → "What and why" (product strategy and planning)
- Project Brief → "How" (technical implementation approach)
- Tasks → "Do this now" (actionable work items)

**Different Levels:**
- PRD → Strategic (business goals, user needs, success metrics)
- Project Brief → Tactical (tech stack, patterns, guidelines)
- Tasks → Operational (specific subtasks, acceptance criteria)

**Different Timeframes:**
- PRD → Stable (updated quarterly or when strategy changes)
- Project Brief → Semi-stable (updated when tech decisions change)
- Tasks → Dynamic (updated daily as work progresses)

**Complementary, Not Redundant:**
- PRD (Section 13) defines high-level Implementation Plan
- Tasks.md breaks that plan into detailed, trackable subtasks
- Project Brief provides technical context for implementing tasks
- All three reference each other
- Single source of truth (setup command generates all three)

---

### File Operations

**Copy Core Files From:**
```
claude-memory/.claude-core/
├── 00-start-here.md
├── tasks.md
├── standards.md
├── CLAUDE.md
└── SETUP.md
```

**Copy Stack Modules From:**
```
claude-memory/stacks/
├── meta-framework/
├── database/
├── api/
├── auth/
├── ui/
├── backend/
├── ai/
├── email/
├── scraping/
├── deployment/
└── platforms/
```

**Create/Update:**
- `/docs/prd.md` (comprehensive PRD for humans with Implementation Plan)
- `.claude/projectbrief.md` (concise brief for AI)
- `.claude/tasks.md` (detailed implementation tasks generated from PRD Implementation Plan)
- `.claude/00-start-here.md` (update routing)
- `.claude/standards.md` (configure for stack)

### Validation Checks

After setup, verify:
- ✅ All referenced modules exist
- ✅ 00-start-here.md routes to all modules
- ✅ projectbrief.md is complete
- ✅ tasks.md has initial tasks
- ✅ No broken cross-references
- ✅ All placeholder text replaced

### Error Handling

**If project brief exists but is incomplete:**
```
I found your project brief, but some information is missing:

Missing:
- Tech stack not specified
- Core features not defined

Would you like me to:
A. Ask questions to complete the project brief
B. Make recommendations based on what's there
C. Start fresh with a new project brief
```

**If incompatible modules detected:**
```
⚠️ I noticed a potential issue:

Your project brief specifies:
- Framework: Next.js
- Database: Convex

But Convex has its own backend runtime that works best with
TanStack Start or vanilla React.

Recommended alternatives:
1. Next.js + Neon + Prisma (relational database)
2. Next.js + Supabase (PostgreSQL with built-in features)
3. TanStack Start + Convex (optimal Convex setup)

Would you like me to explain the tradeoffs?
```

---

## Communication Style

### Be Clear and Helpful

✅ **Good:**
```
I found your project brief for TaskMaster Pro!

Based on your requirements (team collaboration, real-time updates,
scheduled tasks), I recommend these 7 stack modules...
```

❌ **Bad:**
```
projectbrief.md detected. Analyzing dependencies...
Module resolution in progress...
```

### Show Progress Visually

✅ **Good:**
```
Installing stack modules...
✓ framework-nextjs.md
✓ database-neon.md
✓ database-prisma.md
⏳ auth-better-auth.md (copying...)
```

❌ **Bad:**
```
Processing files... please wait...
```

### Ask When Uncertain

✅ **Good:**
```
Your project brief mentions "database" but doesn't specify which one.

Based on your other requirements, I recommend:
A. Neon (serverless PostgreSQL)
B. Supabase (PostgreSQL + built-in features)
C. Convex (real-time backend)

Which fits your needs best?
```

❌ **Bad:**
```
[Automatically installs random database without asking]
```

### Explain Recommendations

✅ **Good:**
```
Recommended: auth-better-auth.md

Why: Your project brief mentions "team workspaces and permissions"
Better Auth has built-in support for organizations and role-based
access control, which maps perfectly to your requirements.

Alternative: auth-clerk.md (if you prefer a hosted solution)
```

❌ **Bad:**
```
Installing: auth-better-auth.md
```

---

## Benefits of Single "setup" Keyword

### For Users

✅ **No Decision Paralysis**
- One command for everything
- System figures out what to do
- No need to understand the internals

✅ **Works for All Scenarios**
- New projects → Guided questionnaire
- Existing projects → Intelligent analysis
- Partial setups → Gap identification

✅ **Always Gets You Started Right**
- Complete configuration
- No missing pieces
- Validated and tested

### For Claude

✅ **Clear Workflow**
- Check for brief
- Branch accordingly
- Follow systematic process

✅ **Comprehensive Context**
- Sees complete picture
- Makes informed recommendations
- Avoids incompatible choices

✅ **Smooth Handoff to Development**
- Everything configured
- Tasks ready
- Can start building immediately

---

## Example Complete Flow

**User:** `setup`

**Claude:**
```
╔═══════════════════════════════════════════════════════════════════╗
║                                                                   ║
║   ░█▀▀░█░░░█▀█░█░█░█▀▄░█▀▀░░░█▄█░█▀▀░█▄█░█▀█░█▀▄░█░█             ║
║   ░█░░░█░░░█▀█░█░█░█░█░█▀▀░░░█░█░█▀▀░█░█░█░█░█▀▄░░█░             ║
║   ░▀▀▀░▀▀▀░▀░▀░▀▀▀░▀▀░░▀▀▀░░░▀░▀░▀▀▀░▀░▀░▀▀▀░▀░▀░░▀░             ║
║                                                                   ║
║              🚀  Intelligent Setup & Configuration  🚀            ║
║                                                                   ║
║   One command to initialize your entire development environment   ║
║                                                                   ║
╚═══════════════════════════════════════════════════════════════════╝

Welcome! I'll help you set up your project memory system.
```

*[Detects no project brief]*

**Claude → User:** *[10 question questionnaire]*

**User:** *[Answers questions]*

**Claude:**
```
✅ Perfect! Setting up your project...

Creating: .claude/projectbrief.md
Installing: 8 stack modules
Configuring: routing and tasks
Validating: setup complete

✅ TaskMaster Pro is ready to build!

Start with: "Set up the database with Neon and Prisma"
```

**5 minutes later, development begins immediately.**

---

## Update Instructions for Claude

Replace all previous references to:
- "guide me"
- "help me choose"
- "what stack should I use"

With single keyword:
- **`setup`**

This command handles:
- ✅ New project questionnaire
- ✅ Existing project analysis
- ✅ Stack module installation
- ✅ Configuration validation
- ✅ Task creation
- ✅ System initialization

**One command. Complete setup. Zero confusion.**
