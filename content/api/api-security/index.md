---
{"publish":true,"aliases":"api-security API Security api-protection","title":"API Security","enableToc":true,"created":"2025-11-16T00:25:07.646+02:00","modified":"2025-11-16T00:26:46.885+02:00","tags":["api","security","authentication","authorization"],"cssclasses":""}
---


## Overview

API security encompasses authentication, authorization, data protection, and security best practices for APIs.

## Authentication Methods

### API Keys
- **Description**: Simple token-based authentication
- **Usage**: `Authorization: Bearer <api-key>`
- **Pros**: Simple to implement
- **Cons**: Less secure, hard to revoke

### Basic Authentication
- **Description**: HTTP Basic Auth with username:password
- **Usage**: `Authorization: Basic <base64-encoded>`
- **Pros**: Built into HTTP
- **Cons**: Credentials sent with every request

### JWT (JSON Web Tokens)
- **Description**: Stateless token-based authentication
- **Structure**: Header.Payload.Signature
- **Pros**: Stateless, self-contained
- **Cons**: Token size, revocation challenges

### OAuth 2.0
- **Description**: Authorization framework
- **Flows**: Authorization Code, Implicit, Client Credentials
- **Pros**: Delegated authorization
- **Cons**: Complex implementation

### OpenID Connect
- **Description**: Identity layer on top of OAuth 2.0
- **Features**: User authentication, profile information
- **Pros**: Standardized identity
- **Cons**: Additional complexity

## Authorization Patterns

### Role-Based Access Control (RBAC)
- Users assigned to roles
- Roles have permissions
- Simple and scalable

### Attribute-Based Access Control (ABAC)
- Policies based on attributes
- Fine-grained control
- Complex but flexible

### API Gateway Authorization
- Centralized authorization logic
- Policy enforcement
- Request routing

## Security Best Practices

### Transport Security
- **HTTPS Only**: Always use TLS 1.3+
- **Certificate Pinning**: Prevent MITM attacks
- **HSTS**: Force HTTPS connections

### Input Validation
- **Sanitize Input**: Prevent injection attacks
- **Validate Schema**: Use JSON Schema validation
- **Type Checking**: Strong typing for parameters

### Rate Limiting
- **Brute Force Protection**: Limit login attempts
- **DDoS Prevention**: Request throttling
- **API Abuse Prevention**: Fair usage policies

### Data Protection
- **Encryption**: Encrypt sensitive data at rest
- **Masking**: Hide sensitive data in logs
- **PII Handling**: GDPR/CCPA compliance

## Common Vulnerabilities

### OWASP API Top 10
1. **Broken Object Level Authorization**: Accessing unauthorized resources
2. **Broken Authentication**: Weak authentication mechanisms
3. **Broken Object Property Level Authorization**: Accessing unauthorized properties
4. **Unrestricted Resource Consumption**: Resource exhaustion attacks
5. **Broken Function Level Authorization**: Privilege escalation
6. **Unrestricted Access to Sensitive Business Flows**: Business logic bypass
7. **Server Side Request Forgery**: SSRF attacks
8. **Security Misconfiguration**: Default configurations
9. **Improper Inventory Management**: Unknown API endpoints
10. **Unsafe Consumption of APIs**: Third-party API risks

## Security Headers

### Essential Headers
```
Content-Security-Policy: default-src 'self'
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Strict-Transport-Security: max-age=31536000
```

### API-Specific Headers
```
X-API-Key: <key>
X-Rate-Limit-Limit: 100
X-Rate-Limit-Remaining: 95
X-Rate-Limit-Reset: 1640995200
```

## API Key Management

### Key Generation
- Use cryptographically secure random generators
- Sufficient entropy (256+ bits)
- Unique per client/application

### Key Storage
- Never store in plain text
- Use secure vaults (AWS Secrets Manager, HashiCorp Vault)
- Rotate regularly

### Key Distribution
- Secure channels only
- Documentation for proper usage
- Revocation procedures

## Monitoring & Logging

### Security Events
- Authentication failures
- Authorization denials
- Rate limit violations
- Suspicious patterns

### Audit Logging
- Who accessed what, when
- Compliance requirements
- Forensic analysis

### Alerting
- Real-time security alerts
- Automated responses
- Incident response procedures

## Testing Security

### Security Testing Types
- **SAST**: Static Application Security Testing
- **DAST**: Dynamic Application Security Testing
- **IAST**: Interactive Application Security Testing
- **Penetration Testing**: Manual security assessment

### API Security Testing
- Authentication bypass attempts
- Authorization testing
- Input validation testing
- Injection attack testing

## Tools & Frameworks

### Authentication Libraries
- **Passport.js** (Node.js)
- **Spring Security** (Java)
- **Django OAuth Toolkit** (Python)
- **IdentityServer** (.NET)

### Security Scanners
- **OWASP ZAP**
- **Burp Suite**
- **Postman Security**
- **42Crunch API Security Audit**

### API Gateways
- **Kong**
- **Apigee**
- **AWS API Gateway**
- **Azure API Management**

## Related
- [[api/api-design/index\|API Design]]
- [[api/api-testing/index\|API Testing]]
- [[api/api-management/index\|API Management]]
- [[../devops/index\|DevSecOps]]
