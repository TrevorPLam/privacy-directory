---
trigger: glob
globs: **/api/**,**/src/nextjs/app/api/**,**src/web/pages/api/**
---

# API Integration Rules

## API Architecture Standards

### RESTful API Design
- **Resource-Based URLs**: Use nouns, not verbs (e.g., `/companies` not `/getCompanies`)
- **HTTP Methods**: Use appropriate methods (GET, POST, PUT, DELETE)
- **Status Codes**: Return proper HTTP status codes
- **Consistent Responses**: Standardized response format across all endpoints

### Response Format Standards
```typescript
// Standard API Response
interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: {
    code: string;
    message: string;
    details?: any;
  };
  meta?: {
    timestamp: string;
    requestId: string;
    version: string;
  };
}

// Paginated Response
interface PaginatedResponse<T> extends ApiResponse<T[]> {
  data: T[];
  pagination: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
    hasNext: boolean;
    hasPrev: boolean;
  };
}
```

## Endpoint Definitions

### Companies API
```typescript
// GET /api/companies
// Query parameters: page, limit, search, category, sensitivity
export async function GET(request: NextRequest) {
  try {
    const { searchParams } = new URL(request.url);
    const page = parseInt(searchParams.get('page') || '1');
    const limit = Math.min(parseInt(searchParams.get('limit') || '20'), 100);
    const search = searchParams.get('search') || '';
    const category = searchParams.get('category');
    const sensitivity = searchParams.get('sensitivity');

    // Validate parameters
    if (page < 1 || limit < 1) {
      return NextResponse.json({
        success: false,
        error: {
          code: 'INVALID_PARAMETERS',
          message: 'Page and limit must be positive integers'
        }
      }, { status: 400 });
    }

    // Fetch companies with filters
    const companies = await getCompanies({
      page,
      limit,
      search,
      category,
      sensitivity
    });

    const total = await getCompaniesCount({ search, category, sensitivity });

    return NextResponse.json({
      success: true,
      data: companies,
      pagination: {
        page,
        limit,
        total,
        totalPages: Math.ceil(total / limit),
        hasNext: page * limit < total,
        hasPrev: page > 1
      },
      meta: {
        timestamp: new Date().toISOString(),
        requestId: generateRequestId(),
        version: '1.0'
      }
    });
  } catch (error) {
    return NextResponse.json({
      success: false,
      error: {
        code: 'INTERNAL_ERROR',
        message: 'Failed to fetch companies'
      }
    }, { status: 500 });
  }
}

// GET /api/companies/[id]
export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    const company = await getCompanyById(params.id);
    
    if (!company) {
      return NextResponse.json({
        success: false,
        error: {
          code: 'NOT_FOUND',
          message: 'Company not found'
        }
      }, { status: 404 });
    }

    return NextResponse.json({
      success: true,
      data: company,
      meta: {
        timestamp: new Date().toISOString(),
        requestId: generateRequestId(),
        version: '1.0'
      }
    });
  } catch (error) {
    return NextResponse.json({
      success: false,
      error: {
        code: 'INTERNAL_ERROR',
        message: 'Failed to fetch company'
      }
    }, { status: 500 });
  }
}
```

### Data Brokers API
```typescript
// GET /api/data-brokers
export async function GET(request: NextRequest) {
  try {
    const { searchParams } = new URL(request.url);
    const hasOptOut = searchParams.get('hasOptOut') === 'true';
    const complianceStatus = searchParams.get('complianceStatus');

    const brokers = await getDataBrokers({
      hasOptOut,
      complianceStatus
    });

    return NextResponse.json({
      success: true,
      data: brokers,
      meta: {
        timestamp: new Date().toISOString(),
        requestId: generateRequestId(),
        version: '1.0'
      }
    });
  } catch (error) {
    return NextResponse.json({
      success: false,
      error: {
        code: 'INTERNAL_ERROR',
        message: 'Failed to fetch data brokers'
      }
    }, { status: 500 });
  }
}

// POST /api/data-brokers/[id]/opt-out
export async function POST(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    const body = await request.json();
    
    // Validate request body
    const validationResult = validateOptOutRequest(body);
    if (validationResult.isErr()) {
      return NextResponse.json({
        success: false,
        error: {
          code: 'VALIDATION_ERROR',
          message: 'Invalid request data',
          details: validationResult.error
        }
      }, { status: 400 });
    }

    const result = await processOptOutRequest(params.id, validationResult.value);
    
    return NextResponse.json({
      success: true,
      data: result,
      meta: {
        timestamp: new Date().toISOString(),
        requestId: generateRequestId(),
        version: '1.0'
      }
    });
  } catch (error) {
    return NextResponse.json({
      success: false,
      error: {
        code: 'INTERNAL_ERROR',
        message: 'Failed to process opt-out request'
      }
    }, { status: 500 });
  }
}
```

### Privacy Incidents API
```typescript
// GET /api/incidents
export async function GET(request: NextRequest) {
  try {
    const { searchParams } = new URL(request.url);
    const incidentType = searchParams.get('type');
    const severity = searchParams.get('severity');
    const dateFrom = searchParams.get('dateFrom');
    const dateTo = searchParams.get('dateTo');

    // Validate date parameters
    if (dateFrom && !isValidDate(dateFrom)) {
      return NextResponse.json({
        success: false,
        error: {
          code: 'INVALID_DATE',
          message: 'Invalid dateFrom format'
        }
      }, { status: 400 });
    }

    const incidents = await getIncidents({
      type: incidentType,
      severity,
      dateFrom,
      dateTo
    });

    return NextResponse.json({
      success: true,
      data: incidents,
      meta: {
        timestamp: new Date().toISOString(),
        requestId: generateRequestId(),
        version: '1.0'
      }
    });
  } catch (error) {
    return NextResponse.json({
      success: false,
      error: {
        code: 'INTERNAL_ERROR',
        message: 'Failed to fetch incidents'
      }
    }, { status: 500 });
  }
}
```

## Security Implementation

### API Key Authentication
```typescript
// src/nextjs/lib/auth.ts
export function validateApiKey(request: NextRequest): boolean {
  const apiKey = request.headers.get('x-api-key');
  const validKey = process.env.API_SECRET_KEY;
  
  return apiKey === validKey;
}

export function requireAuth(handler: (req: NextRequest, ...args: any[]) => Promise<NextResponse>) {
  return async (request: NextRequest, ...args: any[]) => {
    if (!validateApiKey(request)) {
      return NextResponse.json({
        success: false,
        error: {
          code: 'UNAUTHORIZED',
          message: 'Invalid or missing API key'
        }
      }, { status: 401 });
    }
    
    return handler(request, ...args);
  };
}

// Usage
export const GET = requireAuth(async (request: NextRequest) => {
  // Protected endpoint logic
});
```

### Rate Limiting
```typescript
// src/nextjs/lib/rate-limit.ts
const rateLimitMap = new Map<string, { count: number; resetTime: number }>();

export function rateLimit(
  identifier: string,
  limit: number = 100,
  windowMs: number = 60000 // 1 minute
): { allowed: boolean; remaining: number; resetTime: number } {
  const now = Date.now();
  const key = identifier;
  const record = rateLimitMap.get(key);

  if (!record || now > record.resetTime) {
    // New window or expired window
    rateLimitMap.set(key, {
      count: 1,
      resetTime: now + windowMs
    });
    return {
      allowed: true,
      remaining: limit - 1,
      resetTime: now + windowMs
    };
  }

  if (record.count >= limit) {
    return {
      allowed: false,
      remaining: 0,
      resetTime: record.resetTime
    };
  }

  record.count++;
  return {
    allowed: true,
    remaining: limit - record.count,
    resetTime: record.resetTime
  };
}

// Middleware
export function withRateLimit(handler: (req: NextRequest, ...args: any[]) => Promise<NextResponse>) {
  return async (request: NextRequest, ...args: any[]) => {
    const ip = request.ip || request.headers.get('x-forwarded-for') || 'unknown';
    const result = rateLimit(ip);

    if (!result.allowed) {
      return NextResponse.json({
        success: false,
        error: {
          code: 'RATE_LIMIT_EXCEEDED',
          message: 'Too many requests',
          details: {
            resetTime: result.resetTime
          }
        }
      }, { 
        status: 429,
        headers: {
          'X-RateLimit-Limit': '100',
          'X-RateLimit-Remaining': result.remaining.toString(),
          'X-RateLimit-Reset': result.resetTime.toString()
        }
      });
    }

    const response = await handler(request, ...args);
    
    // Add rate limit headers
    response.headers.set('X-RateLimit-Limit', '100');
    response.headers.set('X-RateLimit-Remaining', result.remaining.toString());
    response.headers.set('X-RateLimit-Reset', result.resetTime.toString());
    
    return response;
  };
}
```

### Input Validation
```typescript
// src/nextjs/lib/validation.ts
export function validateQueryParams(
  params: Record<string, string>,
  schema: Record<string, { type: string; required?: boolean; min?: number; max?: number }>
): { valid: boolean; errors: string[] } {
  const errors: string[] = [];

  for (const [key, rules] of Object.entries(schema)) {
    const value = params[key];

    if (rules.required && (!value || value.trim() === '')) {
      errors.push(`${key} is required`);
      continue;
    }

    if (value) {
      switch (rules.type) {
        case 'number':
          const num = parseInt(value);
          if (isNaN(num)) {
            errors.push(`${key} must be a number`);
          } else {
            if (rules.min !== undefined && num < rules.min) {
              errors.push(`${key} must be at least ${rules.min}`);
            }
            if (rules.max !== undefined && num > rules.max) {
              errors.push(`${key} must be at most ${rules.max}`);
            }
          }
          break;

        case 'email':
          const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
          if (!emailRegex.test(value)) {
            errors.push(`${key} must be a valid email`);
          }
          break;

        case 'url':
          try {
            new URL(value);
          } catch {
            errors.push(`${key} must be a valid URL`);
          }
          break;

        case 'enum':
          if (rules.enum && !rules.enum.includes(value)) {
            errors.push(`${key} must be one of: ${rules.enum.join(', ')}`);
          }
          break;
      }
    }
  }

  return {
    valid: errors.length === 0,
    errors
  };
}
```

## Error Handling

### Standardized Error Responses
```typescript
// src/nextjs/lib/errors.ts
export class ApiError extends Error {
  constructor(
    public code: string,
    message: string,
    public statusCode: number = 500,
    public details?: any
  ) {
    super(message);
  }
}

export const ERROR_CODES = {
  // Validation errors
  VALIDATION_ERROR: 'VALIDATION_ERROR',
  INVALID_PARAMETERS: 'INVALID_PARAMETERS',
  MISSING_REQUIRED_FIELD: 'MISSING_REQUIRED_FIELD',
  
  // Authentication/Authorization
  UNAUTHORIZED: 'UNAUTHORIZED',
  FORBIDDEN: 'FORBIDDEN',
  
  // Not found
  NOT_FOUND: 'NOT_FOUND',
  RESOURCE_NOT_FOUND: 'RESOURCE_NOT_FOUND',
  
  // Rate limiting
  RATE_LIMIT_EXCEEDED: 'RATE_LIMIT_EXCEEDED',
  
  // Server errors
  INTERNAL_ERROR: 'INTERNAL_ERROR',
  DATABASE_ERROR: 'DATABASE_ERROR',
  EXTERNAL_SERVICE_ERROR: 'EXTERNAL_SERVICE_ERROR'
};

export function handleApiError(error: unknown): NextResponse {
  if (error instanceof ApiError) {
    return NextResponse.json({
      success: false,
      error: {
        code: error.code,
        message: error.message,
        details: error.details
      },
      meta: {
        timestamp: new Date().toISOString(),
        requestId: generateRequestId(),
        version: '1.0'
      }
    }, { status: error.statusCode });
  }

  // Unexpected errors
  console.error('Unexpected API error:', error);
  return NextResponse.json({
    success: false,
    error: {
      code: ERROR_CODES.INTERNAL_ERROR,
      message: 'An unexpected error occurred'
    },
    meta: {
      timestamp: new Date().toISOString(),
      requestId: generateRequestId(),
      version: '1.0'
    }
  }, { status: 500 });
}
```

### Error Boundary for API Routes
```typescript
// src/nextjs/lib/error-handler.ts
export function withErrorHandler(
  handler: (req: NextRequest, ...args: any[]) => Promise<NextResponse>
) {
  return async (request: NextRequest, ...args: any[]) => {
    try {
      return await handler(request, ...args);
    } catch (error) {
      return handleApiError(error);
    }
  };
}

// Usage
export const GET = withErrorHandler(async (request: NextRequest) => {
  // API logic here
});
```

## Caching Strategy

### Response Caching
```typescript
// src/nextjs/lib/cache.ts
import { unstable_cache } from 'next/cache';

export const getCachedCompanies = unstable_cache(
  async (params: CompanyQueryParams) => {
    return getCompanies(params);
  },
  ['companies'],
  {
    revalidate: 3600, // 1 hour
    tags: ['companies']
  }
);

export const getCachedDataBrokers = unstable_cache(
  async (params: DataBrokerQueryParams) => {
    return getDataBrokers(params);
  },
  ['data-brokers'],
  {
    revalidate: 1800, // 30 minutes
    tags: ['data-brokers']
  }
);

// Usage in API routes
export async function GET(request: NextRequest) {
  const params = parseQueryParams(request.url);
  const companies = await getCachedCompanies(params);
  
  return NextResponse.json({
    success: true,
    data: companies
  });
}
```

### Cache Invalidation
```typescript
// src/nextjs/lib/cache-invalidation.ts
import { revalidateTag } from 'next/cache';

export function invalidateCompaniesCache() {
  revalidateTag('companies');
}

export function invalidateDataBrokersCache() {
  revalidateTag('data-brokers');
}

// Usage after data updates
export async function POST(request: NextRequest) {
  // Create/update company
  const company = await createCompany(data);
  
  // Invalidate cache
  invalidateCompaniesCache();
  
  return NextResponse.json({
    success: true,
    data: company
  });
}
```

## API Documentation

### OpenAPI Specification
```typescript
// src/nextjs/lib/openapi.ts
export const openApiSpec = {
  openapi: '3.0.0',
  info: {
    title: 'Privacy Directory API',
    version: '1.0.0',
    description: 'API for accessing privacy directory data'
  },
  paths: {
    '/api/companies': {
      get: {
        summary: 'Get companies list',
        parameters: [
          {
            name: 'page',
            in: 'query',
            schema: { type: 'integer', minimum: 1, default: 1 }
          },
          {
            name: 'limit',
            in: 'query',
            schema: { type: 'integer', minimum: 1, maximum: 100, default: 20 }
          },
          {
            name: 'search',
            in: 'query',
            schema: { type: 'string' }
          }
        ],
        responses: {
          200: {
            description: 'Successful response',
            content: {
              'application/json': {
                schema: { $ref: '#/components/schemas/CompaniesResponse' }
              }
            }
          }
        }
      }
    }
  },
  components: {
    schemas: {
      CompaniesResponse: {
        type: 'object',
        properties: {
          success: { type: 'boolean' },
          data: {
            type: 'array',
            items: { $ref: '#/components/schemas/Company' }
          },
          pagination: { $ref: '#/components/schemas/Pagination' }
        }
      }
    }
  }
};
```

## Testing API Endpoints

### Unit Tests
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
    expect(data.pagination).toBeDefined();
  });

  it('should handle pagination parameters', async () => {
    const { req } = createMocks({
      method: 'GET',
      query: { page: '2', limit: '10' }
    });
    
    const response = await GET(req);
    const data = await response.json();
    
    expect(data.pagination.page).toBe(2);
    expect(data.pagination.limit).toBe(10);
  });

  it('should validate parameters', async () => {
    const { req } = createMocks({
      method: 'GET',
      query: { page: '-1', limit: '0' }
    });
    
    const response = await GET(req);
    const data = await response.json();
    
    expect(response.status).toBe(400);
    expect(data.success).toBe(false);
    expect(data.error.code).toBe('INVALID_PARAMETERS');
  });
});
```

### Integration Tests
```typescript
// src/nextjs/__tests__/integration/api.test.ts
describe('API Integration', () => {
  it('should handle complete company workflow', async () => {
    // Create company
    const createResponse = await fetch('/api/companies', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        name: 'Test Company',
        url: 'https://example.com',
        practices: [{
          category: 'DATA_COLLECTION',
          details: 'Test practice'
        }],
        dataSensitivityTier: 'MODERATE'
      })
    });
    
    expect(createResponse.status).toBe(201);
    const createdCompany = await createResponse.json();
    
    // Get company
    const getResponse = await fetch(`/api/companies/${createdCompany.data.id}`);
    expect(getResponse.status).toBe(200);
    
    const retrievedCompany = await getResponse.json();
    expect(retrievedCompany.data.name).toBe('Test Company');
  });
});
```

## Performance Monitoring

### Request Logging
```typescript
// src/nextjs/lib/logging.ts
export function withLogging(
  handler: (req: NextRequest, ...args: any[]) => Promise<NextResponse>
) {
  return async (request: NextRequest, ...args: any[]) => {
    const startTime = Date.now();
    const requestId = generateRequestId();
    
    // Log request
    console.log(JSON.stringify({
      type: 'request',
      requestId,
      method: request.method,
      url: request.url,
      userAgent: request.headers.get('user-agent'),
      ip: request.ip,
      timestamp: new Date().toISOString()
    }));
    
    try {
      const response = await handler(request, ...args);
      const duration = Date.now() - startTime;
      
      // Log response
      console.log(JSON.stringify({
        type: 'response',
        requestId,
        status: response.status,
        duration,
        timestamp: new Date().toISOString()
      }));
      
      return response;
    } catch (error) {
      const duration = Date.now() - startTime;
      
      // Log error
      console.error(JSON.stringify({
        type: 'error',
        requestId,
        error: error.message,
        duration,
        timestamp: new Date().toISOString()
      }));
      
      throw error;
    }
  };
}
```

## Verification Checklist

### API Standards
- [ ] RESTful design principles followed
- [ ] Consistent response format
- [ ] Proper HTTP status codes
- [ ] Comprehensive error handling

### Security
- [ ] API key authentication implemented
- [ ] Rate limiting configured
- [ ] Input validation in place
- [ ] SQL injection prevention

### Performance
- [ ] Caching strategy implemented
- [ ] Database query optimization
- [ ] Response size limits
- [ ] Request logging for monitoring

### Documentation
- [ ] OpenAPI specification complete
- [ ] Endpoint documentation
- [ ] Error code documentation
- [ ] Usage examples provided

### Testing
- [ ] Unit tests for all endpoints
- [ ] Integration tests for workflows
- [ ] Error scenario testing
- [ ] Performance testing

This ensures robust, secure, and well-documented API integration for the privacy directory.
