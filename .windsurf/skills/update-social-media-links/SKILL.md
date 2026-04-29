---
name: update-social-media-links
description: Updates all social media links to real URLs with proper accessibility and security attributes
---

# Update Social Media Links

This skill guides you through updating all social media links from placeholders to real URLs.

## Step 1: Identify All Social Media Link Locations

Search for social media links in the codebase:

```bash
grep -r "twitter\|facebook\|linkedin\|instagram\|youtube" src/ --include="*.astro" --include="*.tsx"
```

Common locations:

- `src/components/Footer.astro` - Footer social links
- `src/layouts/BaseLayout.astro` - Header social links
- `src/pages/team/[slug].astro` - Team member social links
- `src/pages/about.astro` - About page social links

## Step 2: Gather Real Social Media URLs

Collect the actual social media URLs for the business:

- Twitter/X: https://twitter.com/yourhandle
- Facebook: https://facebook.com/yourpage
- LinkedIn: https://linkedin.com/company/yourcompany
- Instagram: https://instagram.com/yourhandle
- YouTube: https://youtube.com/@yourchannel

If a platform is not used, remove the link entirely.

## Step 3: Update Footer.astro

### Before (Placeholder)

```astro
<a href="https://twitter.com" target="_blank" rel="noopener noreferrer">
  <TwitterIcon />
</a>
```

### After (Real URL)

```astro
<a
  href="https://twitter.com/yourhandle"
  target="_blank"
  rel="noopener noreferrer"
  aria-label="Follow us on Twitter"
>
  <TwitterIcon />
</a>
```

### Remove Unused Platforms

If a platform is not used, remove the entire link element:

```astro
<!-- Remove this section if not using Facebook --><!-- <a href="https://facebook.com/yourpage" target="_blank" rel="noopener noreferrer" aria-label="Follow us on Facebook">
  <FacebookIcon />
</a> -->
```

## Step 4: Update BaseLayout.astro

Check for social media links in the header or navigation. Update with real URLs and proper attributes.

## Step 5: Update Team Member Pages

### For Individual Team Members

If team members have personal social links, update them in `src/pages/team/[slug].astro`:

```astro
<a
  href="https://linkedin.com/in/teammember"
  target="_blank"
  rel="noopener noreferrer"
  aria-label="View Team Member's LinkedIn profile"
>
  <LinkedInIcon />
</a>
```

## Step 6: Update About Page

Check `src/pages/about.astro` for any social media links and update them.

## Accessibility Requirements

### ARIA Labels

All social media links must have descriptive `aria-label` attributes:

- "Follow us on Twitter"
- "Visit our Facebook page"
- "Connect on LinkedIn"
- "Follow our Instagram"
- "Watch our YouTube channel"

### Icon-Only Links

For icon-only links (no text), `aria-label` is required:

```astro
<a href="https://twitter.com/yourhandle" aria-label="Follow us on Twitter">
  <TwitterIcon />
</a>
```

### Text + Icon Links

If the link has text, ensure the text is descriptive:

```astro
<a href="https://twitter.com/yourhandle">
  <TwitterIcon />
  <span>Follow us on Twitter</span>
</a>
```

## Security Requirements

### External Link Attributes

All external social media links must have:

- `target="_blank"` - Opens in new tab
- `rel="noopener noreferrer"` - Security best practice

```astro
<a href="https://twitter.com/yourhandle" target="_blank" rel="noopener noreferrer">
  <TwitterIcon />
</a>
```

## Consistency Check

### Same URLs Across All Pages

Ensure the same social media URLs are used consistently:

- Company Twitter handle should be the same everywhere
- Company Facebook page should be the same everywhere
- Company LinkedIn should be the same everywhere

### Verify Links Work

After updating, test each link:

- Click each social media link
- Verify it opens the correct profile/page
- Check for 404 errors
- Verify the profile/page is active

## Common Patterns

### Social Link Component

Consider creating a reusable `SocialLink.astro` component:

```astro
---
interface Props {
  href: string;
  platform: string;
  icon: any;
}

const { href, platform, icon: Icon } = Astro.props;
---

<a
  href={href}
  target="_blank"
  rel="noopener noreferrer"
  aria-label={`Follow us on ${platform}`}
  class="social-link"
>
  <Icon />
</a>
```

Usage:

```astro
<SocialLink href="https://twitter.com/yourhandle" platform="Twitter" icon={TwitterIcon} />
```

## Testing

### Manual Testing

1. Click each social media link
2. Verify it opens in a new tab
3. Check that the correct profile/page loads
4. Verify the profile/page is active and current

### Accessibility Testing

1. Test with screen reader (NVDA, JAWS, VoiceOver)
2. Verify aria-labels are announced
3. Check keyboard navigation (Tab to each link)
4. Verify focus indicators are visible

### Link Validation

Use a link checker tool to verify:

- All links return 200 status
- No broken links
- No redirects to wrong pages

## Documentation

After updating:

1. Document the social media handles in README.md
2. Update any marketing materials
3. Inform team of the changes
4. Update TODO.md to mark TASK-007 as complete

## Example Footer Update

```astro
<footer>
  <div class="social-links">
    <a
      href="https://twitter.com/yourhandle"
      target="_blank"
      rel="noopener noreferrer"
      aria-label="Follow us on Twitter"
    >
      <TwitterIcon />
    </a>
    <a
      href="https://linkedin.com/company/yourcompany"
      target="_blank"
      rel="noopener noreferrer"
      aria-label="Connect on LinkedIn"
    >
      <LinkedInIcon />
    </a>
    <a
      href="https://instagram.com/yourhandle"
      target="_blank"
      rel="noopener noreferrer"
      aria-label="Follow us on Instagram"
    >
      <InstagramIcon />
    </a>
  </div>
</footer>
```

## Next Steps

After implementation:

1. Test all links in development
2. Deploy to production
3. Verify links work in production
4. Update documentation
5. Mark TASK-007 as complete in TODO.md
