---
name: security-privacy
description: Comprehensive security and privacy implementation for privacy directory with data protection and compliance
---

# Security and Privacy Skill

## Overview
This skill provides comprehensive security and privacy implementation patterns for the Privacy Directory project, ensuring data protection, user privacy, compliance with regulations, and secure development practices.

## When to Use
- Implementing security controls for user data
- Setting up privacy-preserving analytics
- Configuring secure authentication and authorization
- Implementing data encryption and protection
- Setting up compliance monitoring
- Building secure API endpoints
- Implementing audit logging and monitoring

## Core Security Principles

### Defense in Depth
```typescript
// src/shared/security/security-layers.ts
export enum SecurityLayer {
  NETWORK = 'network',
  APPLICATION = 'application',
  DATA = 'data',
  INFRASTRUCTURE = 'infrastructure'
}

export interface SecurityControl {
  layer: SecurityLayer;
  control: string;
  implementation: string;
  status: 'implemented' | 'planned' | 'missing';
}

export class SecurityFramework {
  private controls: SecurityControl[] = [
    {
      layer: SecurityLayer.NETWORK,
      control: 'HTTPS Enforcement',
      implementation: 'TLS 1.3 for all connections',
      status: 'implemented'
    },
    {
      layer: SecurityLayer.APPLICATION,
      control: 'Input Validation',
      implementation: 'Comprehensive validation on all inputs',
      status: 'implemented'
    },
    {
      layer: SecurityLayer.DATA,
      control: 'Encryption at Rest',
      implementation: 'AES-256 encryption for sensitive data',
      status: 'implemented'
    },
    {
      layer: SecurityLayer.INFRASTRUCTURE,
      control: 'Access Controls',
      implementation: 'Role-based access control',
      status: 'planned'
    }
  ];

  getControlsByLayer(layer: SecurityLayer): SecurityControl[] {
    return this.controls.filter(control => control.layer === layer);
  }

  getSecurityStatus(): { implemented: number; planned: number; missing: number } {
    const implemented = this.controls.filter(c => c.status === 'implemented').length;
    const planned = this.controls.filter(c => c.status === 'planned').length;
    const missing = this.controls.filter(c => c.status === 'missing').length;

    return { implemented, planned, missing };
  }
}
```

### Privacy by Design
```typescript
// src/shared/privacy/privacy-by-design.ts
export enum PrivacyPrinciple {
  LAWFULNESS = 'lawfulness',
  FAIRNESS_TRANSPARENCY = 'fairness_transparency',
  PURPOSE_LIMITATION = 'purpose_limitation',
  DATA_MINIMIZATION = 'data_minimization',
  ACCURACY = 'accuracy',
  STORAGE_LIMITATION = 'storage_limitation',
  INTEGRITY_CONFIDENTIALITY = 'integrity_confidentiality',
  ACCOUNTABILITY = 'accountability'
}

export class PrivacyImpactAssessment {
  private principles: Map<PrivacyPrinciple, boolean> = new Map();

  assessPrinciple(principle: PrivacyPrinciple, compliant: boolean, notes?: string): void {
    this.principles.set(principle, compliant);
    
    if (!compliant && notes) {
      console.warn(`Privacy principle ${principle} not compliant: ${notes}`);
    }
  }

  getComplianceScore(): number {
    const total = this.principles.size;
    const compliant = Array.from(this.principles.values()).filter(Boolean).length;
    return total > 0 ? (compliant / total) * 100 : 0;
  }

  generateReport(): {
    score: number;
    compliant: PrivacyPrinciple[];
    nonCompliant: PrivacyPrinciple[];
  } {
    const compliant: PrivacyPrinciple[] = [];
    const nonCompliant: PrivacyPrinciple[] = [];

    for (const [principle, isCompliant] of this.principles) {
      if (isCompliant) {
        compliant.push(principle);
      } else {
        nonCompliant.push(principle);
      }
    }

    return {
      score: this.getComplianceScore(),
      compliant,
      nonCompliant
    };
  }
}
```

## Data Protection Implementation

### Encryption Utilities
```typescript
// src/shared/security/encryption.ts
import crypto from 'crypto';

export class EncryptionService {
  private readonly algorithm = 'aes-256-gcm';
  private readonly keyLength = 32; // 256 bits

  constructor(private readonly encryptionKey: Buffer) {
    if (encryptionKey.length !== this.keyLength) {
      throw new Error('Encryption key must be 32 bytes');
    }
  }

  encrypt(plaintext: string): { encrypted: string; iv: string; tag: string } {
    const iv = crypto.randomBytes(16); // 128 bits IV
    const cipher = crypto.createCipher(this.algorithm, this.encryptionKey);
    cipher.setAAD(Buffer.from('privacy-directory', 'utf8'));

    let encrypted = cipher.update(plaintext, 'utf8', 'hex');
    encrypted += cipher.final('hex');

    const tag = cipher.getAuthTag();

    return {
      encrypted,
      iv: iv.toString('hex'),
      tag: tag.toString('hex')
    };
  }

  decrypt(encryptedData: { encrypted: string; iv: string; tag: string }): string {
    const decipher = crypto.createDecipher(this.algorithm, this.encryptionKey);
    decipher.setAAD(Buffer.from('privacy-directory', 'utf8'));
    decipher.setAuthTag(Buffer.from(tag, 'hex'));

    let decrypted = decipher.update(encryptedData.encrypted, 'hex', 'utf8');
    decrypted += decipher.final('utf8');

    return decrypted;
  }

  hashSensitiveData(data: string): string {
    return crypto.createHash('sha256').update(data).digest('hex');
  }
}
```

### Data Masking
```typescript
// src/shared/privacy/data-masking.ts
export class DataMaskingService {
  maskEmail(email: string): string {
    const [username, domain] = email.split('@');
    const maskedUsername = username.slice(0, 2) + '*'.repeat(username.length - 2);
    return `${maskedUsername}@${domain}`;
  }

  maskPhoneNumber(phone: string): string {
    return phone.replace(/(\d{2})\d{6}(\d{2})/, '$1******$2');
  }

  maskIpAddress(ip: string): string {
    const parts = ip.split('.');
    return `${parts[0]}.${parts[1]}.***.${parts[3]}`;
  }

  maskSensitiveField(value: string, visibleChars: number = 4): string {
    if (value.length <= visibleChars) {
      return '*'.repeat(value.length);
    }
    return value.slice(0, visibleChars) + '*'.repeat(value.length - visibleChars);
  }

  anonymizeUserData(userData: {
    email?: string;
    phone?: string;
    ipAddress?: string;
    name?: string;
  }): typeof userData {
    const anonymized = { ...userData };

    if (anonymized.email) {
      anonymized.email = this.maskEmail(anonymized.email);
    }

    if (anonymized.phone) {
      anonymized.phone = this.maskPhoneNumber(anonymized.phone);
    }

    if (anonymized.ipAddress) {
      anonymized.ipAddress = this.maskIpAddress(anonymized.ipAddress);
    }

    if (anonymized.name) {
      anonymized.name = this.maskSensitiveField(anonymized.name);
    }

    return anonymized;
  }
}
```

## Privacy-Preserving Analytics

### Plausible Analytics Integration
```typescript
// src/web/lib/analytics.ts
export class PrivacyAnalytics {
  private plausibleDomain: string;
  private plausibleUrl: string;

  constructor() {
    this.plausibleDomain = process.env.PLAUSIBLE_DOMAIN || 'privacy-directory.com';
    this.plausibleUrl = process.env.PLAUSIBLE_URL || 'https://plausible.io';
  }

  trackPageView(page: string, referrer?: string): void {
    // Only track if user has not opted out
    if (this.hasAnalyticsConsent()) {
      const url = `${this.plausibleUrl}/api/event`;
      const data = {
        name: 'pageview',
        url: `${this.plausibleDomain}${page}`,
        domain: this.plausibleDomain,
        referrer: referrer || document.referrer
      };

      // Send using navigator.sendBeacon for better performance
      navigator.sendBeacon(url, JSON.stringify(data));
    }
  }

  trackCustomEvent(name: string, props?: Record<string, string>): void {
    if (this.hasAnalyticsConsent()) {
      const url = `${this.plausibleUrl}/api/event`;
      const data = {
        name,
        url: window.location.href,
        domain: this.plausibleDomain,
        props
      };

      navigator.sendBeacon(url, JSON.stringify(data));
    }
  }

  private hasAnalyticsConsent(): boolean {
    // Check localStorage for consent preference
    const consent = localStorage.getItem('analytics-consent');
    return consent === 'granted';
  }

  setAnalyticsConsent(granted: boolean): void {
    localStorage.setItem('analytics-consent', granted ? 'granted' : 'denied');
  }
}
```

### Privacy-Friendly Error Tracking
```typescript
// src/shared/privacy/error-tracking.ts
interface ErrorReport {
  message: string;
  stack?: string;
  userAgent?: string;
  url?: string;
  timestamp: string;
  userId?: string;
  sessionId: string;
}

export class PrivacyErrorTracker {
  private sessionId: string;
  private dataMasking: DataMaskingService;

  constructor() {
    this.sessionId = this.generateSessionId();
    this.dataMasking = new DataMaskingService();
  }

  trackError(error: Error, context?: Record<string, any>): void {
    const report: ErrorReport = {
      message: error.message,
      stack: error.stack,
      userAgent: navigator.userAgent,
      url: window.location.href,
      timestamp: new Date().toISOString(),
      sessionId: this.sessionId
    };

    // Remove any PII from context
    if (context) {
      const sanitizedContext = this.sanitizeContext(context);
      // Include only non-sensitive context data
    }

    // Send to error tracking service without PII
    this.sendErrorReport(report);
  }

  private sanitizeContext(context: Record<string, any>): Record<string, any> {
    const sanitized: Record<string, any> = {};

    for (const [key, value] of Object.entries(context)) {
      if (typeof value === 'string') {
        // Check if the value might contain PII
        if (this.mightContainPII(key, value)) {
          sanitized[key] = this.dataMasking.maskSensitiveField(value);
        } else {
          sanitized[key] = value;
        }
      } else {
        sanitized[key] = value;
      }
    }

    return sanitized;
  }

  private mightContainPII(key: string, value: string): boolean {
    const piiKeywords = ['email', 'name', 'phone', 'address', 'ip', 'user'];
    const keyLower = key.toLowerCase();
    const valueLower = value.toLowerCase();

    return piiKeywords.some(keyword => 
      keyLower.includes(keyword) || 
      valueLower.includes(keyword)
    );
  }

  private generateSessionId(): string {
    return crypto.randomUUID();
  }

  private sendErrorReport(report: ErrorReport): void {
    // Implementation depends on error tracking service
    console.error('Privacy-safe error report:', report);
  }
}
```

## Secure Authentication

### JWT Token Management
```typescript
// src/nextjs/lib/auth/jwt.ts
import jwt from 'jsonwebtoken';
import { EncryptionService } from '../../../shared/security/encryption';

export interface JWTPayload {
  sub: string; // subject (user ID)
  iat: number; // issued at
  exp: number; // expiration
  scope: string[]; // permissions
  sessionId: string;
}

export class JWTService {
  constructor(
    private readonly secretKey: string,
    private readonly encryptionService: EncryptionService
  ) {}

  generateToken(payload: Omit<JWTPayload, 'iat' | 'exp'>): string {
    const now = Math.floor(Date.now() / 1000);
    const jwtPayload: JWTPayload = {
      ...payload,
      iat: now,
      exp: now + (60 * 60) // 1 hour expiration
    };

    return jwt.sign(jwtPayload, this.secretKey, {
      algorithm: 'HS256',
      issuer: 'privacy-directory',
      audience: 'privacy-directory-users'
    });
  }

  verifyToken(token: string): Result<JWTPayload, string> {
    try {
      const decoded = jwt.verify(token, this.secretKey, {
        algorithms: ['HS256'],
        issuer: 'privacy-directory',
        audience: 'privacy-directory-users'
      }) as JWTPayload;

      return Ok(decoded);
    } catch (error) {
      return Err('Invalid token');
    }
  }

  refreshToken(token: string): Result<string, string> {
    const verification = this.verifyToken(token);
    if (verification.isErr()) {
      return verification;
    }

    const payload = verification.value;
    const newPayload = {
      sub: payload.sub,
      scope: payload.scope,
      sessionId: payload.sessionId
    };

    return Ok(this.generateToken(newPayload));
  }
}
```

### Session Management
```typescript
// src/nextjs/lib/auth/session.ts
export interface SessionData {
  userId: string;
  sessionId: string;
  createdAt: Date;
  lastActivity: Date;
  ipAddress: string;
  userAgent: string;
}

export class SessionManager {
  private sessions = new Map<string, SessionData>();
  private readonly sessionTimeout = 30 * 60 * 1000; // 30 minutes

  createSession(userId: string, request: NextRequest): SessionData {
    const sessionId = crypto.randomUUID();
    const now = new Date();

    const sessionData: SessionData = {
      userId,
      sessionId,
      createdAt: now,
      lastActivity: now,
      ipAddress: request.ip || 'unknown',
      userAgent: request.headers.get('user-agent') || 'unknown'
    };

    this.sessions.set(sessionId, sessionData);
    return sessionData;
  }

  validateSession(sessionId: string): boolean {
    const session = this.sessions.get(sessionId);
    if (!session) {
      return false;
    }

    const now = Date.now();
    const lastActivity = session.lastActivity.getTime();

    if (now - lastActivity > this.sessionTimeout) {
      this.sessions.delete(sessionId);
      return false;
    }

    // Update last activity
    session.lastActivity = new Date();
    return true;
  }

  revokeSession(sessionId: string): void {
    this.sessions.delete(sessionId);
  }

  revokeAllUserSessions(userId: string): void {
    for (const [sessionId, session] of this.sessions) {
      if (session.userId === userId) {
        this.sessions.delete(sessionId);
      }
    }
  }

  cleanupExpiredSessions(): void {
    const now = Date.now();
    for (const [sessionId, session] of this.sessions) {
      const lastActivity = session.lastActivity.getTime();
      if (now - lastActivity > this.sessionTimeout) {
        this.sessions.delete(sessionId);
      }
    }
  }
}
```

## Security Headers and CSP

### Content Security Policy
```typescript
// src/nextjs/lib/security/csp.ts
export class CSPBuilder {
  private directives: Record<string, string[]> = {
    'default-src': ["'self'"],
    'script-src': ["'self'", "'unsafe-inline'", 'plausible.io'],
    'style-src': ["'self'", "'unsafe-inline'"],
    'img-src': ["'self'", 'data:', 'https:'],
    'connect-src': ["'self'", 'plausible.io'],
    'font-src': ["'self'"],
    'object-src': ["'none'"],
    'media-src': ["'self'"],
    'frame-src': ["'none'"],
    'child-src': ["'none'"],
    'worker-src': ["'self'"],
    'manifest-src': ["'self'"],
    'upgrade-insecure-requests': []
  };

  addDirective(directive: string, sources: string[]): this {
    this.directives[directive] = [...(this.directives[directive] || []), ...sources];
    return this;
  }

  removeDirective(directive: string): this {
    delete this.directives[directive];
    return this;
  }

  build(): string {
    return Object.entries(this.directives)
      .map(([directive, sources]) => {
        const sourceString = sources.join(' ');
        return `${directive} ${sourceString}`;
      })
      .join('; ');
  }
}

export const defaultCSP = new CSPBuilder()
  .addDirective('default-src', ["'self'"])
  .addDirective('script-src', ["'self'", 'plausible.io'])
  .addDirective('style-src', ["'self'", "'unsafe-inline'"])
  .addDirective('img-src', ["'self'", 'data:', 'https:'])
  .addDirective('connect-src', ["'self'", 'plausible.io'])
  .build();
```

### Security Headers Middleware
```typescript
// src/nextjs/lib/security/headers.ts
export function addSecurityHeaders(response: NextResponse): NextResponse {
  // Content Security Policy
  response.headers.set('Content-Security-Policy', defaultCSP);

  // Strict Transport Security
  response.headers.set(
    'Strict-Transport-Security',
    'max-age=31536000; includeSubDomains; preload'
  );

  // X-Frame-Options
  response.headers.set('X-Frame-Options', 'DENY');

  // X-Content-Type-Options
  response.headers.set('X-Content-Type-Options', 'nosniff');

  // Referrer Policy
  response.headers.set('Referrer-Policy', 'strict-origin-when-cross-origin');

  // Permissions Policy
  response.headers.set(
    'Permissions-Policy',
    'camera=(), microphone=(), geolocation=(), payment=()'
  );

  // X-XSS-Protection
  response.headers.set('X-XSS-Protection', '1; mode=block');

  return response;
}
```

## Audit Logging

### Security Event Logger
```typescript
// src/shared/security/audit-logger.ts
export enum SecurityEventType {
  LOGIN_SUCCESS = 'login_success',
  LOGIN_FAILURE = 'login_failure',
  LOGOUT = 'logout',
  PASSWORD_CHANGE = 'password_change',
  DATA_ACCESS = 'data_access',
  DATA_MODIFICATION = 'data_modification',
  PRIVILEGE_ESCALATION = 'privilege_escalation',
  SECURITY_VIOLATION = 'security_violation'
}

export interface SecurityEvent {
  type: SecurityEventType;
  userId?: string;
  sessionId?: string;
  ipAddress: string;
  userAgent?: string;
  resource?: string;
  action?: string;
  timestamp: Date;
  severity: 'low' | 'medium' | 'high' | 'critical';
  details?: Record<string, any>;
}

export class SecurityAuditLogger {
  private events: SecurityEvent[] = [];

  logEvent(event: Omit<SecurityEvent, 'timestamp'>): void {
    const securityEvent: SecurityEvent = {
      ...event,
      timestamp: new Date()
    };

    this.events.push(securityEvent);

    // Log to external service
    this.sendToAuditService(securityEvent);

    // Check for security violations
    if (event.severity === 'critical' || event.severity === 'high') {
      this.triggerSecurityAlert(securityEvent);
    }
  }

  logLoginAttempt(userId: string, success: boolean, ipAddress: string, userAgent?: string): void {
    this.logEvent({
      type: success ? SecurityEventType.LOGIN_SUCCESS : SecurityEventType.LOGIN_FAILURE,
      userId,
      ipAddress,
      userAgent,
      severity: success ? 'low' : 'medium',
      details: { loginSuccess: success }
    });
  }

  logDataAccess(userId: string, resource: string, action: string, ipAddress: string): void {
    this.logEvent({
      type: SecurityEventType.DATA_ACCESS,
      userId,
      resource,
      action,
      ipAddress,
      severity: 'low'
    });
  }

  logSecurityViolation(userId: string, violation: string, ipAddress: string): void {
    this.logEvent({
      type: SecurityEventType.SECURITY_VIOLATION,
      userId,
      ipAddress,
      severity: 'high',
      details: { violation }
    });
  }

  private sendToAuditService(event: SecurityEvent): void {
    // Implementation depends on audit service
    console.log('Security audit event:', event);
  }

  private triggerSecurityAlert(event: SecurityEvent): void {
    // Implementation for security alerts
    console.warn('Security alert triggered:', event);
  }

  getEventsByUser(userId: string, limit: number = 100): SecurityEvent[] {
    return this.events
      .filter(event => event.userId === userId)
      .sort((a, b) => b.timestamp.getTime() - a.timestamp.getTime())
      .slice(0, limit);
  }

  getEventsByType(type: SecurityEventType, limit: number = 100): SecurityEvent[] {
    return this.events
      .filter(event => event.type === type)
      .sort((a, b) => b.timestamp.getTime() - a.timestamp.getTime())
      .slice(0, limit);
  }
}
```

## Compliance Monitoring

### GDPR Compliance Tracker
```typescript
// src/shared/compliance/gdpr-tracker.ts
export enum GDPRRight {
  ACCESS = 'right_to_access',
  RECTIFICATION = 'right_to_rectification',
  ERASURE = 'right_to_erasure',
  PORTABILITY = 'right_to_portability',
  OBJECTION = 'right_to_objection',
  RESTRICTION = 'right_to_restriction'
}

export interface GDPRRequest {
  id: string;
  userId: string;
  right: GDPRRight;
  requestDate: Date;
  status: 'pending' | 'processing' | 'completed' | 'rejected';
  completedDate?: Date;
  details?: string;
}

export class GDPRComplianceTracker {
  private requests: Map<string, GDPRRequest> = new Map();

  createRequest(userId: string, right: GDPRRight, details?: string): GDPRRequest {
    const request: GDPRRequest = {
      id: crypto.randomUUID(),
      userId,
      right,
      requestDate: new Date(),
      status: 'pending',
      details
    };

    this.requests.set(request.id, request);
    return request;
  }

  processRequest(requestId: string): boolean {
    const request = this.requests.get(requestId);
    if (!request) {
      return false;
    }

    request.status = 'processing';
    return true;
  }

  completeRequest(requestId: string, outcome: 'completed' | 'rejected', details?: string): boolean {
    const request = this.requests.get(requestId);
    if (!request) {
      return false;
    }

    request.status = outcome;
    request.completedDate = new Date();
    if (details) {
      request.details = details;
    }

    return true;
  }

  getRequestStatus(requestId: string): GDPRRequest | null {
    return this.requests.get(requestId) || null;
  }

  getUserRequests(userId: string): GDPRRequest[] {
    return Array.from(this.requests.values())
      .filter(request => request.userId === userId)
      .sort((a, b) => b.requestDate.getTime() - a.requestDate.getTime());
  }

  getComplianceReport(): {
    totalRequests: number;
    pending: number;
    processing: number;
    completed: number;
    rejected: number;
    averageProcessingTime: number;
  } {
    const requests = Array.from(this.requests.values());
    const total = requests.length;
    const pending = requests.filter(r => r.status === 'pending').length;
    const processing = requests.filter(r => r.status === 'processing').length;
    const completed = requests.filter(r => r.status === 'completed').length;
    const rejected = requests.filter(r => r.status === 'rejected').length;

    const completedRequests = requests.filter(r => r.status === 'completed' && r.completedDate);
    const averageProcessingTime = completedRequests.length > 0
      ? completedRequests.reduce((sum, r) => 
          sum + (r.completedDate!.getTime() - r.requestDate.getTime()), 0) / completedRequests.length
      : 0;

    return {
      totalRequests: total,
      pending,
      processing,
      completed,
      rejected,
      averageProcessingTime
    };
  }
}
```

## Usage Examples

### Setting Up Security Framework
```typescript
// src/nextjs/lib/security-setup.ts
export function setupSecurityFramework(): {
  encryption: EncryptionService;
  auditLogger: SecurityAuditLogger;
  gdprTracker: GDPRComplianceTracker;
  sessionManager: SessionManager;
} {
  const encryptionKey = process.env.ENCRYPTION_KEY
    ? Buffer.from(process.env.ENCRYPTION_KEY, 'hex')
    : crypto.randomBytes(32);

  const encryption = new EncryptionService(encryptionKey);
  const auditLogger = new SecurityAuditLogger();
  const gdprTracker = new GDPRComplianceTracker();
  const sessionManager = new SessionManager();

  return {
    encryption,
    auditLogger,
    gdprTracker,
    sessionManager
  };
}
```

### Middleware Integration
```typescript
// src/nextjs/middleware.ts
import { NextRequest, NextResponse } from 'next/server';
import { addSecurityHeaders } from './lib/security/headers';
import { SecurityAuditLogger } from './shared/security/audit-logger';

const auditLogger = new SecurityAuditLogger();

export function middleware(request: NextRequest) {
  const response = NextResponse.next();

  // Add security headers
  addSecurityHeaders(response);

  // Log security events
  const ipAddress = request.ip || 'unknown';
  const userAgent = request.headers.get('user-agent') || 'unknown';

  // Log suspicious requests
  if (request.url.includes('/admin') || request.url.includes('/api/admin')) {
    auditLogger.logDataAccess('anonymous', request.url, 'ACCESS_ATTEMPT', ipAddress);
  }

  return response;
}
```

## Verification Checklist

### Security Implementation
- [ ] Encryption service implemented for sensitive data
- [ ] Secure session management in place
- [ ] JWT token handling secure
- [ ] Security headers configured
- [ ] Content Security Policy implemented

### Privacy Compliance
- [ ] Privacy by design principles implemented
- [ ] Data masking for PII
- [ ] Privacy-friendly analytics configured
- [ ] GDPR compliance tracking
- [ ] User consent management

### Monitoring and Auditing
- [ ] Security event logging
- [ ] Audit trail for data access
- [ ] Compliance monitoring
- [ ] Security alerting
- [ ] Performance monitoring

### Testing
- [ ] Security unit tests
- [ ] Penetration testing
- [ ] Privacy impact assessment
- [ ] Compliance validation
- [ ] Error handling testing

This skill ensures comprehensive security and privacy implementation for the Privacy Directory project with proper data protection, compliance monitoring, and secure development practices.
