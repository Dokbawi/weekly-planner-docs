# Backend Integration Guide

**Version:** 1.0
**Last Updated:** 2025-12-29
**Target Audience:** Backend Developers

This guide documents the frontend's API integration requirements and current implementation status. Use this to ensure your backend API matches frontend expectations.

---

## Table of Contents

1. [Current API Implementation Status](#current-api-implementation-status)
2. [Critical Endpoints Required](#critical-endpoints-required)
3. [API Contract Changes from Frontend Integration](#api-contract-changes-from-frontend-integration)
4. [Response Format Requirements](#response-format-requirements)
5. [Authentication Implementation](#authentication-implementation)
6. [Task Management Implementation](#task-management-implementation)
7. [Testing Your API](#testing-your-api)
8. [Common Integration Issues](#common-integration-issues)

---

## Current API Implementation Status

### ✅ Fully Implemented (Required)

These endpoints MUST be implemented for the frontend to work:

- `POST /auth/register` - User registration
- `POST /auth/login` - User authentication
- `GET /auth/me` - Get current user info
- `PUT /auth/settings` - Update user settings
- `POST /plans` - Create weekly plan
- `GET /plans` - List weekly plans (paginated)
- `GET /plans/{planId}` - Get specific plan
- `POST /plans/{planId}/confirm` - Confirm plan
- `PUT /plans/{planId}/memo` - Update daily memo
- `GET /plans/{planId}/tasks` - Get tasks for a plan
- `POST /plans/{planId}/tasks` - Create task (with date query param)
- `PUT /plans/{planId}/tasks/{taskId}` - Update task
- `POST /plans/{planId}/tasks/{taskId}/move` - Move task to another day
- `DELETE /plans/{planId}/tasks/{taskId}` - Delete task
- `GET /plans/{planId}/changes` - Get change logs
- `GET /plans/{planId}/review` - Get weekly review
- `GET /notifications` - List notifications
- `GET /notifications/unread/count` - Get unread count
- `POST /notifications/{notificationId}/read` - Mark as read
- `POST /notifications/read-all` - Mark all as read

### ⚠️ Missing (Convenience Endpoints)

These endpoints are NOT required but would improve performance:

- `GET /plans/current` - Get current week's plan directly
- `GET /today` - Get today's tasks and statistics
- `GET /reviews/current` - Get current week's review directly
- `GET /reviews/{weekStartDate}` - Get review by week start date

**Frontend Workaround:**
The frontend implements these missing endpoints by:
1. Fetching the plans list (`GET /plans`)
2. Filtering to find the current week's plan
3. Extracting the relevant data

This works but requires multiple API calls. Implementing these convenience endpoints would reduce network overhead.

---

## Critical Endpoints Required

### 1. GET /plans/current

**Why It's Important:**
Called on every page load for authenticated users to get the current week's plan.

**Expected Behavior:**
- If current week's plan exists, return it
- If not, automatically create and return new plan for current week
- Should never return 404

**Current Frontend Workaround:**

See implementation in frontend repository: `src/api/plans.ts` (getCurrent method)

The frontend calculates the current week's Monday and fetches the plans list to find the matching plan.

**Recommended Implementation:**

See backend implementation reference: `src/plan/plan.controller.ts` and `src/plan/plan.service.ts`

The backend should find or create a plan for the current week based on the user's week start day setting.

### 2. GET /today

**Why It's Important:**
Main endpoint for the "Today" page - one of the most frequently accessed pages.

**Expected Response:**
```json
{
  "success": true,
  "data": {
    "date": "2024-01-01",
    "dayOfWeek": "MONDAY",
    "planId": "plan_123",
    "planStatus": "CONFIRMED",
    "tasks": [...],
    "memo": "Focus on important tasks",
    "statistics": {
      "total": 5,
      "completed": 2,
      "pending": 2,
      "inProgress": 1,
      "cancelled": 0
    }
  }
}
```

**Current Frontend Workaround:**

See implementation in frontend repository: `src/api/today.ts` (get method)

The frontend retrieves the current plan and extracts today's data, calculating statistics from the tasks.

### 3. GET /reviews/current

**Why It's Important:**
Called when user opens the Review page.

**Expected Response:**
```json
{
  "success": true,
  "data": {
    "weeklyPlanId": "plan_123",
    "weekStartDate": "2024-01-01",
    "weekEndDate": "2024-01-07",
    "statistics": {
      "totalPlanned": 20,
      "completed": 15,
      "cancelled": 2,
      "postponed": 3,
      "addedAfterConfirm": 5,
      "completionRate": 75.0,
      "totalChanges": 12,
      "changesByType": {
        "STATUS_CHANGED": 15,
        "TIME_CHANGED": 3,
        "MOVED_TO_ANOTHER_DAY": 3
      }
    },
    "dailyBreakdown": {
      "2024-01-01": {
        "date": "2024-01-01",
        "dayOfWeek": "MONDAY",
        "planned": 5,
        "completed": 4,
        "completionRate": 80.0,
        "changesCount": 3
      }
    },
    "changeHistory": [...]
  }
}
```

**Current Frontend Workaround:**

See implementation in frontend repository: `src/api/reviews.ts` (getCurrent method)

The frontend retrieves the current plan ID first, then fetches the review for that specific plan.

---

## API Contract Changes from Frontend Integration

### Authentication Response Format

**CRITICAL CHANGE:** Token field name

**Old (Incorrect):**
```json
{
  "success": true,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIs...",
    "user": {...}
  }
}
```

**New (Required):**
```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIs...",
    "tokenType": "Bearer",
    "expiresIn": 86400,
    "user": {
      "id": "user_123",
      "email": "user@example.com",
      "name": "John Doe"
    }
  }
}
```

**Frontend Implementation:**

See `src/api/auth.ts` in frontend repository for LoginResponse interface definition.

Key requirement: Field must be named `accessToken`, not `token`.

### Task Endpoints Require planId

**CRITICAL CHANGE:** All task operations need planId in path

**Format:**
- `GET /plans/{planId}/tasks` - NOT `/tasks`
- `POST /plans/{planId}/tasks` - NOT `/tasks`
- `PUT /plans/{planId}/tasks/{taskId}` - NOT `/tasks/{taskId}`
- `DELETE /plans/{planId}/tasks/{taskId}` - NOT `/tasks/{taskId}`

**Reason:** Tasks belong to a specific weekly plan. Plan context is required for change tracking and business logic.

### Task Creation Date Parameter

**CRITICAL CHANGE:** Date must be query parameter, not in request body

**Incorrect:**
```javascript
POST /plans/{planId}/tasks
{
  "date": "2024-01-01",
  "title": "Task title",
  ...
}
```

**Correct:**
```javascript
POST /plans/{planId}/tasks?date=2024-01-01
{
  "title": "Task title",
  "description": "...",
  ...
}
```

**Frontend Implementation:**

See `src/api/tasks.ts` in frontend repository for task creation method.

The date is passed as a query parameter, not in the request body.

### Notification Read Operations

**CRITICAL CHANGE:** Use POST method, not PUT

**Incorrect:**
```javascript
PUT /notifications/{notificationId}/read
PUT /notifications/read-all
```

**Correct:**
```javascript
POST /notifications/{notificationId}/read
POST /notifications/read-all
```

**Reason:** These operations don't update a resource field - they trigger an action. POST is more semantically correct.

**Frontend Implementation:**

See `src/api/notifications.ts` in frontend repository.

Uses POST method (not PUT) for marking notifications as read.

### Plan Confirmation Method

**CRITICAL CHANGE:** Use POST for confirmation, not PUT

**Incorrect:**
```javascript
PUT /plans/{planId}/confirm
```

**Correct:**
```javascript
POST /plans/{planId}/confirm
```

**Reason:** Confirmation is a state transition action, not a field update.

**Frontend Implementation:**

See `src/api/plans.ts` in frontend repository.

Uses POST method (not PUT) for plan confirmation.

---

## Response Format Requirements

### Standard Response Wrapper

All responses MUST use this format:

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

**Frontend Axios Interceptor:**

See `src/api/client.ts` in frontend repository for response interceptor implementation.

The interceptor auto-unwraps responses and handles 401 errors with automatic logout.

### Paginated Responses

**Format:**
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

**Used in:**
- `GET /plans` - Weekly plans list
- `GET /notifications` - Notifications list
- `GET /plans/{planId}/changes` - Change logs

### Task Response After Operations

All task operations (create, update, delete, move) should return the updated task object:

```json
{
  "success": true,
  "data": {
    "id": "task_123",
    "title": "Updated task",
    "status": "COMPLETED",
    "completedAt": "2024-01-01T15:30:00Z",
    ...
  }
}
```

This allows frontend to update UI without refetching.

---

## Authentication Implementation

### Required Endpoints

#### POST /auth/register
```json
Request:
{
  "email": "user@example.com",
  "password": "password123",
  "name": "John Doe"
}

Response (201):
{
  "success": true,
  "data": {
    "id": "user_123",
    "email": "user@example.com",
    "name": "John Doe",
    "createdAt": "2024-01-01T00:00:00Z"
  }
}
```

#### POST /auth/login
```json
Request:
{
  "email": "user@example.com",
  "password": "password123"
}

Response (200):
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIs...",
    "tokenType": "Bearer",
    "expiresIn": 86400,
    "user": {
      "id": "user_123",
      "email": "user@example.com",
      "name": "John Doe"
    }
  }
}
```

#### GET /auth/me

Requires: `Authorization: Bearer {token}`

```json
Response (200):
{
  "success": true,
  "data": {
    "id": "user_123",
    "email": "user@example.com",
    "name": "John Doe",
    "settings": {
      "planningDay": "SUNDAY",
      "reviewDay": "SATURDAY",
      "weekStartDay": "MONDAY",
      "timezone": "Asia/Seoul",
      "defaultReminderMinutes": 10,
      "notificationEnabled": true
    }
  }
}
```

### JWT Token Requirements

1. **Token Type:** JWT (JSON Web Token)
2. **Algorithm:** HS256 or RS256
3. **Expiration:** Recommended 24 hours (86400 seconds)
4. **Claims Required:**
   - `sub` - User ID
   - `email` - User email
   - `exp` - Expiration timestamp
   - `iat` - Issued at timestamp

### Authorization Header Format

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

Frontend adds this automatically to all requests via axios interceptor.

---

## Task Management Implementation

### Task Status Flow

```
PENDING → IN_PROGRESS → COMPLETED
   ↓           ↓            ↓
POSTPONED  CANCELLED    (final)
```

**Status Enum:**
```java
enum TaskStatus {
    PENDING,      // Initial state
    IN_PROGRESS,  // User started working
    COMPLETED,    // Finished successfully
    CANCELLED,    // User cancelled
    POSTPONED     // Moved to another day
}
```

### Task Priority

```java
enum Priority {
    LOW,
    MEDIUM,
    HIGH,
    URGENT
}
```

### Task Creation Flow

1. Frontend: User selects date and fills form
2. Frontend: `POST /plans/{planId}/tasks?date=2024-01-01`
3. Backend: Validate plan exists and user owns it
4. Backend: Create task with status=PENDING
5. Backend: If plan is CONFIRMED, create ChangeLog (TASK_CREATED)
6. Backend: Return created task

**Example Request Format:**

See [API Contract](./api-contract.md#3-tasks) for detailed request/response examples.

Key points:
- Date passed as query parameter
- Reminder settings and priority are optional
- Returns created task with generated ID

### Task Update with Change Tracking

When plan is CONFIRMED, track changes:

**Change Tracking Logic:**

See implementation: `src/changelog/changelog.service.ts:13`

When a task is updated in CONFIRMED plan:
1. Compare old and new values
2. Create ChangeLog entry with detected changes
3. Store change reason if provided
4. Update task and return

See [API Contract](./api-contract.md#3-tasks) for request/response format.

### Task Move Operation

**Task Move Logic:**

See implementation: `src/plan/plan.service.ts:265`

Move operation process:
1. Update task date to target date
2. Set original task status to POSTPONED
3. Create ChangeLog entry
4. Return updated task

See [API Contract](./api-contract.md#put-taskstaskidmove) for request/response format.

---

## Testing Your API

### Using test-api.js Script

A Node.js script is provided to test backend endpoints:

```bash
# From project root
node test-api.js
```

**What it tests:**
1. Login and token retrieval
2. /auth/me endpoint
3. /plans/current (or fallback)
4. /today endpoint (or fallback)
5. /reviews/current (or fallback)
6. Task creation
7. Task status update
8. Notifications

**Expected Output:**
```
=== API Endpoint Test ===

1. Testing Login...
✅ Login successful
   Token: eyJhbGciOiJIUzI1NiIs...

2. Testing /auth/me...
✅ /auth/me successful
   User: { id: 'user_123', email: 'test@example.com', ... }

3. Testing /plans/current...
✅ /plans/current successful
   Plan ID: plan_123
   Status: DRAFT
   Week: 2024-01-01 - 2024-01-07
...
```

**If endpoint is missing:**
```
3. Testing /plans/current...
⚠️ /plans/current not found (404)
   Trying fallback: GET /plans
✅ Fallback successful
```

### Using test-api-endpoints.html

A browser-based testing tool is provided:

```bash
# Open in browser
open test-api-endpoints.html
# or
start test-api-endpoints.html
```

**Features:**
1. Visual interface for testing endpoints
2. Shows request/response in real-time
3. Tests all critical endpoints
4. Highlights missing endpoints
5. Shows CORS errors if present

**Setup:**
1. Start your backend server
2. Open test-api-endpoints.html in browser
3. Update API_BASE if needed (default: http://localhost:8080/api/v1)
4. Click "Run Tests"

### Manual Testing with cURL

**Login:**
```bash
curl -X POST http://localhost:8080/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"password123"}'
```

**Get Current Plan:**
```bash
curl http://localhost:8080/api/v1/plans/current \
  -H "Authorization: Bearer YOUR_TOKEN_HERE"
```

**Create Task:**
```bash
curl -X POST "http://localhost:8080/api/v1/plans/PLAN_ID/tasks?date=2024-01-01" \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -H "Content-Type: application/json" \
  -d '{"title":"Test task","priority":"MEDIUM"}'
```

---

## Common Integration Issues

### Issue 1: Token Field Name Mismatch

**Symptom:** Frontend shows "Not authenticated" after successful login

**Cause:** Backend returns `token` instead of `accessToken`

**Fix:**
```java
// LoginResponse.java
public class LoginResponse {
    private String accessToken;  // NOT "token"
    private String tokenType;
    private int expiresIn;
    private UserDto user;
}
```

### Issue 2: CORS Errors

**Symptom:** Browser console shows CORS errors

**Fix:** Add CORS configuration
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/v1/**")
            .allowedOrigins("http://localhost:3000")
            .allowedMethods("GET", "POST", "PUT", "DELETE")
            .allowedHeaders("*")
            .allowCredentials(true);
    }
}
```

### Issue 3: 401 Unauthorized Loop

**Symptom:** Frontend keeps redirecting to login page

**Cause:** JWT token validation failing or expired

**Debug:**
1. Check token is being sent in header
2. Verify token signature and claims
3. Check token expiration
4. Ensure Authorization header format: `Bearer {token}`

**Frontend disables auto-logout for debugging:**
```typescript
// In src/api/client.ts - currently commented out for debugging
if (error.response?.status === 401) {
  // useAuthStore.getState().logout()
  // window.location.href = '/login'
}
```

### Issue 4: Task Creation Date Mismatch

**Symptom:** Tasks appear on wrong day

**Cause:** Backend reads date from body instead of query param

**Fix:**
```java
@PostMapping("/plans/{planId}/tasks")
public ResponseEntity<ApiResponse<Task>> createTask(
    @PathVariable String planId,
    @RequestParam LocalDate date,  // From query param
    @RequestBody CreateTaskRequest request
) {
    // Use date parameter, not request.getDate()
}
```

### Issue 5: Missing planId in Task Operations

**Symptom:** 404 errors when creating/updating tasks

**Cause:** Frontend sends planId in path, backend expects different route

**Fix:** Ensure routes match:
```java
@RestController
@RequestMapping("/api/v1/plans/{planId}/tasks")
public class TaskController {
    // All task operations here use planId from path
}
```

### Issue 6: Notification Read Method Mismatch

**Symptom:** 405 Method Not Allowed on notification read

**Cause:** Backend uses PUT, frontend uses POST

**Fix:**
```java
@PostMapping("/notifications/{id}/read")  // POST, not PUT
public ResponseEntity<Void> markAsRead(@PathVariable String id) {
    notificationService.markAsRead(id);
    return ResponseEntity.ok().build();
}
```

### Issue 7: Pagination Response Format

**Symptom:** Frontend shows "Cannot read property 'content' of undefined"

**Cause:** Backend returns array instead of paginated object

**Incorrect:**
```json
{
  "success": true,
  "data": [...]
}
```

**Correct:**
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

## Next Steps

After implementing the required endpoints:

1. **Run test-api.js** to verify basic functionality
2. **Test in browser** with test-api-endpoints.html
3. **Start frontend** with `npm run dev`
4. **Test user flows:**
   - Register new user
   - Login
   - Create weekly plan
   - Add tasks
   - Complete tasks
   - View review

5. **Check for errors** in browser console and network tab

---

## Questions or Issues?

If you encounter issues not covered here:

1. Check browser console for error messages
2. Check network tab for failed requests
3. Verify request/response format matches this guide
4. Use test scripts to isolate the problem
5. Check that all required fields are present in responses

**Frontend Repository:** weekly-planner-frontend
**Docs Repository:** weekly-planner-docs (this submodule)
**API Contract:** See `api-contract.md` for full specification

---

## Related Documentation

- [API Contract](./api-contract.md) - Complete REST API specification
- [Domain Model](./domain-model.md) - Entity definitions and schemas
- [Business Rules](./business-rules.md) - Business logic and validation rules
- [Development Workflow](./development-workflow.md) - Development best practices
- [README](./README.md) - Documentation index

---

**Last Updated:** 2025-12-29
