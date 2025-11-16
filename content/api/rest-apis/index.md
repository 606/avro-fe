---
{"publish":true,"aliases":"Rest-Apis Rest Apis Restful-Apis","title":"Rest Apis","enableToc":true,"created":"2025-11-16T00:25:07.529+02:00","modified":"2025-11-16T15:51:15.019+02:00","tags":["api","rest","http"],"cssclasses":""}
---


## Overview

REST (Representational State Transfer) APIs are the most common type of web APIs, following HTTP principles and using standard HTTP methods.

## Core Principles

### Stateless
Each request contains all information needed to process it. No server-side session state.

### Client-Server Architecture
Clear separation between client and server concerns.

### Cacheable
Responses must define themselves as cacheable or not.

### Uniform Interface
Consistent resource identification and manipulation through representations.

### Layered System
Client cannot tell if it's connected directly to server or through intermediaries.

## HTTP Methods

### GET
- **Purpose**: Retrieve resource(s)
- **Safe**: Yes (doesn't modify data)
- **Idempotent**: Yes
- **Body**: No
- **Example**: `GET /users/123`

### POST
- **Purpose**: Create new resource
- **Safe**: No
- **Idempotent**: No
- **Body**: Yes
- **Example**: `POST /users`

### PUT
- **Purpose**: Update/replace entire resource
- **Safe**: No
- **Idempotent**: Yes
- **Body**: Yes
- **Example**: `PUT /users/123`

### PATCH
- **Purpose**: Partial resource update
- **Safe**: No
- **Idempotent**: No (usually)
- **Body**: Yes
- **Example**: `PATCH /users/123`

### DELETE
- **Purpose**: Remove resource
- **Safe**: No
- **Idempotent**: Yes
- **Body**: No
- **Example**: `DELETE /users/123`

## HTTP Status Codes

### 2xx Success
- **200 OK**: Request succeeded
- **201 Created**: Resource created
- **204 No Content**: Success, no response body

### 3xx Redirection
- **301 Moved Permanently**: Resource moved permanently
- **302 Found**: Resource moved temporarily
- **304 Not Modified**: Resource not modified (caching)

### 4xx Client Error
- **400 Bad Request**: Invalid request syntax
- **401 Unauthorized**: Authentication required
- **403 Forbidden**: Access denied
- **404 Not Found**: Resource not found
- **422 Unprocessable Entity**: Validation errors

### 5xx Server Error
- **500 Internal Server Error**: Unexpected error
- **502 Bad Gateway**: Invalid response from upstream
- **503 Service Unavailable**: Server temporarily unavailable

## Resource Design

### Naming Conventions
- Use nouns, not verbs: `/users` not `/getUsers`
- Use plural nouns: `/users` not `/user`
- Use lowercase with hyphens: `/user-profiles`

### Nested Resources
- `/users/123/posts` - User's posts
- `/users/123/posts/456` - Specific post
- Avoid deep nesting (max 3 levels)

### Query Parameters
- Filtering: `GET /users?status=active`
- Sorting: `GET /users?sort=name`
- Pagination: `GET /users?page=2&limit=10`

## Best Practices

### Versioning
- URL versioning: `/v1/users`
- Header versioning: `Accept: application/vnd.api.v1+json`
- Query parameter: `/users?version=1`

### Content Negotiation
- Accept header for response format
- Content-Type header for request format
- Support JSON, XML, etc.

### Error Handling
- Consistent error response format
- Include error codes and messages
- Provide helpful debugging information

### Security
- Use HTTPS
- Implement authentication
- Validate input data
- Rate limiting

## Detailed Topics

### [[api/rest-apis/http-methods/index\|Http Methods Deep Dive]]
Comprehensive guide to HTTP methods, their semantics, safety, idempotency, and implementation patterns.

### [[api/rest-apis/status-codes/index\|Http Status Codes]]
Complete reference of HTTP status codes with detailed explanations, use cases, and best practices.

### [[api/rest-apis/content-negotiation/index\|Content Negotiation]]
How APIs negotiate data formats, media types, and client preferences using HTTP headers.

### [[api/rest-apis/hypermedia/index\|Hypermedia Apis (HATEOAS)]]
Building self-descriptive APIs with links, hypermedia formats, and dynamic client navigation.

## Related
- [[api/api-design/index\|Api Design Principles]]
- [[api/api-security/index\|Api Security]]
- [[api/api-testing/index\|Api Testing]]
- [[api/api-documentation/index\|Api Documentation]]
