# Norsktorget API Error Codes

This document lists the standard machine-readable error codes to be used in all API error responses.

## Authentication & Authorization (AUTH_xxxx)

| Code | HTTP Status | Message |
| :--- | :--- | :--- |
| `AUTH_001` | 401 | `Authentication failed. Invalid credentials.` |
| `AUTH_002` | 401 | `Invalid or expired token.` |
| `AUTH_003` | 403 | `Insufficient permissions.` |

## Validation (VALIDATION_xxxx)

| Code | HTTP Status | Message |
| :--- | :--- | :--- |
| `VALIDATION_001`| 400 | `Invalid input data provided.` |
| `VALIDATION_002`| 422 | `A resource with this value already exists.` (Unprocessable Entity) |

## Resource (RESOURCE_xxxx)

| Code | HTTP Status | Message |
| :--- | :--- | :--- |
| `RESOURCE_001`| 404 | `The requested resource was not found.` |

## Server (SERVER_xxxx)

| Code | HTTP Status | Message |
| :--- | :--- | :--- |
| `SERVER_001` | 500 | `An unexpected internal server error occurred.` |
| `SERVER_002` | 503 | `The service is temporarily unavailable.` |
