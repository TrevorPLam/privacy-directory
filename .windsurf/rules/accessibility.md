---
trigger: always_on
---

<accessibility>

# Accessibility Standards

## WCAG Standards

### Current Standard: WCAG 2.2

- WCAG 2.2 is the current approved standard (ISO/IEC 40500:2025)
- WCAG 2.2 does not deprecate WCAG 2.1 or 2.0
- Content meeting WCAG 2.2 also meets WCAG 2.1 and 2.0
- W3C encourages using the latest version (WCAG 2.2)

### WCAG 3.0 Status

- WCAG 3.0 is in development (working drafts published March 2026)
- WCAG 3.0 will not supersede WCAG 2 for several years
- WCAG 2 will not be deprecated when WCAG 3.0 is released
- Continue following WCAG 2.2 AA standards for now
- Monitor WCAG 3.0 progress for future adoption

### Legal Compliance

- ADA 2026 requirements enforce WCAG 2.1 AA as benchmark
- European Accessibility Act (EAA) uses WCAG 2.1 (expected to update to 2.2)
- Courts consistently apply WCAG 2.1 AA in accessibility lawsuits
- Over 5,000 digital accessibility lawsuits filed in 2025 (37% increase)

## Semantic HTML

- Use proper HTML5 elements: header, nav, main, footer, article, section, aside
- Maintain heading hierarchy (h1 → h2 → h3)
- Use skip links for keyboard navigation
- Use landmark roles when appropriate

## Keyboard Navigation

- Ensure all interactive elements are keyboard accessible
- Use proper focus indicators (focus-ring class)
- Support Tab navigation order
- Provide keyboard alternatives for mouse actions

## ARIA Attributes

- Use `aria-label` for buttons without text
- Use `aria-describedby` for form error messages
- Use `aria-invalid` for form validation errors
- Use `aria-live` for dynamic content updates
- Use `aria-expanded` for collapsible content

## Form Accessibility

- Associate labels with inputs using `htmlFor` and `id`
- Provide required field indicators
- Include error messages linked to inputs
- Use proper input types (email, tel, url, etc.)
- Support form validation with ARIA

## Images

- Always include alt text for images
- Use empty alt text for decorative images: `alt=""`
- Provide text alternatives for complex images
- Use aria-label for icon-only buttons

## Color Contrast

- Ensure text contrast meets WCAG AA (4.5:1 for normal text)
- Ensure large text contrast meets WCAG AA (3:1 for 18pt+)
- Test color combinations in both light and dark modes
- Don't rely on color alone to convey information

## Screen Readers

- Test with screen reader software
- Provide meaningful link text (avoid "click here")
- Use heading levels correctly
- Announce dynamic content changes
- Hide decorative elements with `aria-hidden="true"`

## Focus Management

- Ensure visible focus indicators
- Manage focus in modals and dialogs
- Return focus after closing overlays
- Support focus trapping in modals

## Testing

- Test with keyboard only
- Test with screen reader (NVDA, JAWS, VoiceOver)
- Test with browser zoom (200%)
- Test color contrast with tools
- Test with mobile devices

## Skip Links

- Include skip link at top of page
- Make skip link visible on focus
- Link to main content with `#main`
- Ensure skip link works across pages

</accessibility>
