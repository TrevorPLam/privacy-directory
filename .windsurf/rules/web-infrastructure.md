---
trigger: glob
globs: **/*.astro,**/*.tsx,**/src/web/**
---

# Web Infrastructure Rules

## Astro Static Site Architecture

### Page Structure
- **Static Pages**: Use `.astro` files for static content
- **Dynamic Routes**: Use `[slug].astro` for entity pages
- **API Routes**: Use `src/pages/api/` for serverless functions
- **Layout Components**: Use `src/layouts/` for shared layouts

### Component Organization
```
src/web/
├── pages/              # Astro pages
│   ├── index.astro     # Homepage
│   ├── companies/      # Company pages
│   ├── data-brokers/   # Data broker pages
│   └── incidents/      # Incident pages
├── components/         # React components
│   ├── ui/            # Reusable UI components
│   ├── forms/         # Form components
│   └── interactive/   # Interactive components
├── layouts/           # Layout components
└── styles/           # Global styles
```

## Navigation and Routing

### Global Navigation Requirements
```typescript
// src/web/components/navigation.tsx
interface NavigationItem {
  title: string;
  href: string;
  description?: string;
  children?: NavigationItem[];
}

const navigationStructure: NavigationItem[] = [
  {
    title: 'Technology Directory',
    href: '/technology',
    children: [
      { title: 'Companies', href: '/companies' },
      { title: 'Software', href: '/software' },
      { title: 'Devices', href: '/devices' }
    ]
  },
  {
    title: 'Privacy Incidents',
    href: '/incidents'
  },
  {
    title: 'Data Broker Hub',
    href: '/data-brokers'
  },
  {
    title: 'Privacy Toolkit',
    href: '/toolkit'
  },
  {
    title: 'Legal Guide',
    href: '/legal'
  },
  {
    title: 'Interactive Tools',
    href: '/tools'
  }
];
```

### Cross-Entity Linking
```typescript
// src/web/components/related-links.tsx
interface RelatedLinksProps {
  entityType: 'company' | 'software' | 'device' | 'incident' | 'broker';
  entityId: string;
}

export function RelatedLinks({ entityType, entityId }: RelatedLinksProps) {
  const relatedEntities = getRelatedEntities(entityType, entityId);
  
  return (
    <div className="related-links">
      <h3>Related Information</h3>
      <ul>
        {relatedEntities.map(entity => (
          <li key={entity.id}>
            <Link href={`/${entity.type}/${entity.slug}`}>
              {entity.name}
            </Link>
            <span className="entity-type">{entity.type}</span>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

## Component Standards

### React Component Patterns
```typescript
// src/web/components/ui/card.tsx
interface CardProps {
  title: string;
  description?: string;
  children?: React.ReactNode;
  className?: string;
  variant?: 'default' | 'bordered' | 'elevated';
}

export function Card({ title, description, children, className, variant = 'default' }: CardProps) {
  const baseClasses = 'rounded-lg p-6 transition-shadow';
  const variantClasses = {
    default: 'bg-white shadow-sm',
    bordered: 'bg-white border border-gray-200',
    elevated: 'bg-white shadow-md'
  };

  return (
    <div className={`${baseClasses} ${variantClasses[variant]} ${className || ''}`}>
      <div className="mb-4">
        <h3 className="text-lg font-semibold">{title}</h3>
        {description && (
          <p className="text-gray-600 mt-1">{description}</p>
        )}
      </div>
      {children}
    </div>
  );
}
```

### Form Components
```typescript
// src/web/components/forms/contact-form.tsx
interface ContactFormData {
  name: string;
  email: string;
  subject: string;
  message: string;
}

export function ContactForm() {
  const [formData, setFormData] = useState<ContactFormData>({
    name: '',
    email: '',
    subject: '',
    message: ''
  });
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [submitStatus, setSubmitStatus] = useState<'idle' | 'success' | 'error'>('idle');

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setIsSubmitting(true);

    try {
      // Client-side only submission in v1
      await new Promise(resolve => setTimeout(resolve, 1000));
      setSubmitStatus('success');
      setFormData({ name: '', email: '', subject: '', message: '' });
    } catch (error) {
      setSubmitStatus('error');
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <div>
        <label htmlFor="name" className="block text-sm font-medium text-gray-700">
          Name
        </label>
        <input
          type="text"
          id="name"
          required
          value={formData.name}
          onChange={(e) => setFormData({ ...formData, name: e.target.value })}
          className="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500"
        />
      </div>

      {/* Additional form fields */}

      <button
        type="submit"
        disabled={isSubmitting}
        className="w-full bg-blue-600 text-white py-2 px-4 rounded-md hover:bg-blue-700 disabled:opacity-50"
      >
        {isSubmitting ? 'Sending...' : 'Send Message'}
      </button>

      {submitStatus === 'success' && (
        <div className="bg-green-50 text-green-800 p-3 rounded-md">
          Message sent successfully!
        </div>
      )}

      {submitStatus === 'error' && (
        <div className="bg-red-50 text-red-800 p-3 rounded-md">
          Failed to send message. Please try again.
        </div>
      )}
    </form>
  );
}
```

## Static Page Requirements

### Methodology Page
```astro
---
// src/web/pages/methodology.astro
import Layout from '../layouts/main.astro';
---

<Layout title="Methodology - Privacy Directory">
  <main class="container mx-auto px-4 py-8">
    <h1>Our Methodology</h1>
    
    <section class="mb-8">
      <h2>Data Collection</h2>
      <p>We collect information from verifiable sources including:</p>
      <ul>
        <li>Official company privacy policies</li>
        <li>Regulatory filings and enforcement actions</li>
        <li>Government agency databases</li>
        <li>Established news sources with editorial standards</li>
      </ul>
    </section>

    <section class="mb-8">
      <h2>Fact-Checking Process</h2>
      <p>Every piece of information undergoes our verification process:</p>
      <ol>
        <li>Source verification and reliability assessment</li>
        <li>Cross-reference with independent sources</li>
        <li>Context validation and accuracy confirmation</li>
        <li>Regular quarterly reviews and updates</li>
      </ol>
    </section>

    <section class="mb-8">
      <h2>Neutrality Commitment</h2>
      <p>We present factual information without subjective judgments. Our content focuses on:</p>
      <ul>
        <li>Verifiable data practices and policies</li>
        <li>Documented incidents and regulatory actions</li>
        <li>Factual descriptions of opt-out processes</li>
        <li>Objective compliance status information</li>
      </ul>
    </section>
  </main>
</Layout>
```

### Sitemap Generation
```typescript
// src/web/pages/sitemap.xml.js
export async function GET() {
  const site = 'https://privacy-directory.com';
  
  // Static pages
  const staticPages = [
    { url: '/', changefreq: 'daily', priority: 1.0 },
    { url: '/about', changefreq: 'monthly', priority: 0.8 },
    { url: '/methodology', changefreq: 'monthly', priority: 0.8 },
    { url: '/privacy', changefreq: 'monthly', priority: 0.8 },
    { url: '/contact', changefreq: 'monthly', priority: 0.7 },
    { url: '/companies', changefreq: 'weekly', priority: 0.9 },
    { url: '/software', changefreq: 'weekly', priority: 0.9 },
    { url: '/devices', changefreq: 'weekly', priority: 0.9 },
    { url: '/incidents', changefreq: 'daily', priority: 0.9 },
    { url: '/data-brokers', changefreq: 'weekly', priority: 0.9 }
  ];

  // Dynamic entity pages
  const companies = await getCompanies();
  const software = await getSoftware();
  const devices = await getDevices();
  const incidents = await getIncidents();
  const brokers = await getDataBrokers();

  const entityPages = [
    ...companies.map(company => ({
      url: `/companies/${company.slug}`,
      changefreq: 'monthly',
      priority: 0.8,
      lastmod: company.updatedAt
    })),
    ...software.map(item => ({
      url: `/software/${item.slug}`,
      changefreq: 'monthly',
      priority: 0.8,
      lastmod: item.updatedAt
    })),
    ...devices.map(item => ({
      url: `/devices/${item.slug}`,
      changefreq: 'monthly',
      priority: 0.8,
      lastmod: item.updatedAt
    })),
    ...incidents.map(incident => ({
      url: `/incidents/${incident.slug}`,
      changefreq: 'monthly',
      priority: 0.8,
      lastmod: incident.updatedAt
    })),
    ...brokers.map(broker => ({
      url: `/data-brokers/${broker.slug}`,
      changefreq: 'monthly',
      priority: 0.8,
      lastmod: broker.updatedAt
    }))
  ];

  const allPages = [...staticPages, ...entityPages];

  const sitemap = `<?xml version="1.0" encoding="UTF-8"?>
    <urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
      ${allPages.map(page => `
        <url>
          <loc>${site}${page.url}</loc>
          <lastmod>${page.lastmod || new Date().toISOString()}</lastmod>
          <changefreq>${page.changefreq}</changefreq>
          <priority>${page.priority}</priority>
        </url>
      `).join('')}
    </urlset>`;

  return new Response(sitemap, {
    headers: {
      'Content-Type': 'application/xml',
      'Cache-Control': 'public, max-age=3600'
    }
  });
}
```

## Performance Optimization

### Image Optimization
```astro
---
import { Image } from '@astrojs/image/components';
---

<Image
  src="/images/company-logos/example.png"
  alt="Company Logo"
  width={120}
  height={120}
  format="webp"
  loading="lazy"
  class="company-logo"
/>
```

### Code Splitting
```typescript
// src/web/components/interactive/lazy-component.tsx
import { lazy } from 'react';

const HeavyComponent = lazy(() => import('./heavy-component'));

export function LazyWrapper() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <HeavyComponent />
    </Suspense>
  );
}
```

### Caching Strategy
```typescript
// src/web/lib/cache.ts
export class StaticCache {
  private cache = new Map<string, { data: any; timestamp: number }>();
  private ttl = 3600000; // 1 hour

  get(key: string) {
    const item = this.cache.get(key);
    if (!item) return null;
    
    if (Date.now() - item.timestamp > this.ttl) {
      this.cache.delete(key);
      return null;
    }
    
    return item.data;
  }

  set(key: string, data: any) {
    this.cache.set(key, {
      data,
      timestamp: Date.now()
    });
  }
}
```

## Accessibility Standards

### WCAG Compliance
```typescript
// src/web/components/ui/accessible-button.tsx
interface AccessibleButtonProps {
  children: React.ReactNode;
  onClick: () => void;
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
  ariaLabel?: string;
}

export function AccessibleButton({ 
  children, 
  onClick, 
  variant = 'primary', 
  disabled = false,
  ariaLabel 
}: AccessibleButtonProps) {
  const baseClasses = 'px-4 py-2 rounded-md font-medium transition-colors';
  const variantClasses = {
    primary: 'bg-blue-600 text-white hover:bg-blue-700',
    secondary: 'bg-gray-200 text-gray-900 hover:bg-gray-300'
  };

  return (
    <button
      onClick={onClick}
      disabled={disabled}
      aria-label={ariaLabel}
      className={`${baseClasses} ${variantClasses[variant]} ${
        disabled ? 'opacity-50 cursor-not-allowed' : ''
      }`}
    >
      {children}
    </button>
  );
}
```

### Semantic HTML
```astro
---
// src/web/pages/companies/index.astro
import Layout from '../../layouts/main.astro';
import CompanyCard from '../../components/company-card.astro';
---

<Layout title="Companies - Privacy Directory">
  <header>
    <h1>Privacy-Related Companies</h1>
    <p>Factual directory of companies and their data practices</p>
  </header>

  <main>
    <section aria-label="Company Directory">
      <h2>Companies ({companies.length})</h2>
      <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
        {companies.map(company => (
          <CompanyCard company={company} />
        ))}
      </div>
    </section>
  </main>

  <aside aria-label="Filters">
    <h2>Filter Companies</h2>
    <form>
      {/* Filter controls */}
    </form>
  </aside>
</Layout>
```

## Error Handling

### 404 Page
```astro
---
// src/web/pages/404.astro
import Layout from '../layouts/main.astro';
---

<Layout title="Page Not Found - Privacy Directory">
  <main class="container mx-auto px-4 py-8 text-center">
    <h1 class="text-4xl font-bold mb-4">Page Not Found</h1>
    <p class="text-lg mb-8">
      The page you're looking for doesn't exist or has been moved.
    </p>
    <a href="/" class="bg-blue-600 text-white px-6 py-3 rounded-md hover:bg-blue-700">
      Return to Homepage
    </a>
  </main>
</Layout>
```

### Error Boundaries
```typescript
// src/web/components/error-boundary.tsx
interface ErrorBoundaryState {
  hasError: boolean;
  error?: Error;
}

export class ErrorBoundary extends Component<
  { children: React.ReactNode },
  ErrorBoundaryState
> {
  constructor(props: { children: React.ReactNode }) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error: Error): ErrorBoundaryState {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        <div class="bg-red-50 border border-red-200 rounded-lg p-6">
          <h2 class="text-red-800 font-bold mb-2">Something went wrong</h2>
          <p class="text-red-600 mb-4">
            An error occurred while rendering this component.
          </p>
          <button
            onClick={() => this.setState({ hasError: false })}
            class="bg-red-600 text-white px-4 py-2 rounded hover:bg-red-700"
          >
            Try Again
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}
```

## Testing Requirements

### Component Testing
```typescript
// src/web/components/__tests__/navigation.test.tsx
import { render, screen } from '@testing-library/react';
import { Navigation } from '../navigation';

describe('Navigation', () => {
  it('should render all main navigation items', () => {
    render(<Navigation />);
    
    expect(screen.getByText('Technology Directory')).toBeInTheDocument();
    expect(screen.getByText('Privacy Incidents')).toBeInTheDocument();
    expect(screen.getByText('Data Broker Hub')).toBeInTheDocument();
  });

  it('should be keyboard accessible', () => {
    render(<Navigation />);
    
    const firstNavItem = screen.getByText('Technology Directory');
    firstNavItem.focus();
    
    expect(document.activeElement).toBe(firstNavItem);
  });
});
```

### Page Testing
```typescript
// src/web/pages/__tests__/companies.test.tsx
import { render, screen } from '@testing-library/react';
import CompaniesPage from '../companies/index.astro';

describe('Companies Page', () => {
  it('should render page title and description', async () => {
    const { getByText } = render(await CompaniesPage());
    
    expect(getByText('Privacy-Related Companies')).toBeInTheDocument();
    expect(getByText('Factual directory of companies and their data practices')).toBeInTheDocument();
  });

  it('should render company cards', async () => {
    const { getAllByTestId } = render(await CompaniesPage());
    
    const companyCards = getAllByTestId('company-card');
    expect(companyCards.length).toBeGreaterThan(0);
  });
});
```

## Deployment Configuration

### Build Optimization
```javascript
// astro.config.mjs
export default defineConfig({
  output: 'static',
  build: {
    format: 'directory'
  },
  vite: {
    build: {
      rollupOptions: {
        output: {
          manualChunks: {
            vendor: ['react', 'react-dom'],
            ui: ['@headlessui/react', '@heroicons/react']
          }
        }
      }
    }
  }
});
```

### Environment Variables
```typescript
// src/web/lib/config.ts
export const config = {
  siteUrl: import.meta.env.SITE_URL || 'https://privacy-directory.com',
  analyticsId: import.meta.env.PLAUSIBLE_ANALYTICS_ID,
  apiBaseUrl: import.meta.env.PUBLIC_API_BASE_URL || '/api',
  isDev: import.meta.env.DEV
};
```

## Verification Checklist

### Structure Compliance
- [ ] All pages use proper semantic HTML
- [ ] Navigation follows accessibility guidelines
- [ ] Components are properly organized
- [ ] Error handling implemented

### Performance
- [ ] Images optimized with WebP format
- [ ] Code splitting implemented for heavy components
- [ ] Caching strategy in place
- [ ] Bundle size optimized

### Accessibility
- [ ] WCAG 2.1 AA compliance
- [ ] Keyboard navigation support
- [ ] Screen reader compatibility
- [ ] Color contrast ratios met

### SEO
- [ ] Meta tags properly configured
- [ ] Sitemap automatically generated
- [ ] Structured data implemented
- [ ] Internal linking strategy

This ensures the web infrastructure follows modern best practices for performance, accessibility, and maintainability.
