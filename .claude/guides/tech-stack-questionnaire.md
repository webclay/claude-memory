# Tech Stack Questionnaire Guide

**Purpose:** Interactive guide to help users discover the optimal tech stack for their project

**Trigger Keyword:** `guide me` or `help me choose`

---

## How This Works

When a user types "guide me" or "help me choose", Claude will initiate an interactive questionnaire to understand:
- Project type and goals
- Technical requirements
- Team experience
- Scale and performance needs
- Budget and time constraints

Based on responses, Claude will recommend specific stack modules from the Claude memory system.

---

## Questionnaire Flow

### 1. Project Type Discovery

**Question:** "What type of application are you building?"

**Options:**
- A. Web application (SaaS, dashboard, marketplace, etc.)
- B. Mobile application (iOS, Android, or both)
- C. API/Backend service
- D. Full-stack application (web + mobile + API)
- E. Chrome extension or browser tool
- F. WordPress plugin
- G. AI-powered application
- H. Other (user specifies)

**Follow-up based on answer:**
- If A → Ask about interactivity level
- If B → Ask about native features needed
- If C → Ask about scale and real-time needs
- If D → Ask about primary platform
- If G → Ask about AI use cases

---

### 2. Key Features & Requirements

**Question:** "What are the key features of your application?" (Multi-select)

**Options:**
- Real-time updates (chat, notifications, live data)
- User authentication and authorization
- File uploads and storage
- Payment processing
- Email notifications
- Background jobs/scheduled tasks
- Data visualization and analytics
- AI/ML integration
- Third-party API integrations
- Multi-tenancy
- Internationalization (i18n)
- SEO optimization
- Complex workflows
- Collaboration features

**Recommendations based on selections:**
- Real-time → Convex, Supabase (realtime), or WebSockets
- Auth → Better Auth
- File storage → Supabase Storage, Convex file storage
- Payments → Stripe (full control) or Polar (built-in tax compliance)
- Background jobs → Workflow DevKit, Motia
- AI/ML → Vercel AI SDK, OpenRouter
- SEO → Next.js App Router with SSR

---

### 3. Database Requirements

**Question:** "What are your data requirements?"

**Options:**
- A. Simple data structure, rapid prototyping
- B. Complex relational data with foreign keys
- C. Real-time synchronization across clients
- D. Document-based or flexible schema
- E. Time-series or analytics data
- F. Multi-tenant with data isolation
- G. Serverless/edge deployment
- H. Not sure yet

**Recommendations:**
- A → Convex (fastest setup)
- B → Prisma + Neon or Supabase
- C → Convex or Supabase (realtime subscriptions)
- D → Convex
- E → Specialized databases or Supabase
- F → Neon (database branching), Row-level security
- G → Neon (serverless driver), Supabase
- H → Prisma (flexible, easy to change later)

---

### 4. Team Experience

**Question:** "What's your team's experience level with web development?"

**Options:**
- A. Beginner (new to web development)
- B. Intermediate (comfortable with React/JS)
- C. Advanced (experienced with full-stack development)
- D. Solo developer
- E. Mixed team (various skill levels)

**Recommendations:**
- A → Next.js + Convex + Shadcn UI (minimal backend complexity)
- B → Next.js + Prisma + Better Auth
- C → Any stack, focus on specific needs
- D → Recommend batteries-included solutions (Convex, Supabase)
- E → Well-documented stacks (Next.js + Prisma)

---

### 5. Scale & Performance

**Question:** "What's your expected scale?"

**Options:**
- A. MVP/Prototype (< 100 users)
- B. Small app (100-1,000 users)
- C. Growing startup (1,000-10,000 users)
- D. Large scale (10,000+ users)
- E. Enterprise scale
- F. Not sure yet

**Recommendations:**
- A-B → Any stack, prioritize speed of development
- C → Scalable but not over-engineered (Neon, Next.js)
- D-E → Focus on scalability (Neon with connection pooling, CDN, caching)
- F → Choose flexible stack (Prisma allows DB switching)

---

### 6. AI Integration

**Question:** "Will your application use AI features?" (If yes, follow-up)

**Follow-up:** "What AI features do you need?" (Multi-select)

**Options:**
- Text generation (chatbots, content creation)
- Image generation or analysis
- Code generation
- Embeddings and semantic search (RAG)
- Streaming AI responses
- Multiple model support
- Function/tool calling
- Cost optimization across models

**Recommendations:**
- Text generation + streaming → Vercel AI SDK
- Multiple models → OpenRouter
- RAG/embeddings → Vercel AI SDK + Vector DB
- Function calling → Vercel AI SDK with tools
- Cost optimization → OpenRouter (auto-routing)

---

### 7. Deployment & Hosting

**Question:** "Where do you plan to deploy?"

**Options:**
- A. Vercel (Next.js optimized)
- B. Netlify
- C. AWS/Azure/GCP
- D. Docker/Self-hosted
- E. Edge/Serverless functions
- F. Not decided yet

**Recommendations:**
- A → Next.js + Vercel-optimized stack
- B → Next.js + Netlify Edge Functions
- C → Flexible, any stack
- D → Docker-compatible stacks
- E → Edge-compatible (Neon serverless driver)
- F → Recommend Vercel (easiest)

---

### 8. UI Requirements

**Question:** "What's your UI approach?"

**Options:**
- A. Need pre-built components (fast development)
- B. Custom design system
- C. Mobile-first responsive design
- D. Accessibility is critical
- E. Dark mode support
- F. Admin dashboard
- G. Marketing/landing pages

**Recommendations:**
- A → Shadcn UI + component libraries (Kibo UI, etc.)
- B → Shadcn UI (customizable base)
- C → Shadcn UI + Tailwind responsive utilities
- D → Shadcn UI (ARIA compliant via Radix)
- E → Shadcn UI (built-in dark mode)
- F → Shadcn UI + data table components
- G → Shadcn UI + marketing-focused blocks

---

### 9. Development Speed

**Question:** "What's your timeline?"

**Options:**
- A. Need to ship quickly (days/weeks)
- B. Standard timeline (1-3 months)
- C. Long-term project (3+ months)
- D. Ongoing product

**Recommendations:**
- A → Fastest stack: Next.js + Convex + Shadcn UI
- B → Balanced: Next.js + Prisma + Better Auth
- C-D → Focus on maintainability and scalability

---

### 10. Special Requirements

**Question:** "Any special requirements?" (Multi-select)

**Options:**
- Web scraping
- Scheduled tasks/cron jobs
- Durable workflows
- Real-time collaboration
- Multi-language support
- Offline-first
- Advanced data visualization
- Video/audio processing
- Blockchain/Web3

**Recommendations:**
- Scraping → Firecrawl
- Scheduled tasks → Workflow DevKit, Motia
- Durable workflows → Workflow DevKit
- Real-time collab → Convex, Supabase
- Multi-language → i18n patterns
- Offline-first → Specialized approaches
- Visualization → D3.js, Chart.js patterns
- Video/audio → Specialized services
- Web3 → Specialized stack

---

## Example Recommendation Output

After completing the questionnaire, provide a structured recommendation:

### 📋 Recommended Tech Stack for [Project Name]

**Based on your answers:**
- Project Type: Web Application (SaaS)
- Key Features: Auth, Real-time, Payments
- Team: Intermediate level
- Scale: Growing startup
- Timeline: 1-3 months

---

### 🎯 Core Stack

**Framework:** Next.js App Router
- **Why:** Server-side rendering, excellent DX, Vercel deployment optimization
- **Guide:** `stacks/meta-framework/framework-nextjs.md`

**Database:** Neon + Prisma
- **Why:** Serverless PostgreSQL with branching, excellent for team workflows
- **Guides:**
  - `stacks/database/database-neon.md`
  - `stacks/database/database-prisma.md`

**Authentication:** Better Auth
- **Why:** Modern auth with session management, social providers
- **Guide:** `stacks/auth/auth-better-auth.md`

**UI Components:** Shadcn UI
- **Why:** Accessible, customizable, great component ecosystem
- **Guide:** `stacks/ui/ui-shadcn.md`
- **Additional Resources:** Kibo UI, Shadcn Studio for pre-built blocks

---

### 🔌 Additional Services

**Real-time Updates:** Supabase Realtime
- **Guide:** `stacks/database/database-supabase.md`

**AI Features:** Vercel AI SDK
- **Guide:** `stacks/ai/ai-vercel-ai-sdk.md`

**Background Jobs:** Vercel Workflow DevKit
- **Guide:** `stacks/backend/backend-workflow-devkit.md`

---

### 🚀 Getting Started Order

1. **Set up Next.js project**
   ```bash
   npx create-next-app@latest my-app
   ```

2. **Initialize Shadcn UI**
   - Follow guide: `stacks/ui/ui-shadcn.md`

3. **Set up database**
   - Create Neon project
   - Initialize Prisma
   - Follow guides in `stacks/database/`

4. **Add authentication**
   - Install Better Auth
   - Configure providers
   - Follow guide: `stacks/auth/auth-better-auth.md`

5. **Add real-time features**
   - Set up Supabase Realtime subscriptions
   - Follow guide: `stacks/database/database-supabase.md`

---

### 📚 Learning Path

**Week 1-2:** Core setup
- Next.js basics
- Database schema design
- Basic auth flow

**Week 3-4:** Feature development
- Build main features
- Add real-time updates
- UI polish with Shadcn

**Week 5-6:** Advanced features
- Background jobs
- AI integration
- Testing and optimization

---

### 💡 Alternative Approaches

**For Faster MVP:**
- Replace Neon + Prisma with Convex (all-in-one backend)
- Reduces setup complexity significantly

**For Lower Costs:**
- Use Supabase (free tier) instead of Neon
- Generous free limits for early projects

**For AI-Heavy App:**
- Add OpenRouter for multi-model support
- Lower costs compared to single provider

---

### ⚠️ Common Pitfalls to Avoid

1. **Don't over-engineer early** - Start simple, add complexity as needed
2. **Set up auth early** - Retrofitting auth is painful
3. **Database migrations** - Use Prisma migrations from day 1
4. **Environment variables** - Use `.env.local`, never commit secrets
5. **Type safety** - Let TypeScript catch errors early

---

### 🎓 Next Steps

1. Review the recommended stack guides
2. Set up your development environment
3. Create your first prototype
4. Iterate based on user feedback

**Need help?** Come back anytime and ask specific questions about any part of the stack!

---

## Claude Instructions for Using This Guide

When a user types **"guide me"**, **"help me choose"**, or similar phrases:

1. **Acknowledge the request:**
   ```
   "I'll help you find the perfect tech stack for your project!
   Let me ask you a few questions to understand your needs."
   ```

2. **Ask questions ONE AT A TIME** - Don't overwhelm with all questions at once

3. **Be conversational** - Adapt questions based on previous answers

4. **Provide context** - Briefly explain why you're asking each question

5. **Allow flexibility** - Let users skip or say "not sure" for any question

6. **Progressive disclosure** - Only ask relevant follow-ups

7. **After gathering info** - Provide a comprehensive recommendation using the format above

8. **Be adaptive** - If user has specific constraints, adjust recommendations

9. **Offer alternatives** - Show 2-3 options when multiple stacks could work

10. **End with action items** - Give clear next steps

---

## Example Conversation Flow

**User:** "guide me"

**Claude:** "I'll help you find the perfect tech stack for your project! Let's start with the basics.

What type of application are you building?
A. Web application (SaaS, dashboard, marketplace, etc.)
B. Mobile application (iOS, Android, or both)
C. API/Backend service
D. Full-stack application (web + mobile + API)
E. AI-powered application
F. Other

Just reply with the letter or describe in your own words!"

**User:** "A web application, it's going to be a SaaS for project management"

**Claude:** "Great! A project management SaaS. That helps narrow things down.

What are the key features you'll need? (You can select multiple)
- Real-time updates (for team collaboration)
- User authentication (individual users and teams)
- File uploads (documents, images)
- Background tasks (notifications, scheduled reports)
- Data visualization (charts, analytics)
- Other features you have in mind?"

**[Continue conversation...]**

After 5-7 questions, provide the full recommendation.

---

## Notes

- This guide should be updated as new stack modules are added
- Recommendations should reflect current best practices
- Consider user's constraints (budget, timeline, team size)
- Always provide paths for both beginners and advanced users
- Link to specific guide files for implementation details
