---
trigger: always_on
---

<styling_guidelines>

# Styling Guidelines

## Tailwind CSS v4.0

This project uses Tailwind CSS v4.0 (released January 2025). Note the breaking changes from v3.

## Tailwind v4.0 Key Changes

### Configuration

- CSS-first configuration (no tailwind.config.js)
- Use `@theme` directive in CSS for theme customization
- Automatic content detection (no need to specify content paths)
- Built-in import support for CSS modules

### Theme Variables

- Use CSS custom properties for theme values
- Define colors, spacing, etc. in `@theme` block
- Dynamic utility values and variants
- Modernized P3 color palette

### New Features

- Container queries support
- New 3D transform utilities
- Expanded gradient APIs (linear angles, interpolation modifiers)
- `@starting-style` support for animations
- `not-*` variant for negation
- Enhanced gradient utilities (conic, radial)

## Color Usage

- **Primary Accent**: Use `electric-500` (#0055FF) for primary actions
- **Secondary Accent**: Use `electric-400` for hover states and highlights
- **Dark Background**: Use `dark-900` to `dark-700` for backgrounds
- **Light Background**: Use `light-DEFAULT` to `light-100` for light mode
- **Text**: Use `text-white` (dark) or `text-light-900` (light)

## Dark Mode Priority

- Always design for dark mode first
- Use `dark:` prefix for all dark mode styling
- Default to dark mode color schemes
- Ensure light mode contrast meets WCAG AA standards

## Custom Utilities

- `glass-card`: Glassmorphism effect with backdrop blur
- `focus-ring`: Custom focus outline with electric blue
- `grid-pattern`: Background grid pattern
- `shadow-neon-*`: Neon glow effects (sm, md, lg, xl)

## Tailwind Patterns

- Use utility classes for all styling
- Avoid custom CSS in component styles
- Use Tailwind's responsive prefixes (md:, lg:, xl:)
- Use state variants (hover:, focus:, active:)
- Use container queries for responsive components (v4.0)

## Animations

- `animate-float`: Gentle floating animation
- `animate-slide-up`: Slide up from bottom
- `animate-fade-in`: Fade in effect
- `animate-glow-pulse`: Pulsing glow effect
- `animate-glitch`: Glitch effect (use sparingly)
- Use `@starting-style` for entry animations (v4.0)

## Spacing

- Use Tailwind's spacing scale (4, 8, 12, 16, etc.)
- Maintain consistent spacing patterns
- Use gap utilities for flex/grid layouts
- Use padding/margin utilities consistently

## Typography

- **Headings**: `font-display` (Space Grotesk)
- **Body**: `font-sans` (Inter)
- Use semantic font sizes (text-xl, text-2xl, etc.)
- Use font weights: font-normal, font-semibold, font-bold

## Borders

- Use `border-light-200` for light mode
- Use `border-white/10` for dark mode
- Use `border-electric-500` for accent borders
- Use rounded utilities: rounded-lg, rounded-xl

## Shadows

- Use custom neon shadows for accent effects
- Use glass shadows for glassmorphism
- Avoid heavy shadows on dark backgrounds

## Migration from v3 to v4

- Remove tailwind.config.js (use CSS-first config)
- Update to new Vite plugin: `@tailwindcss/vite`
- Convert theme values to CSS custom properties
- Use `@theme` directive for customization
- Remove content configuration (automatic detection)

## Configuration Example (v4.0)

```css
@import "tailwindcss";

@theme {
  --color-electric-500: #0055ff;
  --color-electric-400: #3366ff;
  --color-dark-900: #0a0a0a;
  --color-dark-700: #1a1a1a;
  --color-light-DEFAULT: #ffffff;
  --color-light-100: #f5f5f5;
}
```

</styling_guidelines>
