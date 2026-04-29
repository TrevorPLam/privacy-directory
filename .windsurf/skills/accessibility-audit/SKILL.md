---
name: accessibility-audit
description: Performs comprehensive accessibility audit following WCAG 2.2 AA standards and ADA Title II compliance
---

# Accessibility Audit

This skill guides you through performing a comprehensive accessibility audit following WCAG 2.2 AA standards.

## WCAG 2.2 Updates (2026)

### New Success Criteria

- **Focus Appearance (2.4.11)**: Focus indicator must be visible and have sufficient contrast
- **Focus Not Obscured (2.4.12)**: Focus must not be hidden by other content
- **Dragging Movements (2.5.7)**: Alternative to dragging operations
- **Target Size (Minimum) (2.5.8)**: Interactive elements must be at least 24x24 CSS pixels
- **Redundant Entry (3.3.7)**: Help users avoid data entry errors
- **Authentication (3.3.8)**: Make authentication accessible

### ADA Title II Compliance (2026)

- **Deadline**: April 24, 2026 for public institutions
- **Standard**: WCAG 2.1 Level AA adopted by DOJ
- **Scope**: State and local government websites and mobile applications
- **Requirements**: Comply with both Level A and Level AA success criteria

## Step 1: Keyboard Navigation Testing

### 1.1 Tab Navigation

**Test:**

- Use Tab key to navigate through the page
- Verify focus moves in logical order
- Check that all interactive elements are reachable
- Ensure no keyboard traps

**Common Issues:**

- Elements not in tab order
- Focus moves to non-interactive elements
- Keyboard traps (can't exit a component)
- Skip link missing or not working

### 1.2 Focus Indicators

**Test:**

- Tab through the page and observe focus indicators
- Verify focus is clearly visible
- Check that focus-ring class is applied
- Test on different browsers

**Best Practices:**

- Use `focus-ring` custom class
- Ensure contrast meets WCAG AA
- Focus indicator should be at least 2px
- Visible on all backgrounds

### 1.3 Keyboard Shortcuts

**Test:**

- Enter key activates buttons and links
- Space key activates buttons
- Escape key closes modals and dropdowns
- Arrow keys navigate within components

## Step 2: Screen Reader Testing

### 2.1 Test with Screen Readers

**Tools:**

- NVDA (Windows, free)
- JAWS (Windows, paid)
- VoiceOver (macOS/iOS, built-in)
- TalkBack (Android, built-in)

### 2.2 Semantic HTML

**Check:**

- Proper use of HTML5 elements (header, nav, main, footer, article, section)
- Heading hierarchy (h1 → h2 → h3)
- No skipped heading levels
- Landmark roles when appropriate

**Example:**

```astro
<header>
  <nav aria-label="Main navigation">
    <ul>
      <li><a href="/">Home</a></li>
    </ul>
  </nav>
</header>
<main>
  <h1>Page Title</h1>
  <section>
    <h2>Section Title</h2>
  </section>
</main>
<footer>
  <p>Footer content</p>
</footer>
```

### 2.3 ARIA Attributes

**Check:**

- `aria-label` for buttons without text
- `aria-describedby` for form error messages
- `aria-invalid` for form validation errors
- `aria-live` for dynamic content updates
- `aria-expanded` for collapsible content
- `aria-hidden="true"` for decorative elements

**Example:**

```astro
<button aria-label="Close modal" onClick={closeModal}>
  <CloseIcon />
</button>

<input id="email" aria-invalid={hasError} aria-describedby="email-error" />

{
  hasError && (
    <p id="email-error" role="alert">
      Please enter a valid email
    </p>
  )
}
```

### 2.4 Alt Text

**Check:**

- All images have alt text
- Alt text is descriptive and meaningful
- Empty alt text for decorative images: `alt=""`
- Complex images have text alternatives
- Icon-only buttons have aria-label

**Example:**

```astro
<!-- Descriptive image -->
<Image src={heroImage} alt="Team collaborating on SEO strategy" />

<!-- Decorative image -->
<Image src={decorativeImage} alt="" aria-hidden="true" />

<!-- Icon-only button -->
<button aria-label="Search">
  <SearchIcon />
</button>
```

## Step 3: Color Contrast Testing

### 3.1 Test Color Contrast

**Tools:**

- WebAIM Contrast Checker
- Chrome DevTools Lighthouse
- axe DevTools
- Colour Contrast Analyser

**WCAG AA Standards:**

- Normal text: 4.5:1 contrast ratio
- Large text (18pt+): 3:1 contrast ratio
- UI components: 3:1 contrast ratio
- Graphics: 3:1 contrast ratio

**Test:**

- Test all text colors against backgrounds
- Test in both light and dark modes
- Test focus indicators
- Test borders and outlines

### 3.2 Don't Rely on Color Alone

**Check:**

- Color is not the only way to convey information
- Use icons, text, or patterns in addition to color
- Error states have text indicators
- Links are distinguished by more than color

**Example:**

```astro
<!-- Bad - color only -->
<span class="error">Error</span>

<!-- Good - color + icon + text -->
<span class="error">
  <ErrorIcon />
  Error: Invalid input
</span>
```

## Step 4: Form Accessibility

### 4.1 Label Association

**Check:**

- All form inputs have associated labels
- Labels use `htmlFor` and `id`
- Required fields are indicated
- Error messages are linked to inputs

**Example:**

```astro
<label htmlFor="email">Email Address</label>
<input id="email" type="email" required aria-invalid={hasError} aria-describedby="email-error" />
{
  hasError && (
    <p id="email-error" class="error-message">
      Please enter a valid email address
    </p>
  )
}
```

### 4.2 Input Types

**Check:**

- Use proper input types (email, tel, url, etc.)
- Use `required` attribute for required fields
- Use `placeholder` for hints (not as replacement for labels)
- Use `autocomplete` where appropriate

### 4.3 Form Validation

**Check:**

- Validation errors are announced to screen readers
- Error messages are clear and specific
- Users can fix errors without losing data
- Form can be submitted with keyboard

## Step 5: Focus Management

### 5.1 Modals and Dialogs

**Check:**

- Focus moves to modal when opened
- Focus is trapped within modal
- Focus returns to trigger when closed
- Escape key closes modal

**Example:**

```tsx
useEffect(() => {
  if (isOpen) {
    // Focus on first focusable element
    modalRef.current?.focus();
    // Trap focus
    const handleTab = (e: KeyboardEvent) => {
      if (e.key === "Tab") {
        // Trap focus logic
      }
    };
    document.addEventListener("keydown", handleTab);
    return () => document.removeEventListener("keydown", handleTab);
  }
}, [isOpen]);
```

### 5.2 Dynamic Content

**Check:**

- Dynamic content updates are announced
- Use `aria-live` for important updates
- Use `role="alert"` for errors
- Use `role="status"` for status updates

**Example:**

```astro
<div aria-live="polite" aria-atomic="true">
  {statusMessage}
</div>
```

## Step 6: Skip Links

### 6.1 Skip Link Implementation

**Check:**

- Skip link at top of page
- Visible on focus
- Links to main content with `#main`
- Works across all pages

**Example:**

```astro
<a href="#main" class="skip-link">Skip to main content</a>
<main id="main">
  <!-- Main content -->
</main>
```

**CSS:**

```css
.skip-link {
  position: absolute;
  top: -40px;
  left: 0;
  background: electric-500;
  color: white;
  padding: 8px;
  z-index: 100;
}

.skip-link:focus {
  top: 0;
}
```

## Step 7: Testing Tools

### 7.1 Automated Testing

**Tools:**

- axe DevTools (Chrome extension)
- Lighthouse accessibility audit
- WAVE (Web Accessibility Evaluation Tool)
- Pa11y (command line)

**Run Lighthouse:**

```bash
npx lighthouse https://yoursite.com --view --only-categories=accessibility
```

### 7.2 Manual Testing Checklist

- [ ] Keyboard navigation works
- [ ] Screen reader announces content correctly
- [ ] Color contrast meets WCAG AA
- [ ] All images have alt text
- [ ] Forms have proper labels
- [ ] Focus management works in modals
- [ ] Skip link is present and functional
- [ ] Dynamic content is announced
- [ ] No keyboard traps
- [ ] Focus indicators are visible

## Step 8: Generate Accessibility Report

### Report Structure

1. **Executive Summary**
   - Overall accessibility score
   - Critical issues
   - Quick wins

2. **Keyboard Navigation**
   - Tab order issues
   - Focus indicator problems
   - Keyboard traps

3. **Screen Reader**
   - Semantic HTML issues
   - ARIA attribute problems
   - Alt text issues

4. **Visual Accessibility**
   - Color contrast failures
   - Color-only information
   - Text readability

5. **Forms**
   - Label association issues
   - Validation problems
   - Error handling

6. **Recommendations**
   - Priority issues to fix
   - Quick wins
   - Long-term improvements

## Common Issues and Fixes

### Missing Alt Text

**Fix:** Add descriptive alt text to all images

```astro
<Image src={image} alt="Descriptive text" />
```

### Poor Color Contrast

**Fix:** Increase contrast or change colors

```css
/* Bad - low contrast */
color: #999;
background: #fff;

/* Good - high contrast */
color: #333;
background: #fff;
```

### Missing Labels

**Fix:** Add labels to all form inputs

```astro
<label htmlFor="input">Label</label>
<input id="input" />
```

### Keyboard Trap

**Fix:** Ensure focus can escape

```tsx
// Add escape key handler
useEffect(() => {
  const handleEscape = (e: KeyboardEvent) => {
    if (e.key === "Escape") onClose();
  };
  document.addEventListener("keydown", handleEscape);
  return () => document.removeEventListener("keydown", handleEscape);
}, [onClose]);
```

## Next Steps

After audit:

1. Fix critical issues first
2. Implement quick wins
3. Plan long-term improvements
4. Re-audit after changes
5. Monitor with automated tools
