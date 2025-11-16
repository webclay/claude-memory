# [PROJECT_NAME] - Coding Standards

**Last Updated:** [Will be updated during initialization]
**Status:** Template

---

## 1. Core Philosophy & General Rules

You are an expert senior developer specializing in TypeScript, React, and modern web development.

Always think carefully and only action the specific task given with the most concise and elegant solution that changes as little code as possible.

### 1.1. Core Principles
* All code MUST be TypeScript with strict mode. NO `any` type.
* Use functional and declarative patterns.
* Implement proper error handling and validation.
* Follow accessibility guidelines (WCAG).
* Optimize for performance and Core Web Vitals.

### 1.2. Code Style & Naming
* **Patterns**: Functional/declarative patterns.
* **Exports**: Named exports for components.
* **Handlers**: Event handlers named `handleXxx`.
* **Directories**: `lowercase-with-dashes`.
* **Components/Interfaces**: `PascalCase`.
* **Variables/Functions**: `camelCase`.
* **Constants**: `UPPER_CASE`.
* **Variable Names**: Use descriptive names (e.g., `isLoading`, `hasError`).

### 1.3. Commenting
* Use JSDoc3 style for functions to explain the "why," not the "what."

### 1.4. File Structure
* Group files by domain/feature.
* `lib/` is for low-level logic (e.g., database client, utils).
* `components/ui` for reusable UI, `components/layout` for layouts, `components/{feature}` for feature-specific components.

### 1.5. Dependencies
* Always ask for confirmation before adding a new dependency.

---

## 2. Entity Reference Standards ⭐ CRITICAL

**ALWAYS use UUIDs for database foreign keys and API parameters. NEVER use names or slugs for database references.**

This prevents breakage when entities are renamed and ensures data integrity.

### The Golden Rules

1. **Database Foreign Keys**: ALWAYS use UUID columns (e.g., `entity_id`)
2. **API Parameters**: PREFER UUIDs for entity references (slugs acceptable ONLY for URL routing)
3. **Component Props**: Pass UUIDs when referencing entities
4. **Display**: Use `name` fields ONLY for UI display, never for lookups

### Reference Decision Matrix

| Use Case | Use UUID `id` | Use `slug` | Use `name` |
|----------|---------------|------------|------------|
| Database foreign key | ✅ ALWAYS | ❌ NEVER | ❌ NEVER |
| API query parameter | ✅ PREFERRED | ⚠️ URL routing only | ❌ NEVER |
| Component prop | ✅ YES | ⚠️ Only for URLs | ❌ NO |
| URL routing | ⚠️ Optional | ✅ YES (human-readable) | ❌ NO |
| User-facing display | ❌ NO | ⚠️ Sometimes | ✅ YES |
| Database queries | ✅ ALWAYS | ⚠️ Avoid | ❌ NEVER |

### Entity-Specific Patterns

**[ENTITY_NAME] Example:**
```typescript
// ✅ CORRECT - Use entity_id
const { data } = await db
  .from("[table_name]")
  .select("*")
  .eq("[entity]_id", entityId); // UUID

// ❌ WRONG - Don't use name for database queries
.eq("name", entityName); // FRAGILE - breaks on rename
```

**Component Patterns:**
```typescript
// ✅ CORRECT - Pass entity ID
<EntitySelect
  value={selectedEntityId}  // UUID
  onChange={(id) => setSelectedEntityId(id)}
/>

// ✅ CORRECT - Use entity ID in API calls
generateContent({ entityId: selectedEntityId });

// ❌ WRONG - Don't use entity name
generateContent({ entityName: "Some Name" }); // FRAGILE
```

### API Endpoint Patterns

```typescript
// ✅ CORRECT - ID-based resource endpoints
GET  /api/[resource]/[id]              // UUID in path
GET  /api/[resource]?id={uuid}         // UUID in query param

// ⚠️ ACCEPTABLE - Slug for URL routing only
GET  /app/[resource]/[slug]            // Human-readable URLs

// ❌ WRONG - Name-based lookups
GET  /api/[resource]?name={name}       // Fragile, breaks on rename
```

---

## 3. TypeScript Patterns

### 3.1. Type Safety
* Use explicit types for function parameters and return values
* Prefer `unknown` over `any` when type is genuinely unknown
* Use const assertions (`as const`) for immutable values
* Leverage type narrowing instead of type assertions

### 3.2. Interfaces vs Types
* Use `interface` for object shapes that might be extended
* Use `type` for unions, intersections, and primitives
* Prefer composition over inheritance

---

## 4. React Patterns

### 4.1. Component Structure
* Use function components over class components
* Call hooks at top level only, never conditionally
* Specify all dependencies in hook dependency arrays
* Use `key` prop for elements in iterables (prefer unique IDs over indices)

### 4.2. State Management
* Keep state as local as possible
* Lift state up only when needed by multiple components
* Use context for global state sparingly
* Consider state management libraries for complex state

### 4.3. Performance
* Use React.memo for expensive components
* Use useCallback for functions passed as props
* Use useMemo for expensive computations
* Avoid premature optimization

---

## 5. Error Handling

### 5.1. API Errors
* Always handle errors in API calls
* Provide user-friendly error messages
* Log errors appropriately
* Use consistent error response formats

### 5.2. Form Validation
* Validate on client and server
* Use validation libraries (e.g., Zod)
* Provide clear validation feedback
* Disable submit during validation

---

## 6. Accessibility

### 6.1. Semantic HTML
* Use semantic elements (`<button>`, `<nav>`, etc.)
* Provide meaningful alt text for images
* Use proper heading hierarchy
* Add labels for form inputs

### 6.2. Keyboard Navigation
* Ensure all interactive elements are keyboard accessible
* Provide focus indicators
* Use proper tab order
* Handle keyboard events alongside mouse events

### 6.3. ARIA
* Use ARIA attributes when semantic HTML isn't enough
* Don't override semantic HTML with ARIA
* Test with screen readers
* Follow ARIA authoring practices

---

## 7. Performance

### 7.1. Loading States
* Show loading indicators for async operations
* Use skeleton screens for better perceived performance
* Implement proper loading boundaries
* Handle loading errors gracefully

### 7.2. Code Splitting
* Split routes for better initial load
* Lazy load components when appropriate
* Use dynamic imports for large dependencies
* Monitor bundle size

### 7.3. Caching
* Implement appropriate caching strategies
* Use stale-while-revalidate patterns
* Cache static assets aggressively
* Invalidate caches appropriately

---

## 8. Security

### 8.1. Input Validation
* Validate all user input
* Sanitize data before rendering
* Prevent XSS attacks
* Avoid SQL injection

### 8.2. Authentication
* Use secure session management
* Implement proper logout
* Handle expired sessions
* Use HTTPS only

### 8.3. Authorization
* Check permissions on client and server
* Implement row-level security
* Use role-based access control
* Deny by default

---

## 9. Testing

### 9.1. Unit Tests
* Test business logic in isolation
* Mock external dependencies
* Aim for high coverage of critical paths
* Keep tests simple and readable

### 9.2. Component Tests
* Test user interactions
* Test different states
* Test accessibility
* Use React Testing Library

### 9.3. Integration Tests
* Test complete user flows
* Test API integration
* Test error scenarios
* Use realistic data

---

## 10. Git Workflow

### 10.1. Commits
* Use Conventional Commit messages (feat:, fix:, refactor:, docs:, etc.)
* Keep commits small and focused
* Test before committing
* Never force push to main/master

### 10.2. Branches
* Create feature branches from main
* Use descriptive branch names
* Delete branches after merge
* Keep branches short-lived

### 10.3. Pull Requests
* Write clear PR descriptions
* Link to related issues
* Request reviews from appropriate people
* Address review comments

---

## 11. Documentation

### 11.1. Code Documentation
* Document complex logic
* Explain non-obvious decisions
* Keep comments up to date
* Prefer self-documenting code

### 11.2. API Documentation
* Document all public APIs
* Include usage examples
* Document error cases
* Keep documentation in sync with code

### 11.3. README
* Explain what the project does
* Provide setup instructions
* Document environment variables
* Include contribution guidelines

---

## 12. Project-Specific Conventions

[This section will be populated during initialization based on projectbrief.md]

### 12.1. Project Entities
[Will list main entities from your project]

### 12.2. Module Structure
[Will describe your specific module organization]

### 12.3. Naming Conventions
[Will document project-specific naming patterns]

---

## 13. UI Component Hierarchy

[This section will be updated based on your chosen UI libraries]

**Preferred Component Sources:**
1. [Your primary UI library] (use first if available)
2. [Your secondary UI library] (use if not in primary)
3. Custom components (only if not available in libraries)

**Component Consistency:**
* Always use the same component from the same library
* Don't mix similar components from different libraries
* Document any custom wrappers or extensions

---

*This file defines the coding standards for [PROJECT_NAME].*
*Update as the project evolves and new patterns emerge.*
*See `.claude/07-documentation-maintenance.md` for update protocols.*
