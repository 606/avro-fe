---
{"publish":true,"aliases":"Api-Testing Api Testing Api-Test","title":"Api Testing","enableToc":true,"created":"2025-11-16T00:25:07.657+02:00","modified":"2025-11-16T15:51:15.019+02:00","tags":["api","testing","quality-assurance"],"cssclasses":""}
---


## Overview

API testing involves validating the functionality, performance, security, and reliability of APIs through various testing methodologies and tools.

## Testing Types

### Unit Testing
- **Scope**: Individual functions/methods
- **Tools**: JUnit, pytest, Jest
- **Focus**: Business logic validation

### Integration Testing
- **Scope**: API endpoints and external services
- **Tools**: Postman, RestAssured, Supertest
- **Focus**: Data flow and service interactions

### Contract Testing
- **Scope**: API contracts between services
- **Tools**: Pact, Spring Cloud Contract
- **Focus**: API compatibility

### End-to-End Testing
- **Scope**: Complete user workflows
- **Tools**: Cypress, Playwright with API testing
- **Focus**: User journey validation

### Performance Testing
- **Scope**: Load, stress, and scalability
- **Tools**: JMeter, k6, Artillery
- **Focus**: Response times and throughput

### Security Testing
- **Scope**: Authentication, authorization, vulnerabilities
- **Tools**: OWASP ZAP, Burp Suite
- **Focus**: Security weaknesses

## Testing Pyramid

```
End-to-End Tests (Slow, Expensive)
    ↕️
Integration Tests (Medium)
    ↕️
Unit Tests (Fast, Cheap)
```

## API Testing Checklist

### Functional Testing
- [ ] All endpoints return correct status codes
- [ ] Request/response formats match specifications
- [ ] Business logic works as expected
- [ ] Error handling is appropriate
- [ ] Edge cases are covered

### Data Validation
- [ ] Required fields are validated
- [ ] Data types are correct
- [ ] Length/format constraints enforced
- [ ] Default values work properly
- [ ] Null/empty values handled

### Authentication & Authorization
- [ ] Valid credentials work
- [ ] Invalid credentials rejected
- [ ] Proper permissions enforced
- [ ] Token expiration handled
- [ ] Role-based access works

### Error Handling
- [ ] Appropriate error codes returned
- [ ] Error messages are helpful
- [ ] Sensitive data not exposed
- [ ] Consistent error format

## Testing Tools

### Postman
```javascript
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

pm.test("Response has required fields", function () {
    const jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('id');
    pm.expect(jsonData).to.have.property('name');
});
```

### REST Assured (Java)
```java
given()
    .header("Authorization", "Bearer " + token)
    .when()
    .get("/api/users/1")
    .then()
    .statusCode(200)
    .body("name", equalTo("John Doe"));
```

### Supertest (Node.js)
```javascript
const request = require('supertest');
const app = require('../app');

describe('GET /api/users', () => {
  it('should return users list', async () => {
    const res = await request(app)
      .get('/api/users')
      .expect(200);
    
    expect(res.body).toBeInstanceOf(Array);
  });
});
```

### Newman (Postman CLI)
```bash
# Run Postman collection
newman run collection.json -e environment.json

# Generate HTML report
newman run collection.json -r html
```

## Test Automation

### CI/CD Integration
```yaml
# GitHub Actions example
- name: API Tests
  run: |
    npm install -g newman
    newman run api-tests.postman_collection.json
```

### Test Data Management
- **Factories**: Generate test data
- **Fixtures**: Predefined test data
- **Mocking**: External service simulation
- **Seeding**: Database test data

## Performance Testing

### Load Testing
- **Concurrent Users**: Simulate multiple users
- **Ramp-up Period**: Gradual load increase
- **Duration**: Sustained load testing

### Stress Testing
- **Breaking Point**: Find system limits
- **Recovery**: Test system recovery
- **Scalability**: Performance under extreme load

### Metrics to Monitor
- Response time (p50, p95, p99)
- Throughput (requests/second)
- Error rate
- Resource utilization (CPU, memory, disk)

## Contract Testing

### Consumer-Driven Contracts
```javascript
// Pact example
const { Pact } = require('@pact-foundation/pact');

describe('User API', () => {
  const provider = new Pact({...});
  
  it('returns user data', () => {
    return provider.addInteraction({...})
      .then(() => {
        return request.get('/users/1');
      });
  });
});
```

## Mocking & Stubbing

### WireMock
```json
{
  "request": {
    "method": "GET",
    "url": "/api/external-service"
  },
  "response": {
    "status": 200,
    "jsonBody": {
      "data": "mocked response"
    }
  }
}
```

### MockServer
- HTTP/HTTPS mocking
- Request matching
- Response templating
- Verification capabilities

## Best Practices

### Test Organization
- Group tests by functionality
- Use descriptive test names
- Follow AAA pattern (Arrange, Act, Assert)
- Keep tests independent

### Test Data
- Use realistic test data
- Avoid test data pollution
- Clean up after tests
- Use factories for complex objects

### CI/CD Integration
- Run tests on every commit
- Parallel test execution
- Proper test reporting
- Fail fast on critical issues

### Maintenance
- Update tests with API changes
- Remove obsolete tests
- Refactor test code
- Document test scenarios

## Related
- [[api/api-design/index\|Api Design]]
- [[api/api-security/index\|Api Security]]
- [[../testing/index\|Testing Strategies]]
- [[../devops/index\|Ci/Cd Pipelines]]
