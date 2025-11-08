---
name: norsktorget-api-design
description: Use when creating or modifying backend API endpoints for the Norsktorget project. This skill provides the official API design guide, covering RESTful principles, versioning, endpoint structure, request/response formats, error handling, and security patterns specific to the NestJS backend.
---

# Norsktorget API Design Guide

## Overview

This skill is the single source of truth for designing and implementing all backend API endpoints in the Norsktorget project. It ensures that our API is consistent, predictable, secure, and easy to use for frontend developers.

**Core Principle:** All API endpoints MUST adhere to the RESTful principles and data structures defined in this guide.

## When to Use

- When creating a new set of API endpoints for a feature.
- When modifying an existing endpoint.
- When designing DTOs (Data Transfer Objects) for requests and responses.
- When implementing error handling and status codes.
- When reviewing a pull request that includes API changes.

## 1. RESTful Principles & Endpoint Structure

Our API follows RESTful conventions.

### Resource Naming

- **Use plural nouns:** `/users`, `/ads`, `/categories`.
- **Use kebab-case:** `/ad-drafts`, `/category-fields`.
- **Never use verbs:** ❌ `/getUser`, ❌ `/createAd`.

### Standard HTTP Methods

| Method | URL | Action |
| :--- | :--- | :--- |
| `GET` | `/resources` | List resources (collection) |
| `GET` | `/resources/{id}` | Get a single resource |
| `POST` | `/resources` | Create a new resource |
| `PUT` | `/resources/{id}` | Replace a resource entirely |
| `PATCH` | `/resources/{id}` | Partially update a resource |
| `DELETE` | `/resources/{id}` | Delete a resource |

### Nested Resources

For clear relationships, nest resources up to one level deep.

```
# Get all ads for a specific category
GET /categories/{categoryId}/ads

# Get comments for a specific ad
GET /ads/{adId}/comments
```

### Actions (Non-CRUD)

For actions that don't fit CRUD, use a sub-resource like `/actions` or a verb at the end of the path.

```
# Publish a draft
POST /ad-drafts/{draftId}/publish

# Analyze a draft with AI
POST /ad-drafts/{draftId}/analyze
```

## 2. API Versioning

All endpoints **must** be versioned. The version is part of the URL prefix.

- **Current Version:** `/api/v1`
- **Example:** `GET /api/v1/users`

## 3. Data Transfer Objects (DTOs)

### Request DTOs

- **Specificity:** Create specific DTOs for each operation. Do not reuse entity models directly as DTOs.
- **Security:** DTOs for updates (`UpdateUserDto`) **must not** include sensitive or protected fields like `role`, `passwordHash`, or `id`.
- **Validation:** All request DTOs **must** use `class-validator` decorators for robust validation.

```typescript
// ❌ BAD: Allows updating role
export class UpdateUserDto {
  @IsOptional() @IsString()
  fullName?: string;

  @IsOptional() @IsEnum(UserRole) // SECURITY RISK!
  role?: UserRole;
}

// ✅ GOOD: Specific and secure
export class UpdateUserProfileDto {
  @IsOptional() @IsString() @Length(2, 50)
  fullName?: string;

  @IsOptional() @IsString()
  bio?: string;
}
```

### Response DTOs

- **Clarity:** Use DTOs to shape the data returned to the client, hiding internal fields.
- **Consistency:** Ensure all endpoints returning a specific resource use the same response DTO.

## 4. Standard Response Formats

### Success Response

For single resources or collections, wrap the data in a `data` property.

```json
// GET /users/{id}
{
  "data": {
    "id": "uuid-123",
    "fullName": "Ola Nordmann",
    "email": "ola@example.com"
  }
}

// GET /users
{
  "data": [
    { "id": "uuid-123", "fullName": "Ola Nordmann" },
    { "id": "uuid-456", "fullName": "Kari Nordmann" }
  ],
  "meta": {
    "page": 1,
    "limit": 20,
    "totalItems": 2,
    "totalPages": 1
  }
}
```

**Reference:** See `references/pagination.md` for detailed pagination implementation.

### Error Response (CRITICAL PATTERN)

All error responses must follow this standardized structure.

```json
{
  "error": {
    "code": "VALIDATION_ERROR", // Machine-readable error code
    "message": "Invalid input data provided.", // Human-readable summary
    "details": { // Optional, context-specific details
      "email": "email must be an email",
      "password": "password is too short"
    }
  }
}
```

**Reference:** See `references/error-codes.md` for a complete list of standard error codes.

## 5. Authentication & Authorization

- **Authentication:** All endpoints (except public ones like `GET /ads` or `POST /auth/login`) must be protected with `JwtAuthGuard`.
- **Authorization:** Use `RolesGuard` for role-based access (e.g., admin-only endpoints). For resource ownership, create specific guards like `AdOwnerGuard`.

```typescript
// ✅ GOOD: Protected and authorized endpoint
@Controller('admin/categories')
@UseGuards(JwtAuthGuard, RolesGuard)
@Roles(UserRole.ADMIN)
export class CategoryAdminController {
  // ...
}
```

## 6. Pagination

For all collection endpoints (`GET /resources`), pagination is mandatory.

**Reference:** The full pagination pattern is detailed in `references/pagination.md`.
