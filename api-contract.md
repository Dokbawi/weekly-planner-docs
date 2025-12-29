# API Contract

Base URL: `/api/v1`

> **Note**: Updated 2024-12-29
> - 일부 엔드포인트는 백엔드 구현 상태에 따라 존재하지 않을 수 있음
> - `/plans/current`, `/today`, `/reviews/current`는 선택적 구현
> - 실제 구현된 API는 백엔드 문서를 참조

## 공통 사항

### 인증
- Bearer Token (JWT)
- Header: `Authorization: Bearer {token}`
- 인증 불필요 API: `/auth/register`, `/auth/login`

### 응답 형식
```json
// 성공
{
    "success": true,
    "data": { ... }
}

// 실패
{
    "success": false,
    "error": {
        "code": "ERROR_CODE",
        "message": "Human readable message"
    }
}
```

### 공통 에러 코드
| Code | HTTP Status | Description |
|------|-------------|-------------|
| `UNAUTHORIZED` | 401 | 인증 필요 |
| `FORBIDDEN` | 403 | 권한 없음 |
| `NOT_FOUND` | 404 | 리소스 없음 |
| `VALIDATION_ERROR` | 400 | 입력값 오류 |
| `INTERNAL_ERROR` | 500 | 서버 오류 |

---

## 1. Authentication

### POST /auth/register
회원가입

**Request**
```json
{
    "email": "user@example.com",
    "password": "password123",
    "name": "홍길동"
}
```

**Response** `201 Created`
```json
{
    "success": true,
    "data": {
        "id": "user_123",
        "email": "user@example.com",
        "name": "홍길동",
        "createdAt": "2024-01-01T00:00:00Z"
    }
}
```

### POST /auth/login
로그인

**Request**
```json
{
    "email": "user@example.com",
    "password": "password123"
}
```

**Response** `200 OK`
```json
{
    "success": true,
    "data": {
        "token": "eyJhbGciOiJIUzI1NiIs...",
        "expiresAt": "2024-01-02T00:00:00Z",
        "user": {
            "id": "user_123",
            "email": "user@example.com",
            "name": "홍길동"
        }
    }
}
```

### GET /auth/me
현재 사용자 정보

**Response** `200 OK`
```json
{
    "success": true,
    "data": {
        "id": "user_123",
        "email": "user@example.com",
        "name": "홍길동",
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

### PUT /auth/settings
사용자 설정 수정

**Request**
```json
{
    "planningDay": "SUNDAY",
    "reviewDay": "SATURDAY",
    "timezone": "Asia/Seoul",
    "defaultReminderMinutes": 15,
    "notificationEnabled": true
}
```

---

## 2. Weekly Plans

### GET /plans/current
현재 주 계획 조회. 없으면 자동 생성.

**Response** `200 OK`
```json
{
    "success": true,
    "data": {
        "id": "plan_123",
        "weekStartDate": "2024-01-01",
        "weekEndDate": "2024-01-07",
        "status": "DRAFT",
        "dailyPlans": {
            "2024-01-01": {
                "date": "2024-01-01",
                "dayOfWeek": "MONDAY",
                "tasks": [],
                "memo": null
            },
            // ... 나머지 요일
        },
        "confirmedAt": null,
        "createdAt": "2024-01-01T00:00:00Z"
    }
}
```

### GET /plans/{planId}
특정 주간 계획 조회

### GET /plans
주간 계획 목록 조회

**Query Parameters**
| Param | Type | Description |
|-------|------|-------------|
| `page` | int | 페이지 번호 (0부터) |
| `size` | int | 페이지 크기 (default: 10) |
| `status` | string | 상태 필터 (DRAFT, CONFIRMED, COMPLETED) |

**Response** `200 OK`
```json
{
    "success": true,
    "data": {
        "content": [ ... ],
        "page": 0,
        "size": 10,
        "totalElements": 25,
        "totalPages": 3
    }
}
```

### PUT /plans/{planId}/confirm
계획 확정. 이후 변경사항 추적 시작.

**Response** `200 OK`
```json
{
    "success": true,
    "data": {
        "id": "plan_123",
        "status": "CONFIRMED",
        "confirmedAt": "2024-01-01T09:00:00Z",
        // ...
    }
}
```

**Error**
- `ALREADY_CONFIRMED`: 이미 확정된 계획

### PUT /plans/{planId}/memo
일일 메모 수정

**Request**
```json
{
    "date": "2024-01-01",
    "memo": "오늘은 집중해서 일하기"
}
```

---

## 3. Tasks

### GET /plans/{planId}/tasks
주간 계획의 전체 Task 조회

**Query Parameters**
| Param | Type | Description |
|-------|------|-------------|
| `date` | LocalDate | 특정 날짜 필터 |
| `status` | string | 상태 필터 |

### POST /plans/{planId}/tasks
Task 추가

**Request**
```json
{
    "date": "2024-01-01",
    "title": "프로젝트 미팅",
    "description": "Q1 계획 논의",
    "scheduledTime": "14:00",
    "estimatedMinutes": 60,
    "reminder": {
        "enabled": true,
        "minutesBefore": 10
    },
    "priority": "HIGH",
    "tags": ["work", "meeting"]
}
```

**Response** `201 Created`
```json
{
    "success": true,
    "data": {
        "id": "task_456",
        "title": "프로젝트 미팅",
        "status": "PENDING",
        "createdAt": "2024-01-01T00:00:00Z",
        // ...
    }
}
```

### PUT /tasks/{taskId}
Task 수정

**Request**
```json
{
    "title": "프로젝트 미팅 (변경)",
    "scheduledTime": "15:00",
    "reason": "시간 변경됨"
}
```

**Note**: `reason`은 선택. CONFIRMED 상태일 때 ChangeLog에 기록됨.

### PUT /tasks/{taskId}/status
Task 상태 변경

**Request**
```json
{
    "status": "COMPLETED",
    "reason": null
}
```

**Response** `200 OK`
```json
{
    "success": true,
    "data": {
        "id": "task_456",
        "status": "COMPLETED",
        "completedAt": "2024-01-01T15:30:00Z"
    }
}
```

### PUT /tasks/{taskId}/move
Task를 다른 날로 이동

**Request**
```json
{
    "targetDate": "2024-01-02",
    "reason": "오늘 시간 부족"
}
```

**Response** `200 OK`
```json
{
    "success": true,
    "data": {
        "id": "task_456",
        "status": "POSTPONED",
        "movedTo": "2024-01-02"
    }
}
```

### DELETE /tasks/{taskId}
Task 삭제

**Query Parameters**
| Param | Type | Description |
|-------|------|-------------|
| `reason` | string | 삭제 사유 (선택) |

---

## 4. Change Logs

### GET /plans/{planId}/changes
변경 이력 조회

**Query Parameters**
| Param | Type | Description |
|-------|------|-------------|
| `date` | LocalDate | 특정 날짜 필터 |
| `type` | string | 변경 유형 필터 |
| `page` | int | 페이지 번호 |
| `size` | int | 페이지 크기 |

**Response** `200 OK`
```json
{
    "success": true,
    "data": {
        "content": [
            {
                "id": "log_789",
                "targetDate": "2024-01-01",
                "taskId": "task_456",
                "taskTitle": "프로젝트 미팅",
                "changeType": "TIME_CHANGED",
                "changes": [
                    {
                        "field": "scheduledTime",
                        "previousValue": "14:00",
                        "newValue": "15:00"
                    }
                ],
                "reason": "시간 변경됨",
                "changedAt": "2024-01-01T10:00:00Z"
            }
        ],
        "page": 0,
        "size": 20,
        "totalElements": 5
    }
}
```

---

## 5. Weekly Review

### GET /reviews/current
현재 주 회고 데이터

**Response** `200 OK`
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
                "MOVED_TO_ANOTHER_DAY": 3,
                "TASK_CREATED": 5,
                "TASK_DELETED": 1
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
            },
            // ... 나머지 요일
        },
        "changeHistory": [
            // ChangeLog 목록 (최신순)
        ],
        "generatedAt": "2024-01-06T18:00:00Z"
    }
}
```

### GET /reviews/{weekStartDate}
특정 주 회고 조회

**Path Parameter**: `weekStartDate` (yyyy-MM-dd)

---

## 6. Notifications

### GET /notifications
알림 목록 조회

**Query Parameters**
| Param | Type | Description |
|-------|------|-------------|
| `unreadOnly` | boolean | 읽지 않은 것만 (default: false) |
| `page` | int | 페이지 번호 |
| `size` | int | 페이지 크기 |

**Response** `200 OK`
```json
{
    "success": true,
    "data": {
        "content": [
            {
                "id": "notif_123",
                "type": "TASK_REMINDER",
                "title": "⏰ 할 일 알림",
                "message": "프로젝트 미팅 - 14:00",
                "relatedTaskId": "task_456",
                "relatedDate": "2024-01-01",
                "isRead": false,
                "createdAt": "2024-01-01T13:50:00Z"
            }
        ],
        "page": 0,
        "size": 20,
        "totalElements": 5
    }
}
```

### GET /notifications/unread/count
읽지 않은 알림 수

**Response** `200 OK`
```json
{
    "success": true,
    "data": {
        "count": 3
    }
}
```

### PUT /notifications/{notificationId}/read
알림 읽음 처리

### PUT /notifications/read-all
전체 알림 읽음 처리

---

## 7. Today (편의 API)

### GET /today
오늘 할 일 조회 (현재 주 계획에서 오늘 날짜)

**Response** `200 OK`
```json
{
    "success": true,
    "data": {
        "date": "2024-01-01",
        "dayOfWeek": "MONDAY",
        "planId": "plan_123",
        "planStatus": "CONFIRMED",
        "tasks": [
            {
                "id": "task_456",
                "title": "프로젝트 미팅",
                "scheduledTime": "14:00",
                "status": "PENDING",
                "priority": "HIGH"
            }
        ],
        "memo": "오늘은 집중해서 일하기",
        "statistics": {
            "total": 5,
            "completed": 2,
            "remaining": 3
        }
    }
}
```
