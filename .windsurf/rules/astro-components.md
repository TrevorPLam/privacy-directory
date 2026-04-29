---
trigger: always_on
---

<astro_components>

# Astro Component Guidelines

When creating or modifying Astro components (.astro files):

## File Structure

- Use the `---` frontmatter fence for component props and logic
- Keep the component interface in the frontmatter
- Use TypeScript interfaces for type safety

## Props Pattern

```astro
---
interface Props {
  title: string;
  description?: string;
  variant?: "primary" | "secondary";
}

const { title, description = "", variant = "primary" } = Astro.props;
---
```

## Styling

- Use Tailwind CSS classes
- Prefer dark mode styling with `dark:` prefix
- Use custom utility classes: `glass-card`, `focus-ring`, `grid-pattern`
- Use electric blue accent colors: `electric-500`, `electric-400`, etc.

## Slot Usage

- Use `<slot />` for component composition
- Use named slots for complex components: `<slot name="header" />`

## Accessibility

- Use semantic HTML elements
- Include proper ARIA attributes when needed
- Ensure keyboard navigability
- Add alt text for images

## Performance

- Keep components small and focused
- Use Astro's static rendering by default
- Only use React islands for interactivity

## Naming

- Use PascalCase for component files: `Button.astro`, `Header.astro`
- Use descriptive names that indicate purpose

</astro_components>
