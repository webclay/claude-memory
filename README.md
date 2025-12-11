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
```

> A smart helper that teaches Claude Code exactly how to build your app, so you never have to explain the same thing twice.

**Version:** 1.3.0
**Last Updated:** 2025-12-11
**Author:** Manuel Merz
**License:** MIT

---

## Changelog

### Version 1.3.0 - December 11, 2025

**TanStack Start Deep Dive & New Stack Modules:**

This release significantly expands TanStack Start documentation based on the official 1-hour tutorial, plus adds new stack modules for modern tooling.

#### TanStack Start Documentation Overhaul

| Addition | Description |
|----------|-------------|
| **Execution Model** | Critical section explaining loaders run on BOTH client AND server (unlike Next.js RSC) |
| **Four Server Function Types** | `createServerFn`, `createServerOnlyFn`, `createClientOnlyFn`, `createIsomorphicFn` |
| **`useServerFn` Hook** | Required for handling redirects from server functions |
| **Data Mutations** | `router.invalidate()` and `useOptimistic` patterns |
| **Prisma/Better Auth Gotcha** | When to use API routes instead of server functions |
| **Troubleshooting Section** | Common issues and solutions |

#### New Stack Modules

| Module | Description |
|--------|-------------|
| **Durable Streams** | Resumable HTTP streams for AI token streaming that survives network failures |
| **Bun** | All-in-one JavaScript runtime, package manager, bundler (Bun 1.3+ with text-based lockfile) |
| **Appwrite** | Open-source backend platform (authentication, databases, storage, functions) |

---

### Version 1.2.0 - December 2, 2025

**CLAUDE.md Optimization - Based on Best Practices:**

Following Kyle Mistele's research on effective CLAUDE.md files, this release dramatically improves how Claude reads and follows project instructions.

#### Key Changes

| Change | Before | After |
|--------|--------|-------|
| **CLAUDE.md size** | 510 lines | 65 lines |
| **Commands** | Keyword-based | Native slash commands `/project:*` |
| **Code style rules** | 120+ lines inline | Single line: "run ultracite" |
| **Task management** | Pre-generated tasks | Dynamic creation with Feature Dependencies |

#### New Native Slash Commands

Type `/project:` to see all available commands:

| Command | Description |
|---------|-------------|
| `/project:setup` | Initialize or configure project |
| `/project:status` | Show current task progress |
| `/project:next` | Get suggestion for next task |
| `/project:done` | Mark current task as complete |
| `/project:terminate` | Kill all dev servers for clean restart |
| `/project:blockers` | List blocked tasks and dependencies |
| `/project:help` | Show all available commands |

#### New Files

- **`.claude/mistakes.md`** - Tracks common AI mistakes to avoid (auto-updated when patterns emerge)
- **`.claude/commands/`** - Native slash command definitions

#### Feature Dependencies System

The projectbrief template now includes a **Feature Dependencies** section that maps how features connect. When you work on one feature, Claude checks this map and warns you if changes might impact other areas.

```markdown
## Feature Dependencies
- **Database Schema** → All features (foundational)
- **Authentication** → Dashboard, User Settings, Protected Routes
- **Payments** → User Settings, Subscription Pages
```

#### Why This Matters

From Anthropic's research: Claude ignores CLAUDE.md content it deems irrelevant. Fewer, more focused instructions = better instruction following.

- ✅ Progressive disclosure - Claude reads specialist files only when needed
- ✅ No content duplication - single source of truth for each topic
- ✅ Impact awareness - warns about side effects before making changes
- ✅ Native slash commands - proper IDE integration

---

### Version 1.1.0 - November 25, 2025

**Stack Updates - Latest Library Versions:**

#### Better Auth v1.4
- ✨ **Stateless Authentication** - Database-optional session management with JWT-only mode
- ✨ **Custom OAuth State** - Pass referral codes and attribution data through OAuth flows
- ✨ **Device Authorization Plugin** - OAuth 2.0 Device Grant for smart TVs and IoT devices
- ✨ **SCIM Provisioning Plugin** - Enterprise identity management for multi-domain scenarios
- ⚡ **Performance Improvements** - Experimental database join optimization (2-3x faster)
- 🔐 **Enhanced Security** - JWE cookie caching, JWT key rotation, improved email change security
- 🔧 **Cookie Chunking** - Automatic handling to prevent size errors
- 🆔 **UUID Support** - Primary key support for UUID alongside nanoid
- 📦 **Bundle Optimization** - Minimal build option for custom database adapters
- ⚠️ **Breaking Changes** - Passkey plugin moved to `@better-auth/passkey`, API naming updates
- 📚 Full documentation: `.claude/stacks/auth/auth-better-auth.md`

#### Prisma ORM v7.0
- 🚀 **3x Faster Query Execution** - Rust-free client rebuilt in TypeScript
- 📦 **90% Smaller Bundle Size** - Significantly reduced client footprint
- ⚡ **70% Faster Type Checking** - Optimized type system (98% fewer types to evaluate)
- 🔧 **Edge-Optimized** - Better compatibility with Vercel Edge and Cloudflare Workers
- 🗄️ **Prisma Postgres** - One-command database provisioning (`npm create db`)
- 📝 **Prisma Config File** - Dynamic configuration with JavaScript/TypeScript support
- 🎨 **Mapped Enums** - Long-requested feature for enum value mapping
- 🤖 **MCP Server Integration** - AI agent support (local & remote) for schema assistance
- ⚠️ **Breaking Changes** - Provider syntax changed from `"prisma-client-js"` to `"prisma-client"`
- 📍 **Generated Code Location** - Now defaults to source directory instead of `node_modules`
- 📚 Full documentation: `.claude/stacks/database/database-prisma.md`

**Key Highlights:**
- Both libraries now offer significantly improved performance for serverless and edge environments
- Enhanced AI integration capabilities (Prisma MCP, Better Auth stateless mode)
- Better developer experience with improved type systems and bundle sizes
- Enterprise-ready features (SCIM provisioning, advanced security options)

---

### Version 1.0.0 - January 12, 2025

**Initial Release:**
- 🎉 Claude Memory system launched
- 📋 Intelligent `setup` command for project initialization
- 📚 Comprehensive stack documentation for 40+ technologies
- ✅ Automatic task tracking and progress management
- 🔄 Pattern consistency enforcement
- 📖 Interactive questionnaire for new projects
- 🔍 Automatic codebase detection for existing projects
- 💾 Two-document system (PRD for humans, projectbrief for AI)
- 🛠️ Support for major frameworks, databases, auth systems, and more

---

## What is this?

Think of this as **giving Claude Code a perfect memory of your project**. You tell Claude what you're building once, and it remembers everything - what tools you're using, how you want things built, and what you're working on right now.

### The Problem It Solves

**Without Claude Memory:**
- ❌ You have to re-explain your app idea every chat session
- ❌ Claude forgets what database or tools you're using
- ❌ You spend time correcting mistakes instead of building
- ❌ Each new feature feels like starting from scratch
- ❌ You're unsure what's been done and what's left to do

**With Claude Memory:**
- ✅ Tell Claude your idea once, it remembers forever
- ✅ Claude knows exactly what tools you're using
- ✅ Everything stays consistent - no random changes
- ✅ Clear progress tracking - see what's done and what's next
- ✅ Just chat naturally and watch your app come to life

---

## How It Works

Claude Memory creates a hidden `.claude/` folder in your project that contains:

```
.claude/
├── projectbrief.md            → What you're building
│                                Your app idea, features, and tools
│
├── tasks.md                   → What needs to be done
│                                Auto-updated as you build
│
├── mistakes.md                → Common AI mistakes to avoid
│                                Auto-updated when patterns emerge
│
├── commands/                  → Native slash commands
│   ├── setup.md               → /project:setup
│   ├── status.md              → /project:status
│   ├── done.md                → /project:done
│   └── ...                    → And more
│
├── Guides for specific tools:
│   ├── database-*.md          → How to work with your database
│   ├── auth-*.md              → How to handle user login
│   ├── payments-*.md          → How to process payments
│   ├── ai-*.md                → How to add AI features
│   └── ui-*.md                → How to build your interface
```

**Here's the magic:**

When you chat with Claude, it automatically reads these files and knows exactly:
- What app you're building
- What tools you're using
- How everything should work together
- What you're working on right now

It's like having a smart assistant who never forgets anything about your project.

---

## Quick Start

### ⚡ Just Type One Word: `setup`

That's it! Claude will:

- 🆕 **Starting fresh?** → Asks you friendly questions about what you want to build
- 💻 **Already have code?** → Figures out what you're using and documents it automatically
- 📋 **Have an idea written down?** → Reads it and sets up everything you need
- 🔧 **Partially set up?** → Finds what's missing and completes it for you

**No technical knowledge needed. No confusing steps. Just chat with Claude.**

---

### Step 1: Get Claude Memory Into Your Project

1. **Download** Claude Memory (click the green "Code" button → Download ZIP)
2. **Unzip** the file you just downloaded
3. **Copy** the `.claude` folder into where your app lives
4. **Open** Claude Code and navigate to your project

That's it! Your project now has:
```
your-project/
├── .claude/              ← The smart memory system
├── your-app-files/       ← Your app (stays exactly the same)
└── package.json          ← Your existing files
```

---

### Step 2: Make the `.claude` Folder Visible (Important!)

The `.claude` folder has a dot at the start, so your computer hides it by default. Here's how to see it:

**On Mac:**
- Open Finder, then press these three keys together: `Cmd + Shift + .` (the period key)
- The `.claude` folder will appear!

**On Windows:**
- Open File Explorer
- Click the "View" tab at the top
- Check the box that says "Hidden items"

**In Claude Code:**
- Good news! Claude Code shows hidden folders automatically
- You should see `.claude` in your file list on the left

---

### Step 3: Chat with Claude

In Claude Code, just type:

```
setup
```

**That's literally it!** Claude will have a friendly conversation with you:

**If you're starting fresh:**
- Claude asks simple questions like "What do you want to build?"
- You describe your app in plain English
- Claude sets up everything automatically
- You get a list of features to build, one by one

**If you already have code:**
- Claude looks at your existing app
- Figures out what you're already using
- Documents everything so it remembers
- You're ready to keep building!

**Here's what it looks like:**

```
Claude: "I see you want to build a task management app!

I'll set you up with:
- A modern web framework (Next.js)
- A database for your tasks (Neon + Prisma)
- User login system (Better Auth)
- Nice looking components (Shadcn UI)

Sound good?"

You: "Yes!"

Claude: "✅ All set! Ready to build your app.
Try saying: 'Let's create the login page'"
```

---

### Step 4: Start Building!

Now just chat naturally with Claude to build your app:

```
"Add a login page"
"Create a dashboard"
"Let users create tasks"
"Add a payment page"
```

**Claude will:**
- ✅ Build exactly what you ask for
- ✅ Keep everything working together nicely
- ✅ Track what's done and what's left to do
- ✅ Never forget what you're building
- ✅ Make consistent choices every time

---

## How Claude Uses Your Memory

### Claude Reads Everything Automatically

When you ask Claude to build something, it **automatically reads** all your memory files:

```
You: "Add a settings page"

Claude (automatically):
1. Reads projectbrief.md → "This app uses Shadcn UI"
2. Reads ui-shadcn.md → "Here's the design system"
3. Reads existing pages → "Here's the layout pattern"
4. Builds the settings page matching everything
```

**You don't need to mention the files!** Claude reads them on its own.

---

### When Claude Doesn't Follow Your Patterns

Sometimes Claude might forget or miss a pattern. Here's how to fix it:

**❌ Problem: New page doesn't match existing layout**

```
You: "The settings page doesn't have the same layout as the dashboard"

Me: "You're right! Let me check the dashboard layout and rebuild the settings page to match."
```

**❌ Problem: Wrong spacing or design**

```
You: "The spacing is different from other pages"

Me: "Sorry about that! I'll fix it to match the design system in ui-shadcn.md"
```

**❌ Problem: Not using the right database patterns**

```
You: "Use the same database pattern as the user table"

Me: "Got it! I'll check how the user table is structured and match that pattern."
```

### How to Keep Everything Consistent

**1. Remind Claude to check existing code:**
```
"Make this match the dashboard page"
"Use the same pattern as the login form"
"Follow the design system we're using"
```

**2. Point out specific examples:**
```
"Look at how the user profile page is structured"
"Use the same spacing as the other cards"
"Match the button style from the homepage"
```

**3. If something keeps breaking, tell Claude:**
```
"Every new page needs to follow the same layout - can you check existing pages first?"
```

**Claude will then:**
- Read your existing code
- Match the patterns exactly
- Keep everything consistent

---

### Pro Tip: Pattern Reinforcement

If Claude keeps forgetting a specific pattern, you can say:

```
"When creating new pages, always:
1. Check existing page layouts first
2. Use the same Shadcn components
3. Match the spacing and structure
Can you add this to the memory?"
```

Claude will remember this for future pages!

---

## Already Have an App Started?

**No problem!** Claude Memory works great with apps you've already started building.

### What Happens

1. **Copy the `.claude` folder** into your existing project
2. **Type `setup`** in Claude Code
3. **Claude looks at your code** and figures out what you're already using
4. **Claude tells you what it found:**
   ```
   "I can see your app is using:
   - Next.js for the website
   - Prisma + PostgreSQL for the database
   - Better Auth for user logins
   - Shadcn UI for the design"
   ```
5. **Claude creates memory files** to remember all of this

### Your Code Stays Exactly the Same

```
your-existing-project/
├── .claude/              ← NEW: Claude's memory files
│   ├── projectbrief.md
│   ├── framework-nextjs.md
│   ├── database-prisma.md
│   └── auth-better-auth.md
│
├── app/                  ← UNCHANGED: Your code
├── components/           ← UNCHANGED: Your code
├── lib/                  ← UNCHANGED: Your code
└── package.json          ← UNCHANGED
```

**Your app code doesn't change at all!** Claude just learns what you have.

---

## What Tools Can Claude Help You Build With?

### Website Frameworks

These are the foundations for building modern websites and apps.

| Framework | What It Does |
|-----------|--------------|
| **Next.js** | The most popular choice - builds fast, modern websites with React |
| **TanStack Start** | Great for interactive apps with smooth navigation |
| **Remix** | Perfect for content-heavy sites with great performance |
| **Astro** | Best for blogs and marketing sites that need to be super fast |
| **SvelteKit** | Lightweight alternative that's easy to learn |

---

### Databases

Where your app stores all its data (users, posts, products, etc.).

| Database | What It Does |
|----------|--------------|
| **Supabase** | Complete backend - database, user logins, file storage, all in one |
| **Convex** | Real-time database - perfect for chat apps, collaborative tools, live updates |
| **Neon** | Modern database that scales automatically as your app grows |
| **Prisma** | Helps you work with databases safely and easily |
| **Drizzle** | Lightweight, fast alternative to Prisma |
| **Firebase** | Google's database - great for mobile apps |

---

### Backend & Server Communication

How your app talks to the database and handles user requests.

| Technology | What It Does |
|------------|--------------|
| **oRPC** | Modern way to connect your frontend to backend - type-safe and reliable |
| **Next.js Server Actions** | Built into Next.js - easiest way to handle forms and data |
| **Convex Functions** | Built into Convex - handles all your backend logic |
| **tRPC** | Popular choice for connecting frontend to backend safely |
| **Express** | Classic approach - works with any setup |

---

### Advanced Backend Tools

For complex apps that need scheduled jobs, workflows, or AI features.

| Technology | What It Does |
|------------|--------------|
| **Motia** | Complete backend platform - handles APIs, scheduled tasks, and AI agents |
| **Workflow DevKit** | Run long-running tasks reliably (payments, email sequences, etc.) |

---

### Where to Host Your App

These services put your app on the internet so people can use it.

| Platform | What It Does |
|----------|--------------|
| **Vercel** | The easiest way to deploy - works perfectly with Next.js |
| **Netlify** | Great alternative with similar features |
| **Cloudflare** | Super fast global hosting |
| **AWS** | Amazon's cloud - very powerful but more complex |

---

### User Accounts & Login

How users sign up, log in, and stay logged in.

| Service | What It Does |
|---------|--------------|
| **Better Auth** | Modern login system with email, Google, GitHub, etc. |
| **Supabase Auth** | Built into Supabase - super easy to set up |
| **Clerk** | Fancy login with pre-built UI components |
| **NextAuth.js** | Popular choice for Next.js apps |

---

### AI Features

Add AI capabilities to your app (chatbots, content generation, etc.).

| Service | What It Does |
|---------|--------------|
| **Vercel AI SDK** | The easiest way to add AI chat to your app |
| **OpenRouter** | Access 100+ different AI models with one setup |
| **OpenAI** | Connect directly to ChatGPT's technology |
| **Anthropic** | Connect directly to Claude AI |

---

### Sending Emails

Send emails from your app (welcome emails, notifications, receipts, etc.).

| Service | What It Does |
|---------|--------------|
| **Resend** | Modern email service - super easy to use |
| **SendGrid** | Trusted email platform used by major companies |

---

### Accepting Payments

Let users pay you for products, subscriptions, or services.

| Service | What It Does |
|---------|--------------|
| **Stripe** | The most popular payment processor - handles everything |
| **Polar** | Open-source alternative with built-in tax handling |

---

### Design & UI Components

Pre-made buttons, forms, modals, and other interface elements.

| Library | What It Does |
|---------|--------------|
| **Shadcn UI** | Beautiful, customizable components - most popular choice |
| **Untitled UI** | 5,000+ premium components for professional apps |
| **Kibo UI** | Enhanced version of Shadcn with extra features |
| **Tailwind CSS** | Makes styling your app super fast and easy |

---

### Web Scraping

Extract data from other websites for your app.

| Service | What It Does |
|---------|--------------|
| **Firecrawl** | Scrape websites reliably, even complex ones |

---

### Special App Types

Build browser extensions, mobile apps, or WordPress plugins.

| Platform | What It Does |
|----------|--------------|
| **WordPress Plugins** | Build plugins for WordPress sites |
| **Chrome Extensions** | Create browser extensions for Chrome |
| **React Native** | Build real iOS and Android apps |
| **Expo** | Easier way to build mobile apps with React Native |

---

### Code Quality Tools

Keep your code clean and consistent automatically.

| Tool | What It Does |
|------|-------------|--------------|
| **Ultracite** | Automatically formats and fixes your code |

---

## Using Claude Memory Every Day

### How to Use Claude Memory Effectively

**The Golden Rule: One Chat = One Task**

This is the most important tip for getting great results:

✅ **Good: Focused sessions**
```
Chat 1: "Create the user profile page"
→ Builds the entire profile page, matches design system, done!

Chat 2: "Set up the database schema for user profiles"
→ Focuses on database, follows patterns, done!

Chat 3: "Add edit functionality to the profile page"
→ Builds on existing work, stays consistent!
```

❌ **Less effective: Mixing unrelated tasks**
```
"Create the user profile page, then set up the database schema,
and also add the settings page"
→ Context gets mixed up, patterns might get inconsistent
```

**Why this works better:**
- ✅ Claude stays focused on one area (frontend OR backend OR database)
- ✅ Design patterns stay consistent across your app
- ✅ Easier to track what's done
- ✅ Better results, fewer mistakes

**Exception:** Related subtasks in one chat are totally fine!
```
"Create the login page with the form, validation, and error handling"
→ This is all one cohesive task ✅
```

---

### More Workflow Tips

**1. Start each session with what you want, not how to do it**

✅ Good:
- "Add a dark mode toggle"
- "Let users upload profile pictures"
- "Build a pricing page"

❌ Less effective:
- "Use useState and create a button component with Tailwind classes..."
- "Import Shadcn components and configure the theme provider..."

→ Let Claude figure out the "how" based on your memory!

**2. When something doesn't match your style:**

If Claude builds something that doesn't match your existing design:

```
"This looks different from my other pages. Can you match the spacing
and layout from the dashboard page?"
```

→ Claude will read the dashboard, see the pattern, and rebuild to match!

**3. Build iteratively, not all at once**

✅ Good progression:
1. "Create a basic blog post page"
2. "Add comments to the blog post"
3. "Add like/share buttons to posts"

❌ Overwhelming:
1. "Build the entire blog with posts, comments, likes, shares, categories, tags, and search"

**4. Let Claude know about changes to your plan**

If you decide to change direction:

```
"Actually, let's use Stripe instead of Paddle for payments.
Can you update the projectbrief?"
```

→ Claude updates memory and uses the new approach going forward!

**5. Use slash commands for common tasks**

- `/project:status` → See current task progress
- `/project:next` → Get suggestion for next task
- `/project:done` → Mark current task complete
- `/project:terminate` → Kill all dev servers
- `/project:help` → See all available commands

**6. When stuck, ask for options**

```
"I need user authentication. What are my options?"

Claude will suggest:
- Better Auth (recommended for most apps)
- Clerk (great for quick setup)
- Supabase Auth (if using Supabase)
- NextAuth (if you prefer it)

And explain trade-offs!
```

**7. New project? Always run `setup` first**

Don't start coding right away. Run:
```
setup
```

This creates your project memory and saves you hours of back-and-forth!

---

### When You Start Building

Just type `setup` once and you're ready. Claude will:
- Learn what you want to build
- Set up all the tools you need
- Create a list of features to build
- You're ready to go in minutes!

---

### Every Time You Chat with Claude

Claude automatically remembers everything. Just talk naturally:

```
"Add a profile page"
"Build the shopping cart"
"Let users upload photos"
```

---

### While Building

**Claude automatically:**
- ✅ Uses the right tools for your project
- ✅ Makes everything look consistent
- ✅ Handles errors gracefully
- ✅ Keeps everything working together

**You enjoy:**
- ✅ Building faster
- ✅ Everything just works
- ✅ No confusing inconsistencies
- ✅ Focus on your ideas, not technical details

---

### Saving Your Progress

When you're ready to save your work:

```
"Save this work"
```

**Claude will:**
- ✅ Save what changed
- ✅ Track what's been completed
- ✅ Write a description of what was done
- ✅ Keep your to-do list updated

---

### Checking What's Done

Open `tasks.md` anytime to see:
- ✅ What features are finished
- ⏳ What you're working on now
- ⏸️ What's coming next
- 📝 Notes about how things work

---

## Why Vibe Coders Love This

✅ **Never Start From Scratch**
- Claude remembers everything, even if you take a break for weeks
- Pick up right where you left off

✅ **Build Way Faster**
- No explaining the same thing over and over
- Claude already knows what you're building

✅ **Everything Stays Organized**
- Clear list of what's done and what's next
- Never lose track of your progress

✅ **Come Back Anytime**
- Take a month off? No problem!
- All your project details are saved and ready

✅ **Actually Ship Your App**
- Less time explaining, more time building
- Finally finish that app idea you've had

---

## How It Actually Works

### The Old Way (Frustrating)

```
You: "Add user login"
Claude: "Okay!" *makes it one way*

*Next day*

You: "Add a settings page"
Claude: "Okay!" *makes it a completely different way*

You: "Wait, why is this different??"
```

### With Claude Memory (Amazing)

```
You: "Add user login"
Claude: "Okay! Using Better Auth..." *saves this choice*

*Next day*

You: "Add a settings page"
Claude: "Got it! Using Better Auth like before..." *stays consistent*

You: "Perfect!"
```

**The secret:** Claude writes down its choices and reads them every time.

---

## Can I Mix and Match Tools?

Absolutely! Combine whatever works for your app:

**Popular Combinations:**
- ✅ Next.js + Convex + Better Auth + Resend
- ✅ Next.js + Supabase (database + auth + storage all-in-one)
- ✅ Next.js + Neon + Prisma + Stripe + Vercel AI SDK

Claude knows how to make them all work together!

---

## Common Questions

**Q: How do I pick which tools to use?**
A: Just type `setup` and Claude will recommend the best tools for your app. You can say yes or pick different ones.

**Q: Do I need to use all these tools?**
A: Nope! Claude only sets up what you actually need for your specific app.

**Q: Can I use this with an app I already started?**
A: Yes! Claude will look at your existing code and learn from it.

**Q: What if I want to switch tools later?**
A: Just tell Claude you want to change, run `setup` again, and it'll help you switch.

**Q: How much time does this take to set up?**
A: About 5 minutes of chatting with Claude. Then you're ready to build!

**Q: Will this mess up my existing code?**
A: Never! Claude Memory only adds memory files - your app code stays exactly the same.

**Q: Can I change the memory files?**
A: You can, but usually Claude handles everything automatically for you.

---

## Something Not Working?

### Claude seems confused

**Try this:**
Just type `setup` again. Claude will check everything and fix any issues.

---

### Progress tracking isn't updating

**Try this:**
Ask Claude: "Update my task list with what we just built"

---

### Setup didn't finish

**Try this:**
Type `setup` again and Claude will pick up where it left off.

---

### New features don't match existing style

**This is the most common issue!** Here's how to fix it:

**Problem:** New page/component looks different from existing ones

**Solution:**
```
"The new settings page doesn't match the dashboard layout.
Please check the dashboard page and rebuild the settings page to match it exactly."
```

**Prevention:**
```
"Before building the settings page, check how existing pages are structured
and match that pattern exactly."
```

**For recurring issues:**
```
"Every time you create a new page:
1. Check 2-3 existing pages first
2. Match the layout, spacing, and components exactly
3. Ask me if you're unsure about any pattern

Can you remember this?"
```

Claude will add this as a rule and follow it for all future work!

---

## Keeping Claude Memory Updated

### Get the Latest Features

When a new version comes out, just type: `update`

```
You: update

Me: "Checking for updates...

✅ New version available: v1.2.0

What's new:
- Better framework support
- New payment features
- Improved error messages

How would you like to update?

A. Automatic (I'll download it from GitHub)
B. Manual (You downloaded it, I'll guide you)

Your choice?"

You: "A"

Me: "Updating... ✅ Done! Now on v1.2.0"
```

**Don't worry!** Updates never touch:
- Your project details (projectbrief.md)
- Your progress tracking (tasks.md)
- Your app code

Everything stays safe!

---

## Need Help?

**First time here?** Start with the Quick Start section at the top!

**Something not working?** Check the "Something Not Working?" section above.

**Have questions?** Check the "Common Questions" section.

**Still stuck?** Open an issue on GitHub and we'll help you out!

---

## What's Coming Next

We're constantly adding support for more tools and making Claude Memory even easier to use. Have a tool you want supported? Let us know!

---

**Ready to start building? 🚀**

Just copy the `.claude` folder to your project, type `setup`, and watch your app come to life!
