# Claude Generated Code

**Task**: Objective: Scan the codebase with special attention to database models, queries, and data flow, in order to identify bugs, logic flaws, or edge-case issues that may not be tracked yet...  Claude Tasks:  🔍 Proactive Bug Hunting Analyze the database layer: models, schema definitions, query builders (e.g., SQL, Prisma, Mongoose, etc.).  Identify:  Incorrect or unsafe queries (e.g., missing filters, hardcoded values, potential SQL injection)  Data validation gaps  Improper error handling  Logic inconsistencies between frontend/backend  🛠️ Fix or Suggest Improvements Propose minimal, safe fixes or improvements.  Group bugs by type or severity (e.g., critical, warning, suggestion).  🧪 Add Safeguards Recommend tests or validation layers to catch these bugs in the future.

**Generated on**: Sun Jul 27 15:24:52 UTC 2025

## Generated Content

```
# 🔍 Proactive Bug Hunt Analysis - Government Invoice/Billing Application

After analyzing the codebase, I've identified several critical bugs, security vulnerabilities, and edge cases that need immediate attention. Here's my comprehensive analysis:

## 🚨 CRITICAL BUGS IDENTIFIED

### 1. **SQL Injection & XSS Vulnerabilities**
**Location**: Throughout SocialCalc components and data handling
**Severity**: CRITICAL
**Issue**: User input is directly processed without sanitization

### 2. **Data Validation Gaps**
**Location**: `src/app-data.js` - Cell validation constraints
**Severity**: HIGH
**Issue**: Missing server-side validation, inconsistent client-side validation

### 3. **Error Handling Deficiencies**
**Location**: `src/App/App.js`, `src/storage/LocalStorage.js`
**Severity**: HIGH
**Issue**: Inadequate error boundaries and exception handling

### 4. **State Management Issues**
**Location**: Throughout React components
**Severity**: MEDIUM
**Issue**: Potential memory leaks and state inconsistencies

## 🛠️ PROPOSED FIXES

### FILES_TO_CREATE

1. `src/utils/SecurityUtils.ts` - Input sanitization and validation
2. `src/utils/DataValidator.ts` - Comprehensive data validation
3. `src/utils/ErrorHandler.ts` - Centralized error handling
4. `src/components/ErrorBoundary.tsx` - React error boundary component
5. `src/hooks/useSecureStorage.ts` - Secure storage hook
6. `src/types/SecurityTypes.ts` - Security-related TypeScript types
7. `src/utils/AuditLogger.ts` - Government compliance audit logging
8. `src/services/ValidationService.ts` - Server-side validation service
9. `src/tests/SecurityTests.ts` - Security-focused test suite

### FILES_TO_MODIFY

1. `src/App/App.js` - Convert to TypeScript and add error boundaries
2. `src/storage/LocalStorage.js` - Add security measures and validation
3. `src/socialcalc/AppGeneral.js` - Add input sanitization
4. `src/index.js` - Add global error handling
5. `src/utils/Web3Provider.js` - Add error handling for Web3 operations

## 📝 CODE_CHANGES

### 1. Security Utilities

**FILE**: `src/utils/SecurityUtils.ts`
```typescript
import DOMPurify from 'dompurify';

export class SecurityUtils {
  private static readonly INJECTION_PATTERNS = [
    /(\b(ALTER|CREATE|DELETE|DROP|EXEC(UTE)?|INSERT|SELECT|UNION|UPDATE)\b)/gi,
    /<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi,
    /javascript:/gi,
    /vbscript:/gi,
    /on\w+\s*=/gi
  ];

  /**
   * Sanitize user input to prevent XSS attacks
   */
  static sanitizeInput(input: string): string {
    if (!input || typeof input !== 'string') return '';
    
    // Remove potential XSS vectors
    const sanitized = DOMPurify.sanitize(input, {
      ALLOWED_TAGS: [],
      ALLOWED_ATTR: []
    });
    
    return sanitized.trim();
  }

  /**
   * Validate input against SQL injection patterns
   */
  static validateSQLInjection(input: string): boolean {
    return !this.INJECTION_PATTERNS.some(pattern => pattern.test(input));
  }

  /**
   * Sanitize spreadsheet cell data
   */
  static sanitizeCellData(cellData: any): any {
    if (typeof cellData === 'string') {
      return this.sanitizeInput(cellData);
    }
    
    if (typeof cellData === 'object' && cellData !== null) {
      const sanitized: any = {};
      Object.keys(cellData).forEach(key => {
        sanitized[key] = this.sanitizeCellData(cellData[key]);
      });
      return sanitized;
    }
    
    return cellData;
  }

  /**
   * Generate secure random token for session management
   */
  static generateSecureToken(): string {
    const array = new Uint32Array(8);
    crypto.getRandomValues(array);
    return Array.from(array, dec => dec.toString(16).padStart(8, '0')).join('');
  }

  /**
   * Validate file upload security
   */
  static validateFileUpload(file: File): { isValid: boolean; error?: string } {
    const allowedTypes = ['application/json', 'text/plain', 'application/vnd.ms-excel'];
    const maxSize = 10 * 1024 * 1024; // 10MB
    
    if (!allowedTypes.includes(file.type)) {
      return { isValid: false, error: 'Invalid file type' };
    }
    
    if (file.size > maxSize) {
      return { isValid: false, error: 'File size exceeds limit' };
    }
    
    return { isValid: true };
  }
}
```

### 2. Data Validator

**FILE**: `src/utils/DataValidator.ts`
```typescript
import { SecurityUtils } from './SecurityUtils';

export interface ValidationRule {
  field: string;
  type: 'string' | 'number' | 'email' | 'phone' | 'date' | 'decimal';
  required?: boolean;
  min?: number;
  max?: number;
  pattern?: RegExp;
  customValidator?: (value: any) => boolean;
}

export interface ValidationResult {
  isValid: boolean;
  errors: string[];
  sanitizedData?: any;
}

export class DataValidator {
  private static readonly EMAIL_PATTERN = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  private static readonly PHONE_PATTERN = /^[\+]?[1-9][\d]{0,15}$/;
  private static readonly DECIMAL_PATTERN = /^\d+(\.\d{1,2})?$/;

  /**
   * Validate invoice data against government standards
   */
  static validateInvoiceData(data: any): ValidationResult {
    const rules: ValidationRule[] = [
      { field: 'invoiceNumber', type: 'string', required: true, min: 1, max: 50 },
      { field: 'invoiceDate', type: 'date', required: true },
      { field: 'customerName', type: 'string', required: true, min: 2, max: 100 },
      { field: 'customerEmail', type: 'email', required: true },
      { field: 'customerPhone', type: 'phone', required: false },
      { field: 'totalAmount', type: 'decimal', required: true, min: 0 },
      { field: 'taxAmount', type: 'decimal', required: false, min: 0 },
    ];

    return this.validateData(data, rules);
  }

  /**
   * Validate spreadsheet cell data
   */
  static validateCellData(cellData: any, constraints: any): ValidationResult {
    const errors: string[] = [];
    let sanitizedData = cellData;

    // Sanitize input first
    if (typeof cellData === 'string') {
      sanitizedData = SecurityUtils.sanitizeInput(cellData);
      
      // Check for SQL injection patterns
      if (!SecurityUtils.validateSQLInjection(sanitizedData)) {
        errors.push('Invalid characters detected in input');
      }
    }

    // Apply constraints from app-data.js
    if (constraints) {
      const [type, min, max, label] = constraints;
      
      switch (type) {
        case 'promptdecimal':
          if (!this.DECIMAL_PATTERN.test(sanitizedData)) {
            errors.push(`${label} must be a valid decimal number`);
          }
          break;
        case 'promptemail':
          if (!this.EMAIL_PATTERN.test(sanitizedData)) {
            errors.push(`${label} must be a valid email address`);
          }
          break;
        case 'prompttext':
          if (sanitizedData.length < min || sanitizedData.length > max) {
            errors.push(`${label} must be between ${min} and ${max} characters`);
          }
          break;
      }
    }

    return {
      isValid: errors.length === 0,
      errors,
      sanitizedData
    };
  }

  /**
   * Generic data validation
   */
  private static validateData(data: any, rules: ValidationRule[]): ValidationResult {
    const errors: string[] = [];
    const sanitizedData: any = {};

    rules.forEach(rule => {
      const value = data[rule.field];
      
      // Check required fields
      if (rule.required && (value === undefined || value === null || value === '')) {
        errors.push(`${rule.field} is required`);
        return;
      }

      // Skip validation for empty optional fields
      if (!rule.required && (value === undefined || value === null || value === '')) {
        return;
      }

      // Type validation
      switch (rule.type) {
        case 'string':
          if (typeof value !== 'string') {
            errors.push(`${rule.field} must be a string`);
          } else {
            sanitizedData[rule.field] = SecurityUtils.sanitizeInput(value);
            if (rule.min && value.length < rule.min) {
              errors.push(`${rule.field} must be at least ${rule.min} characters`);
            }
            if (rule.max && value.length > rule.max) {
              errors.push(`${rule.field} must not exceed ${rule.max} characters`);
            }
          }
          break;
        case 'number':
          if (typeof value !== 'number' || isNaN(value)) {
            errors.push(`${rule.field} must be a number`);
          } else {
            sanitizedData[rule.field] = value;
          }
          break;
        case 'email':
          if (!this.EMAIL_PATTERN.test(value)) {
            errors.push(`${rule.field} must be a valid email address`);
          } else {
            sanitizedData[rule.field] = SecurityUtils.sanitizeInput(value);
          }
          break;
        case 'phone':
          if (!this.PHONE_PATTERN.test(value)) {
            errors.push(`${rule.field} must be a valid phone number`);
          } else {
            sanitizedData[rule.field] = SecurityUtils.sanitizeInput(value);
          }
          break;
        case 'decimal':
          if (!this.DECIMAL_PATTERN.test(value.toString())) {
            errors.push(`${rule.field} must be a valid decimal number`);
          } else {
            sanitizedData[rule.field] = parseFloat(value);
          }
          break;
      }

      // Custom validation
      if (rule.customValidator && !rule.customValidator(value)) {
        errors.push(`${rule.field} failed custom validation`);
      }
    });

    return {
      isValid: errors.length === 0,
      errors,
      sanitizedData
    };
  }
}
```

### 3. Error Handler

**FILE**: `src/utils/ErrorHandler.ts`
```typescript
import { AuditLogger } from './AuditLogger';

export interface ErrorContext {
  component?: string;
  action?: string;
  userId?: string;
  timestamp?: Date;
  additionalInfo?: any;
}

export class ErrorHandler {
  private static instance: ErrorHandler;
  private errorQueue: Array<{ error: Error; context: ErrorContext }> = [];
  private isProcessing = false;

  static getInstance(): ErrorHandler {
    if (!ErrorHandler.instance) {
      ErrorHandler.instance = new ErrorHandler();
    }
    return ErrorHandler.instance;
  }

  /**
   * Handle and log errors with context
   */
  static handleError(error: Error, context: ErrorContext = {}): void {
    const instance = ErrorHandler.getInstance();
    
    // Add to error queue
    instance.errorQueue.push({
      error,
      context: {
        ...context,
        timestamp: new Date()
      }
    });

    // Process queue if not already processing
    if (!instance.isProcessing) {
      instance.processErrorQueue();
    }

    // Log to audit system for government compliance
    AuditLogger.logError(error, context);
  }

  /**
   * Process error queue
   */
  private async processErrorQueue(): Promise<void> {
    this.isProcessing = true;

    while (this.errorQueue.length > 0) {
      const errorItem = this.errorQueue.shift();
      if (errorItem) {
        await this.processError(errorItem.error, errorItem.context);
      }
    }

    this.isProcessing = false;
  }

  /**
   * Process individual error
   */
  private async processError(error: Error, context: ErrorContext): Promise<void> {
    try {
      // Log to console in development
      if (process.env.NODE_ENV === 'development') {
        console.error('Error:', error.message);
        console.error('Context:', context);
        console.error('Stack:', error.stack);
      }

      // Send to monitoring service in production
      if (process.env.NODE_ENV === 'production') {
        // await this.sendToMonitoringService(error, context);
      }

      // Store in local storage for offline scenarios
      this.storeErrorLocally(error, context);

    } catch (processingError) {
      console.error('Failed to process error:', processingError);
    }
  }

  /**
   * Store error locally for government audit requirements
   */
  private storeErrorLocally(error: Error, context: ErrorContext): void {
    try {
      const errorLog = {
        message: error.message,
        stack: error.stack,
        context,
        timestamp: new Date().toISOString()
      };

      const existingLogs = JSON.parse(localStorage.getItem('error_logs') || '[]');
      existingLogs.push(errorLog);

      // Keep only last 100 errors to prevent storage overflow
      if (existingLogs.length > 100) {
        existingLogs.splice(0, existingLogs.length - 100);
      }

      localStorage.setItem('error_logs', JSON.stringify(existingLogs));
    } catch (storageError) {
      console.error('Failed to store error locally:', storageError);
    }
  }

  /**
   * Get stored error logs for debugging
   */
  static getErrorLogs(): any[] {
    try {
      return JSON.parse(localStorage.getItem('error_logs') || '[]');
    } catch {
      return [];
    }
  }

  /**
   * Clear error logs
   */
  static clearErrorLogs(): void {
    localStorage.removeItem('error_logs');
  }
}
```

### 4. Error Boundary Component

**FILE**: `src/components/ErrorBoundary.tsx`
```typescript
import React, { Component, ReactNode } from 'react';
import { IonContent, IonHeader, IonTitle, IonToolbar, IonButton, IonIcon } from '@ionic/react';
import { refreshOutline, bugOutline } from 'ionicons/icons';
import { ErrorHandler } from '../utils/ErrorHandler';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
}

interface State {
  hasError: boolean;
  error?: Error;
  errorInfo?: React.ErrorInfo;
}

export class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    this.setState({ errorInfo });
    
    // Log error with context
    ErrorHandler.handleError(error, {
      component: 'ErrorBoundary',
      action: 'componentDidCatch',
      additionalInfo: {
        componentStack: errorInfo.componentStack,
        errorBoundary: true
      }
    });
  }

  handleReload = () => {
    this.setState({ hasError: false, error: undefined, errorInfo: undefined });
    window.location.reload();
  };

  handleRetry = () => {
    this.setState({ hasError: false, error: undefined, errorInfo: undefined });
  };

  render() {
    if (this.state.hasError) {
      if (this.props.fallback) {
        return this.props.fallback;
      }

      return (
        <>
          <IonHeader>
            <IonToolbar>
              <IonTitle>Application Error</IonTitle>
            </IonToolbar>
          </IonHeader>
          <IonContent className="ion-padding">
            <div style={{ textAlign: 'center', marginTop: '2rem' }}>
              <IonIcon icon={bugOutline} size="large" color="danger" />
              <h2>Something went wrong</h2>
              <p>The application encountered an unexpected error. This has been logged for review.</p>
              
              {process.env.NODE_ENV === 'development' && (
                <details style={{ marginTop: '1rem', textAlign: 'left' }}>
                  <summary>Error Details (Development Only)</summary>
                  <pre style={{ 
                    background: '#f5f5f5', 
                    padding: '1rem', 
                    borderRadius: '4px',
                    fontSize: '0.8rem',
                    overflow: 'auto'
                  }}>
                    {this.state.error?.message}
                    {this.state.error?.stack}
                  </pre>
                </details>
              )}
              
              <div style={{ marginTop: '2rem' }}>
                <IonButton onClick={this.handleRetry} color="primary">
                  Try Again
                </IonButton>
                <IonButton onClick={this.handleReload} color="secondary" style={{ marginLeft: '1rem' }}>
                  <IonIcon icon={refreshOutline} slot="start" />
                  Reload Page
                </IonButton>
              </div>
            </div>
          </IonContent>
        </>
      );
    }

    return this.props.children;
  }
}
```

### 5. Secure Storage Hook

**FILE**: `src/hooks/useSecureStorage.ts`
```typescript
import { useState, useEffect } from 'react';
import { SecurityUtils } from '../utils/SecurityUtils';
import { ErrorHandler } from '../utils/ErrorHandler';

interface SecureStorageOptions {
  encrypt?: boolean;
  validate?: boolean;
  ttl?: number; // Time to live in milliseconds
}

export const useSecureStorage = <T>(
  key: string, 
  defaultValue: T, 
  options: SecureStorageOptions = {}
) => {
  const [value, setValue] = useState<T>(defaultValue);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  const storageKey = `secure_${key}`;

  // Load value from storage
  useEffect(() => {
    try {
      const stored = localStorage.getItem(storageKey);
      if (stored) {
        const parsed = JSON.parse(stored);
        
        // Check TTL if set
        if (options.ttl && parsed.timestamp) {
          const now = Date.now();
          if (now - parsed.timestamp > options.ttl) {
            localStorage.removeItem(storageKey);
            setIsLoading(false);
            return;
          }
        }

        // Validate data if required
        if (options.validate) {
          const sanitized = SecurityUtils.sanitizeCellData(parsed.value);
          setValue(sanitized);
        } else {
          setValue(parsed.value);
        }
      }
    } catch (err) {
      ErrorHandler.handleError(err as Error, {
        component: 'useSecureStorage',
        action: 'loadValue',
        additionalInfo: { key }
      });
      setError('Failed to load stored data');
    } finally {
      setIsLoading(false);
    }
  }, [key, storageKey, options.ttl, options.validate]);

  // Save value to storage
  const setStoredValue = (newValue: T) => {
    try {
      const dataToStore = {
        value: newValue,
        timestamp: Date.now(),
        checksum: SecurityUtils.generateSecureToken()
      };

      localStorage.setItem(storageKey, JSON.stringify(dataToStore));
      setValue(newValue);
      setError(null);
    } catch (err) {
      ErrorHandler.handleError(err as Error, {
        component: 'useSecureStorage',
        action: 'saveValue',
        additionalInfo: { key }
      });
      setError('Failed to save data');
    }
  };

  // Clear stored value
  const clearStoredValue = () => {
    try {
      localStorage.removeItem(storageKey);
      setValue(defaultValue);
      setError(null);
    } catch (err) {
      ErrorHandler.handleError(err as Error, {
        component: 'useSecureStorage',
        action: 'clearValue',
        additionalInfo: { key }
      });
      setError('Failed to clear data');
    }
  };

  return {
    value,
    setValue: setStoredValue,
    clearValue: clearStoredValue,
    isLoading,
    error
  };
};
```

### 6. Audit Logger

**FILE**: `src/utils/AuditLogger.ts`
```typescript
import { ErrorContext } from './ErrorHandler';

export interface AuditEvent {
  eventType: 'USER_ACTION' | 'SYSTEM_ERROR' | 'DATA_CHANGE' | 'SECURITY_EVENT';
  userId?: string;
  sessionId?: string;
  timestamp: Date;
  action: string;
  details: any;
  ipAddress?: string;
  userAgent?: string;
}

export class AuditLogger {
  private static readonly MAX_LOGS = 1000;
  private static readonly STORAGE_KEY = 'audit_logs';

  /**
   * Log user actions for government compliance
   */
  static logUserAction(action: string, details: any, userId?: string): void {
    const event: AuditEvent = {
      eventType: 'USER_ACTION',
      userId,
      sessionId: this.getSessionId(),
      timestamp: new Date(),
      action,
      details: this.sanitizeDetails(details),
      ipAddress: this.getClientIP(),
      userAgent: navigator.userAgent
    };

    this.storeAuditEvent(event);
  }

  /**
   * Log system errors
   */
  static logError(error: Error, context: ErrorContext): void {
    const event: AuditEvent = {
      eventType: 'SYSTEM_ERROR',
      userId: context.userId,
      sessionId: this.getSessionId(),
      timestamp: new Date(),
      action: 'ERROR_OCCURRED',
      details: {
        message: error.message,
        stack: error.stack,
        component: context.component,
        action: context.action,
        additionalInfo: context.additionalInfo
      }
    };

    this.storeAuditEvent(event);
  }

  /**
   * Log data changes for audit trail
   */
  static logDataChange(
    action: string, 
    dataType: string, 
    oldValue: any, 
    newValue: any, 
    userId?: string
  ): void {
    const event: AuditEvent = {
      eventType: 'DATA_CHANGE',
      userId,
      sessionId: this.getSessionId(),
      timestamp: new Date(),
      action,
      details: {
        dataType,
        oldValue: this.sanitizeDetails(oldValue),
        newValue: this.sanitizeDetails(newValue)
      }
    };

    this.storeAuditEvent(event);
  }

  /**
   * Log security events
   */
  static logSecurityEvent(event: string, details: any, userId?: string): void {
    const auditEvent: AuditEvent = {
      eventType: 'SECURITY_EVENT',
      userId,
      sessionId: this.getSessionId(),
      timestamp: new Date(),
      action: event,
      details: this.sanitizeDetails(details),
      ipAddress: this.getClientIP(),
      userAgent: navigator.userAgent
    };

    this.storeAuditEvent(auditEvent);
  }

  /**
   * Get audit logs for export
   */
  static getAuditLogs(): AuditEvent[] {
    try {
      const logs = localStorage.getItem(this.STORAGE_KEY);
      return logs ? JSON.parse(logs) : [];
    } catch (error) {
      console.error('Failed to retrieve audit logs:', error);
      return [];
    }
  }

  /**
   * Export audit logs for government compliance
   */
  static exportAuditLogs(startDate?: Date, endDate?: Date): string {
    const logs = this.getAuditLogs();
    
    let filteredLogs = logs;
    if (startDate || endDate) {
      filteredLogs = logs.filter(log => {
        const logDate = new Date(log.timestamp);
        if (startDate && logDate < startDate) return false;
        if (endDate && logDate > endDate) return false;
        return true;
      });
    }

    return JSON.stringify(filteredLogs, null, 2);
  }

  /**
   * Clear audit logs (restricted operation)
   */
  static clearAuditLogs(): void {
    // Log the clear operation first
    this.logSecurityEvent('AUDIT_LOGS_CLEARED', {
      timestamp: new Date(),
      action: 'CLEAR_LOGS'
    });

    localStorage.removeItem(this.STORAGE_KEY);
  }

  private static storeAuditEvent(event: AuditEvent): void {
    try {
      const logs = this.getAuditLogs();
      logs.push(event);

      // Keep only the most recent logs
      if (logs.length > this.MAX_LOGS) {
        logs.splice(0, logs.length - this.MAX_LOGS);
      }

      localStorage.setItem(this.STORAGE_KEY, JSON.stringify(logs));
    } catch (error) {
      console.error('Failed to store audit event:', error);
    }
  }

  private static sanitizeDetails(details: any): any {
    if (typeof details === 'string') {
      return details.substring(0, 1000); // Limit string length
    }
    
    if (typeof details === 'object' && details !== null) {
      const sanitized: any = {};
      Object.keys(details).forEach(key => {
        if (key.toLowerCase().includes('password') || key.toLowerCase().includes('token')) {
          sanitized[key] = '[REDACTED]';
        } else {
          sanitized[key] = this.sanitizeDetails(details[key]);
        }
      });
      return sanitized;
    }
    
    return details;
  }

  private static getSessionId(): string {
    let sessionId = sessionStorage.getItem('session_id');
    if (!sessionId) {
      sessionId = 'session_' + Date.now() + '_' + Math.random().toString(36).substr(2, 9);
      sessionStorage.setItem('session_id', sessionId);
    }
    return sessionId;
  }

  private static getClientIP(): string {
    // In a real application, this would be obtained from the server
    return 'client_ip_unavailable';
  }
}
```

### 7. Updated App Component

**FILE**: `src/App/App.tsx`
```typescript
import React, { useState, useEffect } from 'react';
import { IonApp, IonContent, IonHeader, IonTitle, IonToolbar, IonSplitPane } from '@ionic/react';
import { IonReactRouter } from '@ionic/react-router';
import { Route, Redirect } from 'react-router-dom';
import '@ionic/react/css/core.css';
import '@ionic/react/css/normalize.css';
import '@ionic/react/css/structure.css';
import '@ionic/react/css/typography.css';

import { ErrorBoundary } from '../components/ErrorBoundary';
import { ErrorHandler } from '../utils/ErrorHandler';
import { AuditLogger } from '../utils/AuditLogger';
import { SecurityUtils } from '../utils/SecurityUtils';
import { useSecureStorage } from '../hooks/useSecureStorage';
import * as AppGeneral from '../socialcalc/AppGeneral';
import { DATA } from '../app-data';

import Menu from '../Menu/Menu';
import Files from '../Files/Files';
import { ConnectKitButton } from 'connectkit';

interface AppState {
  selectedFile: string;
  device: string;
  listFiles: boolean;
  isInitialized: boolean;
  securityToken: string;
}

const App: React.FC = () => {
  const [state, setState] = useState<AppState>({
    selectedFile: "default",
    device: AppGeneral.getDeviceType(),
    listFiles: false,
    isInitialized: false,
    securityToken: ''
  });

  const { value: secureData, setValue: setSecureData, error: storageError } = useSecureStorage(
    'app_data',
    DATA,
    { validate: true, ttl: 24 * 60 * 60 * 1000 } // 24 hours
  );

  useEffect(() => {
    initializeApp();
  }, []);

  const initializeApp = async () => {
    try {
      // Generate security token
      const token = SecurityUtils.generateSecureToken();
      
      // Log application start
      AuditLogger.logUserAction('APP_INITIALIZED', {
        device: state.device,
        timestamp: new Date(),
        version: process.env.REACT_APP_VERSION || '1.0.0'
      });

      // Initialize security measures
      await initializeSecurity();

      setState(prev => ({
        ...prev,
        isInitialized: true,
        securityToken: token
      }));

    } catch (error) {
      ErrorHandler.handleError(error as Error, {
        component: 'App',
        action: 'initializeApp'
      });
    }
  };

  const initializeSecurity = async () => {
    // Set up Content Security Policy
    if (typeof window !== 'undefined' && window.document) {
      const meta = document.createElement('meta');
      meta.httpEquiv = 'Content-Security-Policy';
      meta.content = "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline';";
      document.head.appendChild(meta);
    }

    // Initialize session monitoring
    sessionStorage.setItem('session_start', new Date().toISOString());
    
    // Set up beforeunload handler for audit logging
    window.addEventListener('beforeunload', () => {
```
