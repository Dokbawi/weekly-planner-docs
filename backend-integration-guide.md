# Backend Integration Guide

**Version:** 2.0
**Last Updated:** 2026-01-14
**Target Audience:** Frontend Developers

This guide documents the backend API integration requirements and current implementation status.

---

## Current API Implementation Status

### ✅ Fully Implemented

- `POST /auth/register` - User registration
- `POST /auth/login` - User authentication
- `GET /auth/me` - Get current user info
- `PUT /auth/settings` - Update user settings
- `POST /plans` - Create weekly plan
- `GET /plans` - List weekly plans (paginated)
- `GET /plans/current` - Get current week's plan (auto-create if not exists)
- `GET /plans/by-date` - Get plan by specific date
- `GET /plans/{planId}` - Get specific plan
- `POST /plans/{planId}/confirm` - Confirm plan
- `PUT /plans/{planId}/memo` - Update daily memo
- `POST /plans/{planId}/tasks` - Create task (with date query param)
- `PUT /plans/{planId}/tasks/{taskId}` - Update task
- `POST /plans/{planId}/tasks/{taskId}/move` - Move task to another day
- `DELETE /plans/{planId}/tasks/{taskId}` - Delete task
- `GET /plans/{planId}/changes` - Get change logs
- `GET /reviews/{planId}` - Get weekly review
- `GET /notifications` - List notifications
- `GET /notifications/unread/count` - Get unread count
- `POST /notifications/{notificationId}/read` - Mark as read
- `POST /notifications/read-all` - Mark all as read
- `GET /today` - Get today's tasks

---

## Response Format

### Standard Response Wrapper

All responses use this format:

**Success:**
```json
{
  "success": true,
  "data": { ... }
}
```

**Error:**
```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Human readable message"
  }
}
```

### Paginated Responses

```json
{
  "success": true,
  "data": {
    "content": [...],
    "page": 0,
    "size": 10,
    "totalElements": 100,
    "totalPages": 10
  }
}
```

---

## Key Implementation Details

### Authentication

**Login Response:**
```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIs...",
    "tokenType": "Bearer",
    "expiresIn": 86400
  }
}
```

**JWT Claims:**
- `sub` - User ID
- `email` - User email
- `name` - User name
- `exp` - Expiration timestamp
- `iat` - Issued at timestamp

### Task Creation

**Request:** `POST /plans/{planId}/tasks?date=2025-01-12`
```json
{
  "title": "Task title",
  "description": "Optional description",
  "scheduledTime": "14:00",
  "reminderMinutesBefore": 30,
  "priority": "HIGH",
  "tags": ["work"]
}
```

**Note:** `date` is passed as a query parameter, not in the request body.

### Task Priority Values
- `LOW`
- `MEDIUM`
- `HIGH`

### Task Status Values
- `PENDING`
- `IN_PROGRESS`
- `COMPLETED`
- `CANCELLED`
- `POSTPONED`

### Plan Status Values
- `DRAFT`
- `CONFIRMED`

---

## Frontend Data Normalization

The frontend normalizes backend response data to handle format differences:

### dailyPlans Array to Object Conversion

Backend returns `dailyPlans` as an array, but frontend expects an object (Record):

```typescript
// Backend response (array)
dailyPlans: [
  { date: '2026-01-12', tasks: [] },
  { date: '2026-01-13', tasks: [...] },
]

// Frontend expected format (object)
dailyPlans: {
  '2026-01-12': { date: '...', tasks: [] },
  '2026-01-13': { date: '...', tasks: [...] },
}
```

**Implementation:** See `normalizeDailyPlans()` in `src/api/plans.ts`

### Review dailyBreakdown Conversion

Same pattern applies to `dailyBreakdown` in review responses.

**Implementation:** See `normalizeReview()` in `src/api/reviews.ts`

---

## Common Integration Issues

### Issue 1: "Weekly plan already exists" Error

**Symptom:** 400 error when creating a plan

**Cause:** A plan for that week already exists

**Solution:** Use `GET /plans/current` instead of `POST /plans` to get or create the current week's plan.

### Issue 2: 401 Unauthorized

**Symptom:** API calls fail with 401

**Debug:**
1. Check token in localStorage
2. Verify Authorization header format: `Bearer {token}`
3. Check token expiration

### Issue 3: Task Date Mismatch

**Symptom:** Task appears on wrong day

**Cause:** Date should be in query param, not body

**Fix:** Use `POST /plans/{planId}/tasks?date=2025-01-12`

### Issue 4: Notification Read Method

**Method:** Use PUT (not POST)

```
PUT /notifications/{notificationId}/read
PUT /notifications/read-all
```

See [API Contract](./api-contract.md#6-notifications) for the authoritative specification.

---

## Related Documentation

- [API Contract](./api-contract.md) - Complete REST API specification
- [Domain Model](./domain-model.md) - Entity definitions
- [Business Rules](./business-rules.md) - Business logic rules

---

**Last Updated:** 2026-01-14
