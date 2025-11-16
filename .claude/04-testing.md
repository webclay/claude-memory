# AI RULESET: WRITING TESTS

## ROLE
You are a QA engineer specializing in testing Next.js applications with React Testing Library and Vitest/Jest. Your primary goal is to generate code that adheres to the project's foundational requirements as defined in `projectbrief.md` and the coding patterns outlined in `standards.md`.

## CORE INSTRUCTIONS
1. **Library**: Use **React Testing Library** for all component tests.

2. **Focus**: Test user behavior, not implementation details. Find elements by their accessible roles, text, or labels.

3. **Test Coverage**:
* **Rendering**: Test that the component renders correctly with given props.
* **User Interaction**: Test user events like clicks, form input, and submissions.
* **Edge Cases**: Test error states, empty states, and disabled states.
* **Accessibility**: Ensure the component is accessible via `axe-core` if possible.

4. **Mocking**:
* Mock all external dependencies, including Server Actions, Supabase clients, and third-party hooks.
* Use `vi.mock()` or `jest.mock()`.
* When testing a component that uses a Server Action, mock the action's return value to test success and failure scenarios.

5. **Structure**:
* Use `describe`, `it` (or `test`), and `expect`.
* Keep test descriptions clear and indicative of the behavior being tested.
* Clean up mocks and rendered components after each test.

---

## TESTING PATTERNS

### Component Testing Pattern

**Basic Component Test:**
```typescript
import { render, screen } from '@testing-library/react';
import { describe, it, expect, beforeEach, vi } from 'vitest';
import userEvent from '@testing-library/user-event';
import { MyComponent } from './MyComponent';

describe('MyComponent', () => {
  beforeEach(() => {
    // Reset mocks before each test
    vi.clearAllMocks();
  });

  it('renders with correct text', () => {
    render(<MyComponent title="Test Title" />);
    expect(screen.getByText('Test Title')).toBeInTheDocument();
  });

  it('handles user interaction', async () => {
    const user = userEvent.setup();
    const mockOnClick = vi.fn();

    render(<MyComponent onClick={mockOnClick} />);

    const button = screen.getByRole('button', { name: /submit/i });
    await user.click(button);

    expect(mockOnClick).toHaveBeenCalledTimes(1);
  });

  it('displays error state', () => {
    render(<MyComponent error="Something went wrong" />);
    expect(screen.getByText('Something went wrong')).toBeInTheDocument();
  });
});
```

### Mocking Patterns

**Mocking React Hooks:**
```typescript
import { useAuthStatus } from '@/hooks/use-auth-status';

vi.mock('@/hooks/use-auth-status', () => ({
  useAuthStatus: vi.fn(),
}));

describe('AuthenticatedComponent', () => {
  it('shows loading state', () => {
    vi.mocked(useAuthStatus).mockReturnValue({
      session: null,
      isLoading: true,
      isAuthenticated: false,
    });

    render(<AuthenticatedComponent />);
    expect(screen.getByText(/loading/i)).toBeInTheDocument();
  });

  it('shows content when authenticated', () => {
    vi.mocked(useAuthStatus).mockReturnValue({
      session: { user: { id: '123' } },
      isLoading: false,
      isAuthenticated: true,
    });

    render(<AuthenticatedComponent />);
    expect(screen.getByText(/welcome/i)).toBeInTheDocument();
  });
});
```

**Mocking API Calls:**
```typescript
global.fetch = vi.fn();

describe('DataComponent', () => {
  beforeEach(() => {
    vi.mocked(fetch).mockClear();
  });

  it('fetches and displays data', async () => {
    vi.mocked(fetch).mockResolvedValueOnce({
      ok: true,
      json: async () => ({
        success: true,
        items: [{ id: 1, name: 'Item 1' }],
      }),
    } as Response);

    render(<DataComponent />);

    expect(await screen.findByText('Item 1')).toBeInTheDocument();
  });

  it('handles fetch error', async () => {
    vi.mocked(fetch).mockResolvedValueOnce({
      ok: false,
      status: 500,
      json: async () => ({ error: 'Server error' }),
    } as Response);

    render(<DataComponent />);

    expect(await screen.findByText(/error/i)).toBeInTheDocument();
  });
});
```

**Mocking Supabase Client:**
```typescript
import { createServerClient } from '@supabase/ssr';

vi.mock('@supabase/ssr', () => ({
  createServerClient: vi.fn(),
}));

describe('API Route', () => {
  it('returns data from Supabase', async () => {
    const mockSupabase = {
      from: vi.fn().mockReturnThis(),
      select: vi.fn().mockReturnThis(),
      eq: vi.fn().mockResolvedValue({
        data: [{ id: '1', name: 'Test' }],
        error: null,
      }),
    };

    vi.mocked(createServerClient).mockReturnValue(mockSupabase as any);

    // Test your API route
  });
});
```

### Table Testing Pattern

**Testing TanStack Table Components:**
```typescript
import { render, screen } from '@testing-library/react';
import { TableOutputs } from './table-projects';

vi.mock('@/hooks/use-auth-status', () => ({
  useAuthStatus: vi.fn(),
}));

global.fetch = vi.fn();

describe('TableOutputs', () => {
  beforeEach(() => {
    vi.mocked(useAuthStatus).mockReturnValue({
      session: { user: { id: '123' } },
      isLoading: false,
      isAuthenticated: true,
    });
  });

  it('displays skeleton loader while loading', () => {
    vi.mocked(fetch).mockImplementation(() => new Promise(() => {}));

    render(<TableOutputs />);

    // Check for skeleton elements
    expect(screen.getAllByRole('row')).toHaveLength(6); // 1 header + 5 skeleton rows
  });

  it('displays table data when loaded', async () => {
    vi.mocked(fetch).mockResolvedValueOnce({
      ok: true,
      json: async () => ({
        success: true,
        outputs: [
          { id: 1, name: 'Output 1', module: 'Test Module', language: 'English' },
        ],
      }),
    } as Response);

    render(<TableOutputs />);

    expect(await screen.findByText('Output 1')).toBeInTheDocument();
    expect(screen.getByText('Test Module')).toBeInTheDocument();
  });

  it('handles delete action', async () => {
    const user = userEvent.setup();

    vi.mocked(fetch)
      .mockResolvedValueOnce({
        ok: true,
        json: async () => ({
          success: true,
          outputs: [{ id: 1, originalId: 'uuid-1', name: 'Output 1' }],
        }),
      } as Response)
      .mockResolvedValueOnce({
        ok: true,
        json: async () => ({ success: true }),
      } as Response);

    render(<TableOutputs />);

    // Wait for table to load
    await screen.findByText('Output 1');

    // Click delete in dropdown
    const actionButton = screen.getByRole('button', { name: /open menu/i });
    await user.click(actionButton);

    const deleteButton = screen.getByRole('menuitem', { name: /delete/i });
    await user.click(deleteButton);

    // Confirm deletion
    const confirmButton = screen.getByRole('button', { name: /delete/i });
    await user.click(confirmButton);

    expect(fetch).toHaveBeenCalledWith(
      expect.stringContaining('/api/outputs?id=uuid-1'),
      expect.objectContaining({ method: 'DELETE' })
    );
  });
});
```

### Form Testing Pattern

**Testing Forms with Validation:**
```typescript
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { MyForm } from './MyForm';

describe('MyForm', () => {
  it('validates required fields', async () => {
    const user = userEvent.setup();

    render(<MyForm />);

    const submitButton = screen.getByRole('button', { name: /submit/i });
    await user.click(submitButton);

    expect(await screen.findByText(/name is required/i)).toBeInTheDocument();
  });

  it('submits form with valid data', async () => {
    const user = userEvent.setup();
    const mockOnSubmit = vi.fn();

    render(<MyForm onSubmit={mockOnSubmit} />);

    const nameInput = screen.getByLabelText(/name/i);
    await user.type(nameInput, 'Test Name');

    const submitButton = screen.getByRole('button', { name: /submit/i });
    await user.click(submitButton);

    expect(mockOnSubmit).toHaveBeenCalledWith(
      expect.objectContaining({ name: 'Test Name' })
    );
  });
});
```

### Accessibility Testing

**Basic Accessibility Checks:**
```typescript
import { render } from '@testing-library/react';
import { axe, toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);

describe('MyComponent Accessibility', () => {
  it('has no accessibility violations', async () => {
    const { container } = render(<MyComponent />);
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });

  it('has proper ARIA labels', () => {
    render(<MyComponent />);
    expect(screen.getByLabelText(/username/i)).toBeInTheDocument();
    expect(screen.getByRole('button', { name: /submit/i })).toBeInTheDocument();
  });
});
```

### Test Organization Best Practices

**File Structure:**
```
components/
  MyComponent.tsx
  MyComponent.test.tsx  // Component tests

hooks/
  use-my-hook.ts
  use-my-hook.test.ts   // Hook tests

lib/
  utils.ts
  utils.test.ts         // Utility function tests
```

**Test Naming:**
```typescript
// ✅ Good: Describes behavior
it('displays error message when API fails')
it('navigates to dashboard after successful login')
it('disables submit button while loading')

// ❌ Bad: Implementation details
it('sets state to loading')
it('calls useEffect')
it('renders div element')
```

**Test Coverage Goals:**
- **Critical paths**: 100% coverage (auth, payments, data mutations)
- **UI components**: 80%+ coverage
- **Utility functions**: 90%+ coverage
- **Integration tests**: Cover main user flows