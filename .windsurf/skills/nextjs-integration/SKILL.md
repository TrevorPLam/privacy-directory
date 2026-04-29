---
name: nextjs-integration
description: Complete Next.js integration for interactive privacy tools with proper separation from Astro static site
---

# Next.js Integration Skill

## Overview
This skill provides comprehensive guidance for integrating Next.js as a sub-application for interactive privacy tools while maintaining proper separation from the main Astro static site. Includes opt-out wizard, data broker visualizer, and other dynamic features.

## When to Use
- Building interactive opt-out wizards for data brokers
- Creating data broker visualizations and dashboards
- Implementing dynamic search and filtering tools
- Adding user interaction features that require server-side rendering
- Building complex forms that need backend processing

## Architecture Principles

### Separation of Concerns
```
Privacy Directory/
├── src/                    # Astro static site
│   ├── web/               # Static pages and components
│   └── domain/            # Domain models (shared)
├── src/nextjs/            # Next.js sub-application
│   ├── app/              # Next.js App Router
│   ├── components/       # Interactive components
│   ├── lib/              # Shared utilities
│   └── types/            # Type definitions
└── shared/               # Shared DTOs and interfaces
    ├── dto.ts            # Data transfer objects
    └── types.ts          # Common types
```

### Communication Patterns
- **REST APIs**: Next.js exposes APIs for Astro consumption
- **Shared DTOs**: Common data structures in `shared/` directory
- **No Direct Imports**: Astro never imports Next.js code
- **Environment Variables**: Separate configs for each application

## Implementation Steps

### 1. Next.js Project Setup
Initialize Next.js sub-application:

```bash
# Create Next.js app in subdirectory
npx create-next-app@latest src/nextjs --typescript --tailwind --eslint --app --src-dir --import-alias "@/*"

# Configure package.json scripts
{
  "scripts": {
    "next:dev": "cd src/nextjs && next dev",
    "next:build": "cd src/nextjs && next build",
    "next:start": "cd src/nextjs && next start",
    "next:lint": "cd src/nextjs && next lint"
  }
}
```

### 2. Shared Type Definitions
Create shared DTOs and types:

```typescript
// shared/dto.ts
export interface CompanyDto {
  id: string;
  name: string;
  url: string;
  description: string;
  practices: PracticeDto[];
  dataSensitivityTier: DataSensitivityTier;
}

export interface PracticeDto {
  category: string;
  details: string;
}

export interface DataBrokerDto {
  id: string;
  name: string;
  url: string;
  optOutProcess: OptOutProcessDto;
  complianceStatus: ComplianceStatusDto;
}

export interface OptOutProcessDto {
  type: 'WEB_FORM' | 'EMAIL' | 'API' | 'PORTAL';
  instructions: string;
  url?: string;
  email?: string;
  estimatedTime: string;
}

// API Response types
export interface ApiResponse<T> {
  data: T;
  success: boolean;
  message?: string;
}

export interface PaginatedResponse<T> extends ApiResponse<T[]> {
  pagination: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
  };
}
```

### 3. Next.js API Routes
Create API endpoints for data access:

```typescript
// src/nextjs/app/api/companies/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { CompanyDto, ApiResponse } from '../../../../shared/dto';

export async function GET(request: NextRequest) {
  try {
    const { searchParams } = new URL(request.url);
    const page = parseInt(searchParams.get('page') || '1');
    const limit = parseInt(searchParams.get('limit') || '20');
    const search = searchParams.get('search') || '';

    // Fetch from domain layer (implement actual data access)
    const companies = await getCompaniesPaginated(page, limit, search);
    
    const response: ApiResponse<CompanyDto[]> = {
      data: companies,
      success: true
    };

    return NextResponse.json(response);
  } catch (error) {
    const response: ApiResponse<CompanyDto[]> = {
      data: [],
      success: false,
      message: 'Failed to fetch companies'
    };

    return NextResponse.json(response, { status: 500 });
  }
}

// src/nextjs/app/api/data-brokers/[id]/opt-out/route.ts
export async function POST(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    const body = await request.json();
    const { userInfo, optOutMethod } = body;

    // Process opt-out request (implement actual logic)
    const result = await processOptOutRequest(params.id, userInfo, optOutMethod);

    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    return NextResponse.json({
      success: false,
      message: 'Opt-out request failed'
    }, { status: 500 });
  }
}
```

### 4. Interactive Components
Create reusable interactive components:

```typescript
// src/nextjs/components/OptOutWizard.tsx
'use client';

import { useState } from 'react';
import { DataBrokerDto, OptOutProcessDto } from '../../../shared/dto';

interface OptOutWizardProps {
  broker: DataBrokerDto;
  onComplete: (result: OptOutResult) => void;
}

export function OptOutWizard({ broker, onComplete }: OptOutWizardProps) {
  const [currentStep, setCurrentStep] = useState(0);
  const [userInfo, setUserInfo] = useState<UserInfo>({
    firstName: '',
    lastName: '',
    email: '',
    phone: ''
  });
  const [selectedMethod, setSelectedMethod] = useState<OptOutMethod | null>(null);

  const steps = [
    { title: 'Verify Identity', component: IdentityVerification },
    { title: 'Choose Method', component: OptOutMethodSelection },
    { title: 'Complete Process', component: ProcessCompletion }
  ];

  const CurrentStepComponent = steps[currentStep].component;

  return (
    <div className="max-w-2xl mx-auto p-6">
      <div className="mb-8">
        <h2 className="text-2xl font-bold mb-2">
          Opt Out from {broker.name}
        </h2>
        <div className="flex items-center space-x-2">
          {steps.map((step, index) => (
            <div
              key={index}
              className={`flex items-center ${
                index <= currentStep ? 'text-blue-600' : 'text-gray-400'
              }`}
            >
              <div className={`w-8 h-8 rounded-full flex items-center justify-center ${
                index <= currentStep ? 'bg-blue-600 text-white' : 'bg-gray-200'
              }`}>
                {index + 1}
              </div>
              {index < steps.length - 1 && (
                <div className={`w-16 h-1 ${
                  index < currentStep ? 'bg-blue-600' : 'bg-gray-200'
                }`} />
              )}
            </div>
          ))}
        </div>
      </div>

      <CurrentStepComponent
        broker={broker}
        userInfo={userInfo}
        onUserInfoChange={setUserInfo}
        selectedMethod={selectedMethod}
        onMethodSelect={setSelectedMethod}
        onNext={() => setCurrentStep(currentStep + 1)}
        onPrevious={() => setCurrentStep(currentStep - 1)}
        onComplete={onComplete}
      />
    </div>
  );
}
```

### 5. Data Broker Visualizer
Create interactive visualization components:

```typescript
// src/nextjs/components/DataBrokerVisualizer.tsx
'use client';

import { useMemo } from 'react';
import { DataBrokerDto } from '../../../shared/dto';

interface DataBrokerVisualizerProps {
  brokers: DataBrokerDto[];
  selectedCategory?: string;
}

export function DataBrokerVisualizer({ brokers, selectedCategory }: DataBrokerVisualizerProps) {
  const visualizationData = useMemo(() => {
    const filtered = selectedCategory
      ? brokers.filter(b => b.category === selectedCategory)
      : brokers;

    return {
      totalBrokers: filtered.length,
      complianceRate: calculateComplianceRate(filtered),
      optOutMethods: groupByOptOutMethod(filtered),
      processingTimes: getProcessingTimeStats(filtered)
    };
  }, [brokers, selectedCategory]);

  return (
    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
      <MetricCard
        title="Total Brokers"
        value={visualizationData.totalBrokers}
        icon={<BuildingIcon />}
      />
      <MetricCard
        title="Compliance Rate"
        value={`${visualizationData.complianceRate}%`}
        icon={<CheckCircleIcon />}
      />
      <MetricCard
        title="Easy Opt-Out"
        value={visualizationData.optOutMethods.easy}
        icon={<ThumbsUpIcon />}
      />
      <MetricCard
        title="Avg Processing Time"
        value={visualizationData.processingTimes.average}
        icon={<ClockIcon />}
      />
    </div>
  );
}

function MetricCard({ title, value, icon }: MetricCardProps) {
  return (
    <div className="bg-white rounded-lg shadow p-6">
      <div className="flex items-center justify-between">
        <div>
          <p className="text-sm font-medium text-gray-600">{title}</p>
          <p className="text-2xl font-bold text-gray-900">{value}</p>
        </div>
        <div className="text-blue-600">
          {icon}
        </div>
      </div>
    </div>
  );
}
```

### 6. Integration with Astro
Create bridge components for Astro integration:

```typescript
// src/web/components/interactive-opt-out.tsx
interface InteractiveOptOutProps {
  brokerId: string;
  brokerName: string;
}

export function InteractiveOptOut({ brokerId, brokerName }: InteractiveOptOutProps) {
  return (
    <div className="interactive-opt-out-container">
      <iframe
        src={`/nextjs/opt-out-wizard?broker=${brokerId}`}
        className="w-full h-96 border-0 rounded-lg"
        title={`Opt out from ${brokerName}`}
      />
      <noscript>
        <div className="bg-yellow-50 border border-yellow-200 rounded-lg p-4">
          <p className="text-sm text-yellow-800">
            Please enable JavaScript to use the interactive opt-out wizard.
            <a href={`/data-brokers/${brokerId}`} className="underline ml-1">
              View manual opt-out instructions
            </a>
          </p>
        </div>
      </noscript>
    </div>
  );
}
```

### 7. Deployment Configuration
Configure separate deployment:

```javascript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  output: 'standalone',
  experimental: {
    serverComponentsExternalPackages: ['@cloudflare/workers-types']
  },
  env: {
    CUSTOM_DOMAIN: process.env.NEXTJS_CUSTOM_DOMAIN,
    API_BASE_URL: process.env.NEXTJS_API_BASE_URL
  },
  async rewrites() {
    return [
      {
        source: '/api/:path*',
        destination: `${process.env.API_BASE_URL}/api/:path*`
      }
    ];
  }
};

module.exports = nextConfig;
```

## Environment Configuration

### Development Environment
```bash
# .env.local
NEXT_PUBLIC_API_BASE_URL=http://localhost:3001
NEXT_PUBLIC_ASTRO_BASE_URL=http://localhost:4321
DATABASE_URL=postgresql://localhost/privacy_directory
```

### Production Environment
```bash
# Production variables
NEXT_PUBLIC_API_BASE_URL=https://api.privacy-directory.com
NEXT_PUBLIC_ASTRO_BASE_URL=https://privacy-directory.com
DATABASE_URL=postgresql://prod-db/privacy_directory
```

## Testing Strategy

### Unit Tests
```typescript
// src/nextjs/__tests__/components/OptOutWizard.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { OptOutWizard } from '../components/OptOutWizard';

describe('OptOutWizard', () => {
  it('should render first step by default', () => {
    const mockBroker = createMockBroker();
    render(<OptOutWizard broker={mockBroker} onComplete={jest.fn()} />);
    
    expect(screen.getByText('Verify Identity')).toBeInTheDocument();
    expect(screen.getByText('Opt Out from Test Broker')).toBeInTheDocument();
  });

  it('should navigate to next step', async () => {
    const mockBroker = createMockBroker();
    const onComplete = jest.fn();
    
    render(<OptOutWizard broker={mockBroker} onComplete={onComplete} />);
    
    const nextButton = screen.getByText('Next');
    fireEvent.click(nextButton);
    
    expect(screen.getByText('Choose Method')).toBeInTheDocument();
  });
});
```

### Integration Tests
```typescript
// src/nextjs/__tests__/api/companies.test.ts
import { createMocks } from 'node-mocks-http';
import { GET } from '../app/api/companies/route';

describe('/api/companies', () => {
  it('should return companies list', async () => {
    const { req } = createMocks({ method: 'GET' });
    
    const response = await GET(req);
    const data = await response.json();
    
    expect(response.status).toBe(200);
    expect(data.success).toBe(true);
    expect(Array.isArray(data.data)).toBe(true);
  });
});
```

## Performance Optimization

### Caching Strategy
```typescript
// src/nextjs/app/api/companies/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { unstable_cache } from 'next/cache';

const getCompaniesCached = unstable_cache(
  async (page: number, limit: number, search: string) => {
    return getCompaniesPaginated(page, limit, search);
  },
  ['companies-list'],
  {
    revalidate: 3600, // 1 hour
    tags: ['companies']
  }
);

export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url);
  const page = parseInt(searchParams.get('page') || '1');
  const limit = parseInt(searchParams.get('limit') || '20');
  const search = searchParams.get('search') || '';

  const companies = await getCompaniesCached(page, limit, search);
  
  return NextResponse.json({
    data: companies,
    success: true
  });
}
```

### Bundle Optimization
```javascript
// next.config.js
const nextConfig = {
  webpack: (config, { isServer }) => {
    if (!isServer) {
      config.optimization.splitChunks = {
        chunks: 'all',
        cacheGroups: {
          vendor: {
            test: /[\\/]node_modules[\\/]/,
            name: 'vendors',
            chunks: 'all',
          },
        },
      };
    }
    return config;
  },
};
```

## Security Considerations

### API Security
```typescript
// src/nextjs/lib/api-security.ts
import { NextRequest } from 'next/server';

export function validateApiKey(request: NextRequest): boolean {
  const apiKey = request.headers.get('x-api-key');
  return apiKey === process.env.API_SECRET_KEY;
}

export function rateLimitMiddleware(ip: string): boolean {
  // Implement rate limiting logic
  return true;
}
```

### Data Protection
```typescript
// src/nextjs/lib/data-protection.ts
export function sanitizeUserData(data: any): any {
  // Remove sensitive information before logging
  const { password, ssn, ...sanitized } = data;
  return sanitized;
}

export function encryptSensitiveData(data: string): string {
  // Implement encryption for sensitive data
  return data;
}
```

## Usage Examples

### Opt-Out Wizard Integration
```typescript
// In Astro page
---
import { InteractiveOptOut } from '../components/interactive-opt-out';
---

<InteractiveOptOut brokerId="broker-123" brokerName="Acme Data Broker" />
```

### Data Broker Visualizer
```typescript
// In Next.js page
import { DataBrokerVisualizer } from '@/components/DataBrokerVisualizer';

export default function DataBrokersPage() {
  const [brokers, setBrokers] = useState<DataBrokerDto[]>([]);
  
  return (
    <div>
      <h1>Data Broker Landscape</h1>
      <DataBrokerVisualizer brokers={brokers} />
    </div>
  );
}
```

## Verification Checklist

### Architecture Compliance
- [ ] Next.js code isolated in `src/nextjs/`
- [ ] No direct imports from Next.js to Astro
- [ ] Shared DTOs properly defined
- [ ] API endpoints follow REST conventions

### Functionality
- [ ] Opt-out wizard works end-to-end
- [ ] Data visualizer displays correct metrics
- [ ] Search and filtering functions properly
- [ ] Error handling implemented

### Performance
- [ ] Caching strategy implemented
- [ ] Bundle optimization configured
- [ ] Loading states handled
- [ ] Responsive design implemented

### Security
- [ ] API key validation implemented
- [ ] Rate limiting configured
- [ ] Data sanitization in place
- [ ] HTTPS enforced in production

This skill ensures proper Next.js integration while maintaining architectural separation and following best practices for interactive privacy tools.
