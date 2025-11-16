# [Project Name] - What You're Working On

> **How this works:** Claude automatically updates this file as you build. It tracks what's done, what's next, and what's left to do.

**Created from:** `/docs/prd.md` (your planning document)
**Last Updated:** [Claude updates this automatically]

---

## Quick Jump

- [🚧 Working on Now](#-working-on-now)
- [📋 Up Next](#-up-next)
- [✅ Done](#-done)
- [🐛 Bugs to Fix](#-bugs-to-fix)
- [💡 Future Ideas](#-future-ideas)

---

## 🚧 Working on Now

**This Week's Focus:** [What you're building - e.g., "Getting the login system working"]

**Timeline:** [When you want it done]

### Currently Building

**Task 1: Get the Project Set Up**
- Status: 🚧 Working on it (Started: [Date])
- Who's doing it: [Your name]
- How important: Must do first
- Waiting on: Nothing

**What needs to happen:**
- [ ] Create the basic app
  - Start a new project
  - Set up the configuration
  - Organize the folders
- [ ] Set up the database
  - Sign up for database hosting
  - Create the database
  - Connect it to the app
- [ ] Get the database tools working
  - Install what's needed
  - Set it up
  - Make sure it connects
  - Test it works
- [ ] Add the design system
  - Install the UI components
  - Set up the theme
  - Test with a button or card
- [ ] Set up secrets and passwords
  - Create the file for secrets
  - Make sure it's private
  - Write down what secrets you need
  - Add them to hosting platform

**Notes:**
- Nothing blocking this
- From planning doc: Section 13, Phase 1, Task 1

---

## 📋 Up Next

### Phase 1: Getting Started (Weeks 1-2)

---

**Task 2: Build Login and Sign Up**
- Status: 📋 Not started yet
- Who's doing it: Nobody yet
- How important: Very important
- Can't start until: Task 1 is done

**What needs to happen:**
- [ ] Set up the login system
  - Install the login tools
  - Configure it
  - Add email/password login
  - Add Google/GitHub login
  - Add the secret keys
- [ ] Make the login page
  - Create the /login page
  - Build the login form
  - Make sure emails and passwords are valid
  - Connect it to the login system
  - Show errors if login fails
  - Test it works
- [ ] Make the sign up page
  - Create the /signup page
  - Build the signup form
  - Check passwords are strong enough
  - Connect it to the signup system
  - Test it works
- [ ] Add "forgot password"
  - Create the forgot password page
  - Send reset emails
  - Create the reset password page
  - Let users set new password
  - Test it works
- [ ] Keep users logged in
  - Save their login session
  - Auto-logout after a while
  - Let them stay logged in
  - Add logout button
- [ ] Verify email addresses (optional)
  - Send verification emails
  - Create the verification page
  - Check the verification link works
- [ ] Test everything
  - Try logging in
  - Try signing up
  - Try forgot password
  - Make sure it all works

**How you'll know it's done:**
- People can sign up
- People can log in
- People can log in with Google/GitHub
- People can reset their password
- People stay logged in
- People can log out
- Email verification works (if you're using it)

**Notes:**
- From planning doc: Section 13, Phase 1, Task 2
- Designs: [Link to how it should look]

---

**[TASK-003] Create base application layout**
- Status: 📋 Pending
- Owner: Unassigned
- Priority: High
- Dependencies: TASK-002

**Subtasks:**
- [ ] Build main navigation component
  - Create Header component with logo
  - Add navigation links (Dashboard, Profile, Settings, etc.)
  - Implement responsive mobile menu (hamburger)
  - Add user avatar/menu dropdown
  - Style with [UI Component Library]
- [ ] Create sidebar navigation (if applicable)
  - Build Sidebar component
  - Add navigation items with icons
  - Implement active state highlighting
  - Make collapsible for mobile
  - Test responsive behavior
- [ ] Create page layouts
  - Build AuthLayout (for login/signup pages)
  - Build DashboardLayout (for authenticated pages)
  - Build PublicLayout (for landing/marketing pages)
  - Add proper semantic HTML structure
  - Test layout responsiveness
- [ ] Implement responsive design foundation
  - Set up mobile-first breakpoints
  - Test on mobile, tablet, desktop viewports
  - Ensure touch-friendly interactive elements
  - Test navigation on all screen sizes
- [ ] Add dark mode support (if required)
  - Set up theme provider
  - Create theme toggle component
  - Add dark mode styles to all components
  - Persist theme preference
  - Test dark mode across all pages

**Acceptance Criteria:**
- Navigation works on all screen sizes
- Layouts properly contain page content
- Active navigation states are highlighted
- Responsive design works on mobile, tablet, desktop
- Dark mode toggle works (if required)

**Notes:**
- Related PRD Section: Section 13, Phase 1, Task 3
- Design system: [Link to design tokens]

---

**[TASK-004] Set up database schema**
- Status: 📋 Pending
- Owner: Unassigned
- Priority: High
- Dependencies: TASK-001

**Subtasks:**
- [ ] Design core entities and relationships
  - Review PRD Section 8 (Database Schema)
  - Create entity-relationship diagram
  - Define all tables and columns
  - Define foreign key relationships
  - Document data types and constraints
- [ ] Create initial migration files
  - Create users table migration
  - Create [main entity 1] table migration
  - Create [main entity 2] table migration
  - Add foreign key constraints
  - Add indexes for frequently queried columns
- [ ] Set up database seeding for development
  - Create seed script
  - Add sample users
  - Add sample data for main entities
  - Test seeding script
- [ ] Test database connections and queries
  - Test basic CRUD operations
  - Test relationship queries (joins)
  - Test data validation
  - Verify indexes are working

**Acceptance Criteria:**
- All tables are created in database
- Foreign key relationships are correct
- Migrations run successfully
- Seed data populates correctly
- Basic queries work as expected

**Notes:**
- Related PRD Section: Section 8 (Database Schema)
- ERD diagram: [Link to diagram]

---

### Phase 2: Core Features (Weeks 3-5)

---

**[TASK-005] Implement [Core Feature 1]**
- Status: 📋 Pending
- Owner: Unassigned
- Priority: High
- Dependencies: TASK-004

**Subtasks:**
- [ ] Create database models and migrations
  - Define [entity] model/schema
  - Create migration file
  - Add relationships to other entities
  - Run and test migration
- [ ] Build API endpoints (CRUD operations)
  - POST /api/[resource] - Create
  - GET /api/[resource] - List all
  - GET /api/[resource]/[id] - Get single
  - PUT /api/[resource]/[id] - Update
  - DELETE /api/[resource]/[id] - Delete
  - Add authentication checks
  - Add authorization (user can only access their data)
- [ ] Develop UI components and forms
  - Create [Resource]List component
  - Create [Resource]Card component
  - Create [Resource]Form component (create/edit)
  - Create [Resource]Detail component
  - Add loading states
  - Add empty states
- [ ] Add validation and error handling
  - Create Zod schema for [resource]
  - Add client-side validation
  - Add server-side validation
  - Implement error messages
  - Add success notifications
- [ ] Write initial tests
  - Unit tests for API endpoints
  - Integration tests for user flows
  - Test edge cases

**Acceptance Criteria:**
- Users can create new [resource]
- Users can view list of [resources]
- Users can view single [resource] details
- Users can edit existing [resource]
- Users can delete [resource]
- Validation works on client and server
- Error handling provides clear feedback

**Notes:**
- Related PRD Section: Section 4 (Functional Requirements)
- User story: [Link to specific user story]

---

**[TASK-006] Implement [Core Feature 2]**
- Status: 📋 Pending
- Owner: Unassigned
- Priority: High
- Dependencies: TASK-004, TASK-005

**Subtasks:**
- [ ] Create database models and migrations
  - [Specific subtasks based on feature]
- [ ] Build API endpoints (CRUD operations)
  - [Specific endpoints based on feature]
- [ ] Develop UI components and forms
  - [Specific components based on feature]
- [ ] Add validation and error handling
  - [Specific validation rules based on feature]
- [ ] Write initial tests
  - [Specific test scenarios based on feature]

**Acceptance Criteria:**
- [Feature-specific acceptance criteria]

**Notes:**
- Related PRD Section: [Section number]

---

**[TASK-007] Implement [Core Feature 3]**
- Status: 📋 Pending
- Owner: Unassigned
- Priority: Medium
- Dependencies: TASK-005, TASK-006

**Subtasks:**
- [ ] Create database models and migrations
- [ ] Build API endpoints (CRUD operations)
- [ ] Develop UI components and forms
- [ ] Add validation and error handling
- [ ] Write initial tests

**Acceptance Criteria:**
- [Feature-specific acceptance criteria]

**Notes:**
- Related PRD Section: [Section number]

---

**[TASK-008] Add data validation and error handling**
- Status: 📋 Pending
- Owner: Unassigned
- Priority: High
- Dependencies: TASK-005, TASK-006, TASK-007

**Subtasks:**
- [ ] Implement Zod schemas for all inputs
  - Create schema for user inputs
  - Create schema for [feature 1] inputs
  - Create schema for [feature 2] inputs
  - Add custom validation rules
  - Test all schemas
- [ ] Create error boundary components
  - Create global error boundary
  - Create feature-specific error boundaries
  - Add error logging
  - Design error UI states
- [ ] Add loading states throughout app
  - Add skeleton loaders for data fetching
  - Add spinners for form submissions
  - Add progress indicators for long operations
  - Ensure consistent loading UX
- [ ] Implement toast notifications
  - Install/configure toast library
  - Create success toast
  - Create error toast
  - Create info toast
  - Add toast to all user actions
- [ ] Handle edge cases and empty states
  - Design empty state UI
  - Add empty states to all lists
  - Handle no results found
  - Handle network errors
  - Handle unauthorized access

**Acceptance Criteria:**
- All user inputs are validated
- Errors are caught and displayed gracefully
- Loading states are clear and consistent
- Toast notifications provide feedback
- Empty states guide users appropriately

**Notes:**
- Related PRD Section: Section 13, Phase 2, Task 8

---

### Phase 3: Integration & Enhancement (Weeks 6-7)

---

**[TASK-009] Integrate [Third-Party Service 1]**
- Status: 📋 Pending
- Owner: Unassigned
- Priority: Medium
- Dependencies: TASK-005

**Example: Stripe for Payments**

**Subtasks:**
- [ ] Set up Stripe account and get API keys
  - Create Stripe account
  - Get test API keys
  - Add API keys to .env.local
  - Install Stripe SDK
- [ ] Configure webhooks
  - Set up webhook endpoint
  - Configure webhook secret
  - Add webhook signature verification
  - Handle webhook events
- [ ] Implement payment flow
  - Create checkout session endpoint
  - Build payment form component
  - Handle successful payment
  - Handle failed payment
  - Store payment records in database
- [ ] Test integration flows
  - Test with Stripe test cards
  - Test success scenarios
  - Test failure scenarios
  - Test webhook delivery
- [ ] Handle errors and edge cases
  - Handle network errors
  - Handle declined cards
  - Handle webhook failures
  - Add retry logic

**Acceptance Criteria:**
- Users can initiate payment flow
- Payments are processed successfully
- Webhooks are received and handled
- Payment records are stored
- Errors are handled gracefully

**Notes:**
- Related PRD Section: Section 4 (Functional Requirements)
- Stripe docs: [Link to integration guide]

---

**[TASK-010] Integrate [Third-Party Service 2]**
- Status: 📋 Pending
- Owner: Unassigned
- Priority: Medium
- Dependencies: TASK-006

**Example: OpenAI/Anthropic for AI Content Generation**

**Subtasks:**
- [ ] Set up API account and get keys
  - Create account with AI provider
  - Get API key
  - Add to .env.local
  - Install SDK
- [ ] Build prompt engineering and response handling
  - Design prompt templates
  - Create prompt construction logic
  - Implement API call with retry logic
  - Parse and validate responses
  - Handle rate limits
- [ ] Add streaming support (if needed)
  - Implement streaming endpoint
  - Add streaming UI component
  - Handle stream interruptions
  - Test streaming performance
- [ ] Implement rate limiting and error handling
  - Add rate limiting middleware
  - Handle API errors
  - Add fallback for API failures
  - Implement usage tracking

**Acceptance Criteria:**
- AI API is integrated successfully
- Prompts generate quality responses
- Streaming works smoothly (if applicable)
- Rate limits are respected
- Errors are handled appropriately

**Notes:**
- Related PRD Section: Section 4 (Functional Requirements)

---

**[TASK-011] Implement background jobs and workflows**
- Status: 📋 Pending
- Owner: Unassigned
- Priority: Medium
- Dependencies: TASK-008

**Subtasks:**
- [ ] Set up [Workflow/Job Queue System]
  - Install workflow system
  - Configure storage backend
  - Set up workflow functions
  - Create reusable steps
- [ ] Create scheduled tasks
  - Daily digest emails
  - Data cleanup jobs
  - Report generation
  - Test scheduling
- [ ] Implement email notifications
  - Set up email service
  - Create email templates
  - Implement notification triggers
  - Test email delivery
- [ ] Add retry logic and monitoring
  - Configure retry policies
  - Add error logging
  - Set up monitoring dashboard
  - Test failure scenarios

**Acceptance Criteria:**
- Background jobs execute reliably
- Scheduled tasks run on time
- Email notifications are sent correctly
- Failed jobs are retried
- Monitoring provides visibility

**Notes:**
- Related PRD Section: Section 13, Phase 3, Task 11

---

**[TASK-012] Add real-time features**
- Status: 📋 Pending
- Owner: Unassigned
- Priority: Low
- Dependencies: TASK-008

**Subtasks:**
- [ ] Configure real-time subscriptions
  - Set up real-time service/library
  - Configure connection handling
  - Create subscription channels
  - Test connection stability
- [ ] Implement live updates in UI
  - Subscribe to data changes
  - Update UI on data changes
  - Handle optimistic updates
  - Add visual indicators for updates
- [ ] Handle connection states
  - Show connected/disconnected states
  - Handle reconnection logic
  - Buffer updates during disconnection
  - Test offline scenarios
- [ ] Test synchronization logic
  - Test concurrent updates
  - Test conflict resolution
  - Test data consistency
  - Performance test with multiple clients

**Acceptance Criteria:**
- Real-time updates work reliably
- UI reflects changes immediately
- Connection issues are handled gracefully
- Data stays synchronized across clients

**Notes:**
- Related PRD Section: Section 13, Phase 3, Task 12

---

### Phase 4: Polish & Optimization (Week 8)

---

**[TASK-013] UI/UX polish and accessibility**
- Status: 📋 Pending
- Owner: Unassigned
- Priority: High
- Dependencies: All core feature tasks

**Subtasks:**
- [ ] Refine component styling and interactions
  - Review all components for consistency
  - Refine spacing and typography
  - Polish hover/focus states
  - Add smooth transitions
  - Review with design system
- [ ] Ensure WCAG 2.1 Level AA compliance
  - Run accessibility audit tools
  - Fix color contrast issues
  - Add proper heading hierarchy
  - Ensure form labels are correct
  - Test with accessibility checkers
- [ ] Test keyboard navigation
  - Test all interactive elements with keyboard
  - Ensure logical tab order
  - Add skip navigation links
  - Test modal focus trapping
  - Verify no keyboard traps
- [ ] Add proper ARIA labels
  - Add ARIA labels to interactive elements
  - Add ARIA live regions for dynamic content
  - Add ARIA descriptions where needed
  - Test with accessibility tree
- [ ] Test with screen readers
  - Test with VoiceOver (macOS/iOS)
  - Test with NVDA (Windows)
  - Verify all content is announced
  - Fix any issues found

**Acceptance Criteria:**
- All components have consistent styling
- WCAG 2.1 Level AA standards are met
- Keyboard navigation works throughout
- Screen readers can navigate the app
- ARIA labels are appropriate

**Notes:**
- Related PRD Section: Section 5 (Non-Functional Requirements - Accessibility)

---

**[TASK-014] Performance optimization**
- Status: 📋 Pending
- Owner: Unassigned
- Priority: High
- Dependencies: All core feature tasks

**Subtasks:**
- [ ] Implement code splitting and lazy loading
  - Add dynamic imports for routes
  - Lazy load heavy components
  - Split vendor bundles
  - Test bundle sizes
- [ ] Optimize images and assets
  - Use Next.js Image component
  - Compress images
  - Use modern formats (WebP, AVIF)
  - Add lazy loading for images
  - Implement responsive images
- [ ] Add caching strategies
  - Configure HTTP caching headers
  - Add service worker caching (if applicable)
  - Implement API response caching
  - Add client-side data caching
- [ ] Minimize bundle size
  - Remove unused dependencies
  - Tree-shake unused code
  - Minimize CSS and JS
  - Analyze bundle with tools
- [ ] Test Core Web Vitals
  - Test LCP (Largest Contentful Paint)
  - Test FID (First Input Delay)
  - Test CLS (Cumulative Layout Shift)
  - Optimize for scores > 90

**Acceptance Criteria:**
- Page load time < 2 seconds
- Bundle size is minimized
- Images are optimized
- Core Web Vitals scores are good
- Caching improves perceived performance

**Notes:**
- Related PRD Section: Section 5 (Non-Functional Requirements - Performance)

---

**[TASK-015] Testing and quality assurance**
- Status: 📋 Pending
- Owner: Unassigned
- Priority: High
- Dependencies: All feature tasks

**Subtasks:**
- [ ] Write integration tests for critical flows
  - Test auth flows
  - Test main user journeys
  - Test payment flows (if applicable)
  - Test data creation/editing
  - Achieve >70% coverage for critical paths
- [ ] Perform manual testing across browsers/devices
  - Test on Chrome, Firefox, Safari, Edge
  - Test on mobile Safari (iOS)
  - Test on Chrome Android
  - Test on tablets
  - Document any browser-specific issues
- [ ] Test error scenarios
  - Test network failures
  - Test API errors
  - Test validation errors
  - Test unauthorized access
  - Test edge cases
- [ ] Security review and penetration testing
  - Review authentication implementation
  - Test for XSS vulnerabilities
  - Test for CSRF vulnerabilities
  - Test for SQL injection (if applicable)
  - Review API security
- [ ] Load testing (if applicable)
  - Set up load testing environment
  - Test with expected user load
  - Test with 2x expected load
  - Identify bottlenecks
  - Optimize as needed

**Acceptance Criteria:**
- Integration tests pass for all critical flows
- App works on all supported browsers
- Error scenarios are handled correctly
- Security review passes
- Load testing meets performance targets

**Notes:**
- Related PRD Section: Section 5 (Non-Functional Requirements - Security)

---

**[TASK-016] Documentation and deployment prep**
- Status: 📋 Pending
- Owner: Unassigned
- Priority: Medium
- Dependencies: All feature tasks

**Subtasks:**
- [ ] Write user documentation/help articles
  - Create getting started guide
  - Document key features
  - Create FAQ
  - Add screenshots/videos
  - Review for clarity
- [ ] Create API documentation
  - Document all API endpoints
  - Add request/response examples
  - Document error codes
  - Create Postman collection
  - Test examples work
- [ ] Set up production environment
  - Create production database
  - Configure environment variables
  - Set up CDN (if needed)
  - Configure domain
  - Set up SSL certificate
- [ ] Configure monitoring and analytics
  - Set up error tracking (Sentry, etc.)
  - Configure analytics (PostHog, etc.)
  - Set up uptime monitoring
  - Configure performance monitoring
  - Test monitoring is working
- [ ] Prepare deployment pipeline
  - Configure CI/CD
  - Set up automated tests in CI
  - Configure deployment triggers
  - Test deployment process
  - Create rollback plan

**Acceptance Criteria:**
- User documentation is complete and clear
- API documentation is accurate
- Production environment is configured
- Monitoring and analytics are set up
- Deployment pipeline is tested

**Notes:**
- Related PRD Section: Section 13, Phase 4, Task 16

---

### Phase 5: Launch (Week 9+)

---

**[TASK-017] Deploy to production**
- Status: 📋 Pending
- Owner: Unassigned
- Priority: High
- Dependencies: TASK-013, TASK-014, TASK-015, TASK-016

**Subtasks:**
- [ ] Deploy application to [Deployment Platform]
  - Trigger production deployment
  - Monitor deployment logs
  - Verify deployment succeeded
  - Check application starts correctly
- [ ] Configure domain and SSL
  - Point domain to application
  - Verify SSL certificate
  - Test HTTPS redirect
  - Check all pages load via HTTPS
- [ ] Set up CDN (if applicable)
  - Configure CDN
  - Test asset delivery
  - Verify caching is working
  - Test from multiple regions
- [ ] Enable monitoring and error tracking
  - Verify error tracking is active
  - Test error reporting
  - Set up alert notifications
  - Configure uptime monitoring
- [ ] Test production environment
  - Test all critical user flows
  - Verify all integrations work
  - Test payment processing
  - Check analytics tracking
  - Verify email sending

**Acceptance Criteria:**
- Application is deployed and accessible
- Domain and SSL are configured correctly
- Monitoring and error tracking are active
- All features work in production
- No critical bugs are present

**Notes:**
- Related PRD Section: Section 13, Phase 5, Task 17
- Deployment checklist: [Link to checklist]

---

**[TASK-018] Post-launch monitoring and iteration**
- Status: 📋 Pending
- Owner: Unassigned
- Priority: High
- Dependencies: TASK-017

**Subtasks:**
- [ ] Monitor analytics and user behavior
  - Review daily active users
  - Analyze user funnels
  - Identify drop-off points
  - Track feature usage
  - Review user paths
- [ ] Track error rates and performance
  - Monitor error rates
  - Track response times
  - Review Core Web Vitals
  - Check uptime metrics
  - Identify performance bottlenecks
- [ ] Gather user feedback
  - Set up feedback mechanism
  - Conduct user interviews
  - Send user surveys
  - Monitor support requests
  - Collect feature requests
- [ ] Plan iteration roadmap
  - Review analytics insights
  - Prioritize bug fixes
  - Prioritize feature requests
  - Create next sprint plan
  - Update PRD with learnings
- [ ] Address critical issues
  - Fix critical bugs immediately
  - Address security issues
  - Resolve performance problems
  - Improve UX pain points

**Acceptance Criteria:**
- Analytics provide actionable insights
- Error monitoring catches issues
- User feedback is collected
- Iteration roadmap is defined
- Critical issues are addressed promptly

**Notes:**
- Related PRD Section: Section 13, Phase 5, Task 18
- This is an ongoing task post-launch

---

## ✅ Done

**Example: Something you finished**
- Status: ✅ Done on [Date]
- Who did it: [Your name]
- Notes: [Anything important about how it went]

---

## 🐛 Bugs to Fix

**Bug #1: Example problem**
- How bad: Really bad | Kind of bad | Not urgent
- Found on: [Date]
- Who's fixing it: [Name]
- Status: Need to fix | Fixing now | Fixed

**What's wrong:**
[Describe what's broken]

**How to see the bug:**
1. [Do this]
2. [Then do this]
3. [See the problem]

**What should happen:**
[What it's supposed to do]

**What actually happens:**
[What it's doing wrong]

**Notes:**
[Any other details or screenshots]

---

## 💡 Future Ideas

Cool things you might add later (after launch):

**Idea #1: Example feature**
- How important: Really want | Would be nice | Someday maybe
- How much work: A little | Medium amount | Lots of work
- How useful: Super useful | Pretty useful | Nice to have

**What it is:**
[Describe the feature]

**Why add it:**
[Why users would love this]

**Notes:**
[Any thoughts on how to build it]

---

## How Claude Updates This File

### Task Status (The Symbols)

- 📋 **Not started** - Haven't begun yet
- 🚧 **Working on it** - Building it now
- ✅ **Done** - Finished and tested
- ⏸️ **Stuck** - Waiting on something
- 🐛 **Bug** - Something to fix

### How Important

- **Must do first** - Can't skip this
- **Important** - Need it but can wait a bit
- **Nice to have** - Can do later

### How Claude Works with This

1. **When starting something new:**
   - Claude moves it to "Working on Now"
   - Updates status to 🚧 Working on it
   - Adds the date
   - Marks who's doing it (you!)

2. **While building:**
   - Claude checks off boxes as things get done
   - Adds notes about choices made
   - Updates if plans change

3. **When finished:**
   - Claude makes sure everything works
   - Tests it
   - Updates any docs needed
   - Moves it to "Done"
   - Adds when it was finished

4. **If stuck:**
   - Claude updates status to ⏸️ Stuck
   - Writes what the problem is
   - Notes what needs to happen to fix it

---

## Links to Other Files

- **Your planning doc:** `/docs/prd.md` (detailed plan for humans)
- **Claude's memory:** `/.claude/projectbrief.md` (short version for Claude)
- **Setup guide:** `/.claude/guides/setup-workflow.md` (how to get started)
- **Your code:** [GitHub URL]
- **Design files:** [Figma URL if you have designs]
- **Live app:** [Your URL when it's online]

---

**Remember:** Claude keeps this file updated for you automatically as you build!
