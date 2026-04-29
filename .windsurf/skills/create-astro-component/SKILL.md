---
name: create-astro-component
description: Guides creation of new Astro components with proper TypeScript interfaces and styling
---

# Astro Component Creation Guide

## File Location

Create static UI components in `src/components/` with `.astro` extension.

## Component Structure

```astro
---
interface Props {
  title: string;
  description?: string;
  variant?: "primary" | "secondary";
}

const { title, description = "", variant = "primary" } = Astro.props;
---

<div class="component-class">
  <slot />
</div>
```

## TypeScript Best Practices

- Define interfaces for props
- Use optional properties with `?`
- Provide default values in destructuring
- Use proper type annotations

## Styling Guidelines

- Use Tailwind CSS classes
- Prefer dark mode styling with `dark:` prefix
- Use custom utilities: `glass-card`, `focus-ring`, `grid-pattern`
- Use electric blue accent colors: `electric-500`, `electric-400`
- Use neon effects: `shadow-neon-sm/md/lg/xl`

## Component Types

**Static Components (.astro)**: Use for:

- Display cards (ServiceCard, TeamCard, PricingCard)
- Layout components (Header, Footer)
- Static UI elements (Button, ClientLogos)

**Interactive Components (.tsx)**: Use React for:

- Forms (ContactForm)
- Accordions (FAQAccordion)
- Filters (CaseStudyFilter)
- Modals (SearchModal)

## Slot Usage

```astro
<!-- Default slot -->
<slot />

<!-- Named slots -->
<slot name="header" />
<slot name="footer" />
```

## Accessibility

- Use semantic HTML elements
- Include proper ARIA attributes when needed
- Ensure keyboard navigability
- Add alt text for images

## Naming Convention

- Use PascalCase: `Button.astro`, `Header.astro`, `ServiceCard.astro`
- Use descriptive names that indicate purpose

## Import Path Aliases

Use path aliases instead of relative imports:

```astro
import Button from '@components/Button.astro'; import {someFunction} from '@utils/helpers';
```

## After Creation

1. Test the component in isolation
2. Verify styling in both light and dark modes
3. Check responsive behavior
4. Test keyboard navigation
5. Verify accessibility with screen reader
