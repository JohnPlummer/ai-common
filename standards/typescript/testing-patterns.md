# Vitest and React Testing Library Pattern

Testing React components using Vitest test runner with React Testing Library for component queries and user interactions.

## Pattern

```tsx
import { render, screen } from '@testing-library/react';
import { describe, it, expect } from 'vitest';

describe('ComponentName', () => {
  it('describes expected behavior', () => {
    render(<ComponentName />);

    expect(screen.getByText('Expected Text')).toBeInTheDocument();
  });
});
```

## Why Use This Pattern

- **Fast**: Vitest 10-20x faster than Jest with TypeScript
- **User-centric**: Tests how users interact with components
- **Maintainable**: Tests are resilient to implementation changes
- **Accessible**: Encourages accessible component design
- **Modern**: Native ESM support, Vite integration

## Route Testing

```tsx
// frontend/src/tests/unit/router.test.tsx:24
describe('Application Routing', () => {
  it('should render Home page at /', async () => {
    await act(async () => {
      render(
        <MemoryRouter initialEntries={['/']}>
          <App />
        </MemoryRouter>
      );
    });

    const mainContent = screen.getByRole('main');
    expect(mainContent).toBeInTheDocument();
  });
});
```

## Component Testing

```tsx
describe('ActivityCard', () => {
  it('displays activity details', () => {
    const activity = {
      id: '1',
      title: 'Brighton Pier',
      description: 'Historic pier',
      category: 'entertainment'
    };

    render(<ActivityCard activity={activity} />);

    expect(screen.getByText('Brighton Pier')).toBeInTheDocument();
    expect(screen.getByText('entertainment')).toBeInTheDocument();
  });
});
```

## User Interactions

```tsx
import userEvent from '@testing-library/user-event';

it('handles button click', async () => {
  const handleClick = vi.fn();
  render(<Button onClick={handleClick} label="Click me" />);

  await userEvent.click(screen.getByRole('button', { name: 'Click me' }));

  expect(handleClick).toHaveBeenCalledOnce();
});
```

## Async Operations

```tsx
import { waitFor } from '@testing-library/react';

it('loads and displays data', async () => {
  render(<DataLoader />);

  expect(screen.getByText('Loading...')).toBeInTheDocument();

  await waitFor(() => {
    expect(screen.getByText('Data loaded')).toBeInTheDocument();
  });
});
```

## Form Testing

```tsx
it('submits form with valid data', async () => {
  const handleSubmit = vi.fn();
  render(<Form onSubmit={handleSubmit} />);

  await userEvent.type(screen.getByLabelText('Email'), 'user@example.com');
  await userEvent.type(screen.getByLabelText('Password'), 'password123');
  await userEvent.click(screen.getByRole('button', { name: 'Submit' }));

  expect(handleSubmit).toHaveBeenCalledWith({
    email: 'user@example.com',
    password: 'password123'
  });
});
```

## Query Priorities

Use queries in this priority order:

1. **getByRole**: Most accessible (e.g., `getByRole('button', { name: 'Submit' })`)
2. **getByLabelText**: Form inputs (e.g., `getByLabelText('Email')`)
3. **getByPlaceholderText**: When label is not available
4. **getByText**: Content matching (e.g., `getByText('Welcome')`)
5. **getByTestId**: Last resort (e.g., `getByTestId('custom-element')`)

## Query Variants

```tsx
// getBy: Throws error if not found (use for assertions)
const button = screen.getByRole('button');

// queryBy: Returns null if not found (use for checking absence)
const button = screen.queryByRole('button');
expect(button).not.toBeInTheDocument();

// findBy: Async, waits for element (use for async content)
const button = await screen.findByRole('button');
```

## Mocking with Vitest

```tsx
import { vi } from 'vitest';

// Mock function
const mockFn = vi.fn();

// Mock module
vi.mock('./api', () => ({
  fetchData: vi.fn(() => Promise.resolve({ data: [] }))
}));

// Spy on method
const spy = vi.spyOn(console, 'error');
```

## MSW for API Mocking

```tsx
import { rest } from 'msw';
import { setupServer } from 'msw/node';

const server = setupServer(
  rest.get('/api/activities', (req, res, ctx) => {
    return res(
      ctx.status(200),
      ctx.json([{ id: '1', title: 'Activity 1' }])
    );
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

## Guidelines

1. **Test behavior**: Focus on what users see and do
2. **Avoid implementation details**: Don't test internal state
3. **Use semantic queries**: Prefer roles and labels over test IDs
4. **Wait for async**: Use waitFor or findBy for async operations
5. **Clean up**: Vitest auto-cleans between tests

## Common Mistakes

```tsx
// Bad: Testing implementation details
expect(component.state.count).toBe(1);

// Good: Testing user-visible behavior
expect(screen.getByText('Count: 1')).toBeInTheDocument();

// Bad: Using querySelector
const button = container.querySelector('.submit-button');

// Good: Using accessible query
const button = screen.getByRole('button', { name: 'Submit' });

// Bad: Not waiting for async
render(<AsyncComponent />);
expect(screen.getByText('Loaded')).toBeInTheDocument(); // Fails!

// Good: Wait for async content
render(<AsyncComponent />);
expect(await screen.findByText('Loaded')).toBeInTheDocument();
```

## References

- Vitest: <https://vitest.dev/>
- React Testing Library: <https://testing-library.com/react>
- Testing Library Queries: <https://testing-library.com/docs/queries/about>
