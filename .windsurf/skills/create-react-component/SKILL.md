---
name: create-react-component
description: Guides creation of new React components with proper TypeScript interfaces, hooks patterns, and testing
---

# React Component Creation Guide

## File Location

Create interactive React components in `src/components/` with `.tsx` extension.

## When to Use React

Use React (.tsx) for:

- Interactive forms (ContactForm)
- Accordions and expandable content (FAQAccordion)
- Filters and dynamic content (CaseStudyFilter)
- Modals and overlays (SearchModal)
- State management requiring hooks
- User input handling

Use Astro (.astro) for:

- Static UI components (Button, Footer, Header)
- Display cards (ServiceCard, TeamCard, PricingCard)
- Layout components
- Content that doesn't need interactivity

## Component Structure

```tsx
import { useState, useEffect } from "react";

interface Props {
  title: string;
  description?: string;
  variant?: "primary" | "secondary";
  onAction?: () => void;
}

export default function ComponentName({
  title,
  description = "",
  variant = "primary",
  onAction,
}: Props) {
  const [state, setState] = useState(initialValue);
  const baseId = useId();

  useEffect(() => {
    // Side effects
    return () => {
      // Cleanup
    };
  }, [dependencies]);

  const handleClick = () => {
    // Event handler
    onAction?.();
  };

  return (
    <div className="component-class">
      <h2>{title}</h2>
      {description && <p>{description}</p>}
      <button onClick={handleClick}>Action</button>
    </div>
  );
}
```

## TypeScript Best Practices

### Props Interface

- Define interfaces for all props
- Use optional properties with `?`
- Provide default values in destructuring
- Use discriminated unions for conditional props

### Type Safety

- Avoid `any` type - use `unknown` when truly unknown
- Use proper type annotations for functions
- Type event handlers: `FormEvent`, `MouseEvent`, etc.
- Use generic types for reusable components

### Example Patterns

```tsx
// Discriminated unions for conditional rendering
type ButtonProps =
  | {
      variant: "primary";
      primaryAction: () => void;
    }
  | {
      variant: "secondary";
      secondaryAction: () => void;
    };

// Generic component
function List<T>({ items, renderItem }: { items: T[]; renderItem: (item: T) => JSX.Element }) {
  return <ul>{items.map(renderItem)}</ul>;
}

// Event handler typing
const handleSubmit = (e: FormEvent<HTMLFormElement>) => {
  e.preventDefault();
  // Handle form
};
```

## Hooks Usage

### useState

- Use for local component state
- Initialize with proper type
- Use functional updates for derived state
- Keep state minimal

```tsx
const [isOpen, setIsOpen] = useState(false);
const [data, setData] = useState<DataItem[]>([]);
```

### useEffect

- Use for side effects (API calls, subscriptions)
- Always include cleanup for subscriptions
- Specify all dependencies
- Avoid missing dependency warnings

```tsx
useEffect(() => {
  const fetchData = async () => {
    const result = await api.getData();
    setData(result);
  };
  fetchData();

  return () => {
    // Cleanup
  };
}, [dependency]);
```

### useRef

- Use for DOM references
- Use for persisting values across renders
- Access with `.current`

```tsx
const inputRef = useRef<HTMLInputElement>(null);
const previousValue = useRef(value);
```

### useId

- Use for unique IDs (form labels, error messages)
- Generates stable IDs across server/client
- Accessible for ARIA attributes

```tsx
const baseId = useId();
<input id={`${baseId}-name`} aria-describedby={`${baseId}-error`} />;
```

### useCallback and useMemo

- Use `useCallback` for functions passed to children
- Use `useMemo` for expensive computations
- Don't over-optimize prematurely

```tsx
const handleClick = useCallback(() => {
  doSomething(dependency);
}, [dependency]);

const expensiveValue = useMemo(() => {
  return computeExpensiveValue(data);
}, [data]);
```

## React 19 New Hooks

### useOptimistic

Manage optimistic UI updates during async operations:

```tsx
import { useOptimistic } from "react";

function TodoList({ todos, addTodo }) {
  const [optimisticTodos, addOptimisticTodo] = useOptimistic(todos, (state, newTodo) => [
    ...state,
    { ...newTodo, sending: true },
  ]);

  const handleSubmit = async (formData) => {
    const newTodo = { id: Date.now(), text: formData.get("text") };
    addOptimisticTodo(newTodo);
    await addTodo(newTodo);
  };

  return (
    <form action={handleSubmit}>
      <input name="text" />
      <button type="submit">Add</button>
      {optimisticTodos.map((todo) => (
        <div key={todo.id}>
          {todo.text} {todo.sending && "(sending...)"}
        </div>
      ))}
    </form>
  );
}
```

### useActionState

Handle form state with Actions pattern:

```tsx
import { useActionState } from "react";

async function submitForm(prevState, formData) {
  const email = formData.get("email");
  // Process form submission
  return { success: true, message: "Form submitted!" };
}

function ContactForm() {
  const [state, formAction, isPending] = useActionState(submitForm, null);

  return (
    <form action={formAction}>
      <input name="email" type="email" required />
      <button type="submit" disabled={isPending}>
        {isPending ? "Submitting..." : "Submit"}
      </button>
      {state?.message && <p>{state.message}</p>}
    </form>
  );
}
```

### useFormStatus

Track form submission status in nested components:

```tsx
import { useFormStatus } from "react";

function SubmitButton() {
  const { pending } = useFormStatus();
  return (
    <button type="submit" disabled={pending}>
      {pending ? "Submitting..." : "Submit"}
    </button>
  );
}

function ContactForm() {
  return (
    <form action={submitForm}>
      <input name="email" type="email" required />
      <SubmitButton />
    </form>
  );
}
```

## Form Patterns

### Controlled Components

- Use controlled components with state
- Implement proper validation
- Show loading states during async operations
- Display error messages with proper ARIA attributes

```tsx
const [formData, setFormData] = useState<FormData>({});
const [isSubmitting, setIsSubmitting] = useState(false);
const [errors, setErrors] = useState<Partial<FormData>>({});

const handleChange = (e: ChangeEvent<HTMLInputElement>) => {
  const { name, value } = e.target;
  setFormData((prev) => ({ ...prev, [name]: value }));
};

const handleSubmit = async (e: FormEvent) => {
  e.preventDefault();
  setIsSubmitting(true);
  try {
    await submitForm(formData);
  } catch (error) {
    setErrors(error.response.data);
  } finally {
    setIsSubmitting(false);
  }
};
```

### Form Validation

- Validate on change or blur
- Show inline error messages
- Use ARIA attributes for accessibility
- Disable submit button during submission

```tsx
<input
  id={`${baseId}-email`}
  name="email"
  type="email"
  value={formData.email}
  onChange={handleChange}
  aria-invalid={!!errors.email}
  aria-describedby={`${baseId}-email-error`}
/>;
{
  errors.email && (
    <p id={`${baseId}-email-error`} className="error">
      {errors.email}
    </p>
  );
}
```

## Accessibility

### ARIA Attributes

- Use `aria-invalid` for form validation errors
- Use `aria-describedby` for error messages
- Use `aria-live` for dynamic content updates
- Use `aria-expanded` for collapsible content
- Use `aria-label` for icon-only buttons

### Keyboard Navigation

- Ensure all interactive elements are keyboard accessible
- Use proper focus indicators
- Support Tab navigation order
- Provide keyboard alternatives for mouse actions

### Semantic HTML

- Use semantic elements (button, form, input)
- Associate labels with form elements
- Use proper heading hierarchy
- Add alt text for images

## Styling

### Tailwind CSS

- Use Tailwind CSS classes
- Maintain consistency with Astro components
- Use dark mode styling with `dark:` prefix
- Use electric blue accent colors

### Styling Pattern

```tsx
<div className="text-white dark:bg-dark-900">
  <div className="glass-card shadow-neon-md">
    <Button variant="primary" size="lg">
      Action
    </Button>
  </div>
</div>
```

## Performance

### Optimization

- Keep component state minimal
- Avoid unnecessary re-renders
- Use `useCallback` and `useMemo` when needed
- Lazy load heavy components if needed
- Use React.memo for expensive components

### React.memo

```tsx
export default React.memo(function ComponentName({ prop }: Props) {
  // Component logic
});
```

## Testing

### Unit Tests

- Write tests for component logic
- Test user interactions
- Test edge cases
- Mock external dependencies

### Example Test

```tsx
import { render, screen } from "@testing-library/react";
import ComponentName from "./ComponentName";

describe("ComponentName", () => {
  it("should render title", () => {
    render(<ComponentName title="Test" />);
    expect(screen.getByText("Test")).toBeInTheDocument();
  });
});
```

## Naming Convention

- Use PascalCase: `ContactForm.tsx`, `FAQAccordion.tsx`
- Use descriptive names that indicate purpose
- Match file name with component name

## Import Path Aliases

Use path aliases instead of relative imports:

```tsx
import Button from "@components/Button.astro";
import { someFunction } from "@utils/helpers";
```

## Common Patterns

### Event Delegation

```tsx
const handleItemClick = (item: Item) => {
  setSelectedItem(item);
};

{
  items.map((item) => (
    <button key={item.id} onClick={() => handleItemClick(item)}>
      {item.name}
    </button>
  ));
}
```

### Conditional Rendering

```tsx
{
  isLoading && <LoadingSpinner />;
}
{
  error && <ErrorMessage message={error} />;
}
{
  data && <DataList items={data} />;
}
```

### List Rendering

```tsx
{
  items.map((item) => <div key={item.id}>{item.name}</div>);
}
```

## After Creation

1. Test the component in isolation
2. Verify styling in both light and dark modes
3. Check responsive behavior
4. Test keyboard navigation
5. Verify accessibility with screen reader
6. Write unit tests for component logic
7. Test error handling
8. Verify performance (no unnecessary re-renders)
