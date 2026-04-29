---
trigger: glob
globs: **/*.tsx
---

<react_components>

# React Component Guidelines

When creating or modifying React components (.tsx files):

## React 19 Compatibility

This project uses React 19 (stable since December 2024). Leverage new features when appropriate.

## When to Use React

- Interactive forms (ContactForm)
- Accordions and expandable content (FAQAccordion)
- Filters and dynamic content (CaseStudyFilter)
- Modals and overlays (SearchModal)
- State management requiring hooks

## React 19 New Features

### Actions

- Use Actions for form submissions instead of manual event handlers
- Actions automatically handle pending, error, and success states
- Simplifies form data handling and validation

### New Hooks

- `useActionState` - Manage form state with Actions
- `useFormStatus` - Access form submission status in nested components
- `useOptimistic` - Show optimistic UI updates during async operations
- `use` - Read resources (Promises, Context) in render

### Server Components

- Use Server Components for data fetching when appropriate
- Mark components with `use client` directive for interactivity
- Server Actions for server-side form handling

### Other React 19 Improvements

- `ref` can be passed as a prop (no need for `forwardRef`)
- Better hydration error reporting with diffs
- `<Context>` as a provider component
- Support for Document Metadata (title, meta tags)
- Support for stylesheets and async scripts
- Support for preloading resources

## Hooks Usage

- Use `useState` for local component state
- Use `useEffect` for side effects (cleanup when needed)
- Use `useRef` for DOM references
- Use `useId` for unique IDs (form labels, error messages)
- Use `useActionState` for form state management (React 19)
- Use `useFormStatus` for form submission status (React 19)
- Use `useOptimistic` for optimistic updates (React 19)

## Form Patterns (React 19)

- Use Actions for form submissions when possible
- Use `useActionState` to manage form state and errors
- Use `useFormStatus` to show loading/error states in nested components
- Implement proper validation with Actions
- Display error messages with proper ARIA attributes

### Example Form Pattern (React 19)

```tsx
async function submitForm(formData: FormData) {
  // Server-side validation and processing
  const data = Object.fromEntries(formData);
  // Return errors or success
}

function ContactForm() {
  const [state, formAction] = useActionState(submitForm, null);
  const { pending } = useFormStatus();

  return (
    <form action={formAction}>
      <input name="email" disabled={pending} />
      <button type="submit" disabled={pending}>
        {pending ? "Sending..." : "Submit"}
      </button>
    </form>
  );
}
```

### Legacy Form Pattern (Pre-React 19)

```tsx
const [formData, setFormData] = useState<FormData>({});
const [isSubmitting, setIsSubmitting] = useState(false);
const [errors, setErrors] = useState<Partial<FormData>>({});
const baseId = useId();
```

## Accessibility

- Use proper ARIA attributes: `aria-invalid`, `aria-describedby`, `aria-live`
- Associate labels with form elements using `htmlFor` and `id`
- Provide error messages linked to inputs
- Use semantic HTML
- Use Document Metadata for SEO (React 19)

## Styling

- Use Tailwind CSS classes
- Maintain consistency with Astro components
- Use dark mode styling with `dark:` prefix
- Use electric blue accent colors

## Performance

- Keep component state minimal
- Avoid unnecessary re-renders
- Use `useCallback` and `useMemo` when needed
- Lazy load heavy components if needed
- Leverage Server Components for data fetching (React 19)

## TypeScript

- Define interfaces for props and state
- Use proper type annotations
- Avoid `any` type
- Use generic types when appropriate
- Type Actions with proper return types

## Migration Notes

- Existing controlled components continue to work
- Gradually adopt Actions for new forms
- Consider Server Components for data-heavy pages
- Use `ref` as prop instead of `forwardRef` when refactoring

</react_components>
