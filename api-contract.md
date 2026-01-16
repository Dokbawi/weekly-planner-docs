# API Contract

Base URL: `/api/v1`

> **Note**: Updated 2025-01-13
> - 실제 백엔드 구현과 동기화됨
> - Swagger UI: http://localhost:3000/api-docs 에서 실시간 확인 가능

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
| `AUTH004` | 401 | 인증 필요 |
| `AUTH004` | 403 | 권한 없음 |
| `NOT_FOUND` | 404 | 리소스 없음 |
| `GEN001` | 400 | 입력값 오류 |
| `GEN002` | 500 | 서버 오류 |

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
        "id": "507f1f77bcf86cd799439011",
        "email": "user@example.com",
        "name": "홍길동",
        "settings": {
            "planningDay": 0,
            "reviewDay": 6,
            "defaultReminderMinutes": 30,
            "timezone": "Asia/Seoul",
            "notificationEnabled": true
        },
        "createdAt": "2025-01-01T00:00:00.000Z"
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
        "accessToken": "eyJhbGciOiJIUzI1NiIs...",
        "tokenType": "Bearer",
        "expiresIn": 86400
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
        "id": "507f1f77bcf86cd799439011",
        "email": "user@example.com",
        "name": "홍길동",
        "settings": {
            "planningDay": 0,
            "reviewDay": 6,
            "defaultReminderMinutes": 30,
            "timezone": "Asia/Seoul",
            "notificationEnabled": true
        },
        "createdAt": "2025-01-01T00:00:00.000Z"
    }
}
```

### PUT /auth/settings
사용자 설정 수정

**Request**
```json
{
    "planningDay": 0,
    "reviewDay": 6,
    "timezone": "Asia/Seoul",
    "defaultReminderMinutes": 15,
    "notificationEnabled": true
}
```

**Response** `200 OK`
```json
{
    "success": true,
    "data": {
        "id": "507f1f77bcf86cd799439011",
        "email": "user@example.com",
        "name": "홍길동",
        "settings": {
            "planningDay": 0,
            "reviewDay": 6,
            "defaultReminderMinutes": 15,
            "timezone": "Asia/Seoul",
            "notificationEnabled": true
        },
        "createdAt": "2025-01-01T00:00:00.000Z"
    }
}
```

---

## 2. Weekly Plans

### POST /plans
주간 계획 생성

**Request**
```json
{
    "weekStartDate": "2025-01-12"
}
```

**Response** `201 Created`
```json
{
    "success": true,
    "data": {
        "id": "507f1f77bcf86cd799439011",
        "userId": "507f1f77bcf86cd799439012",
        "weekStartDate": "2025-01-12",
        "weekEndDate": "2025-01-18",
        "status": "DRAFT",
        "dailyPlans": [
            {
                "date": "2025-01-12",
                "tasks": []
            },
            // ... 7일치
        ],
        "confirmedAt": null,
        "createdAt": "2025-01-12T00:00:00.000Z",
        "updatedAt": "2025-01-12T00:00:00.000Z"
    }
}
```

**Error**
- `400`: `Weekly plan already exists for week starting {date}` - 해당 주 계획이 이미 존재

### GET /plans/current
현재 주 계획 조회. 없으면 자동 생성.

**Response** `200 OK`
- POST /plans 응답과 동일한 형태

### GET /plans/by-date
날짜로 주간 계획 조회

**Query Parameters**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `date` | string | Yes | 조회할 날짜 (yyyy-MM-dd) |

**Response** `200 OK`
- 해당 날짜를 포함하는 주간 계획 반환
- 없으면 `data: null`

### GET /plans/{planId}
특정 주간 계획 조회

**Response** `200 OK`
- POST /plans 응답과 동일한 형태

### GET /plans
주간 계획 목록 조회

**Query Parameters**
| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `page` | int | 0 | 페이지 번호 (0부터) |
| `size` | int | 10 | 페이지 크기 |
| `status` | string | - | 상태 필터 (DRAFT, CONFIRMED) |

**Response** `200 OK`
```json
{
    "success": true,
    "data": {
        "content": [ /* WeeklyPlan 배열 */ ],
        "page": 0,
        "size": 10,
        "totalElements": 25,
        "totalPages": 3
    }
}
```

### POST /plans/{planId}/confirm
계획 확정. 이후 변경사항 추적 시작.

**Response** `201 Created`
```json
{
    "success": true,
    "data": {
        "id": "507f1f77bcf86cd799439011",
        "status": "CONFIRMED",
        "confirmedAt": "2025-01-12T09:00:00.000Z",
        // ... 나머지 필드
    }
}
```

**Error**
- `400`: `Plan is already confirmed` - 이미 확정된 계획

### PUT /plans/{planId}/memo
일일 메모 수정

**Request**
```json
{
    "date": "2025-01-12",
    "memo": "오늘은 집중해서 일하기"
}
```

**Response** `200 OK`
- WeeklyPlan 전체 반환

---

## 3. Tasks

### POST /plans/{planId}/tasks
Task 추가

**Query Parameters**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `date` | string | Yes | Task를 추가할 날짜 (yyyy-MM-dd) |

**Request**
```json
{
    "title": "프로젝트 미팅",
    "description": "Q1 계획 논의",
    "scheduledTime": "14:00",
    "reminderMinutesBefore": 30,
    "priority": "HIGH",
    "tags": ["work", "meeting"]
}
```

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `title` | string | Yes | - | Task 제목 |
| `description` | string | No | - | 상세 설명 |
| `scheduledTime` | string | No | - | 예정 시간 (HH:mm) |
| `reminderMinutesBefore` | number | No | 30 | 리마인더 (몇 분 전) |
| `priority` | string | No | MEDIUM | 우선순위 (LOW, MEDIUM, HIGH) |
| `tags` | string[] | No | [] | 태그 목록 |

**Response** `201 Created`
```json
{
    "success": true,
    "data": {
        "id": "507f1f77bcf86cd799439011",
        "title": "프로젝트 미팅",
        "description": "Q1 계획 논의",
        "status": "PENDING",
        "priority": "HIGH",
        "scheduledTime": "14:00",
        "reminderMinutesBefore": 30,
        "tags": ["work", "meeting"],
        "createdAt": "2025-01-12T00:00:00.000Z",
        "completedAt": null
    }
}
```

**Error**
- `400`: `Date {date} is not in this weekly plan` - 주간 계획 범위 외 날짜

### PUT /plans/{planId}/tasks/{taskId}
Task 수정

**Request**
```json
{
    "title": "프로젝트 미팅 (변경)",
    "description": "업데이트된 설명",
    "status": "COMPLETED",
    "priority": "MEDIUM",
    "scheduledTime": "15:00",
    "reminderMinutesBefore": 15,
    "tags": ["updated", "tags"],
    "reason": "시간 변경됨"
}
```

모든 필드는 선택사항. `reason`은 CONFIRMED 상태일 때 ChangeLog에 기록됨.

**Task Status 값**
- `PENDING`: 대기 중
- `IN_PROGRESS`: 진행 중
- `COMPLETED`: 완료됨
- `CANCELLED`: 취소됨
- `POSTPONED`: 연기됨

**Response** `200 OK`
- Task 객체 반환

### DELETE /plans/{planId}/tasks/{taskId}
Task 삭제

**Query Parameters**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `reason` | string | No | 삭제 사유 (ChangeLog 기록용) |

**Response** `200 OK`
```json
{
    "success": true,
    "data": null
}
```

### POST /plans/{planId}/tasks/{taskId}/move
Task를 다른 날로 이동

**Request**
```json
{
    "targetDate": "2025-01-13",
    "reason": "오늘 시간 부족"
}
```

**동작 방식**
1. 원본 Task의 상태를 `POSTPONED`로 변경
2. 대상 날짜에 새 Task 생성 (`PENDING` 상태)
3. ChangeLog 자동 기록 (CONFIRMED 상태일 때)

**Response** `201 Created`
- 새로 생성된 Task 객체 반환

**Error**
- `400`: `Target date {date} is not in this weekly plan` - 주간 계획 범위 외 날짜

---

## 4. Change Logs

### GET /plans/{planId}/changes
변경 이력 조회

**Query Parameters**
| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `date` | string | - | 특정 날짜 필터 |
| `type` | string | - | 변경 유형 필터 |
| `page` | int | 0 | 페이지 번호 |
| `size` | int | 20 | 페이지 크기 |

**Change Types**
- `TASK_CREATED`: Task 추가됨
- `TASK_UPDATED`: Task 수정됨
- `TASK_DELETED`: Task 삭제됨
- `MOVED_TO_ANOTHER_DAY`: 다른 날로 이동됨

**Response** `200 OK`
```json
{
    "success": true,
    "data": {
        "content": [
            {
                "id": "507f1f77bcf86cd799439011",
                "weeklyPlanId": "507f1f77bcf86cd799439012",
                "userId": "507f1f77bcf86cd799439013",
                "targetDate": "2025-01-12",
                "taskId": "507f1f77bcf86cd799439014",
                "taskTitle": "프로젝트 미팅",
                "changeType": "TASK_UPDATED",
                "changes": [
                    {
                        "field": "scheduledTime",
                        "oldValue": "14:00",
                        "newValue": "15:00"
                    }
                ],
                "reason": "시간 변경됨",
                "createdAt": "2025-01-12T10:00:00.000Z"
            }
        ],
        "page": 0,
        "size": 20,
        "totalElements": 5,
        "totalPages": 1
    }
}
```

---

## 5. Weekly Review

### GET /reviews/{planId}
주간 회고 데이터 조회

**Response** `200 OK`
```json
{
    "success": true,
    "data": {
        "weeklyPlanId": "507f1f77bcf86cd799439011",
        "weekStartDate": "2025-01-12",
        "weekEndDate": "2025-01-18",
        "statistics": {
            "totalTasks": 20,
            "completedTasks": 15,
            "cancelledTasks": 2,
            "postponedTasks": 3,
            "completionRate": 75.0
        },
        "dailyBreakdown": [
            {
                "date": "2025-01-12",
                "totalTasks": 5,
                "completedTasks": 4,
                "completionRate": 80.0
            }
            // ... 나머지 요일
        ],
        "changeHistory": [
            // ChangeLog 목록
        ]
    }
}
```

---

## 6. Notifications

### GET /notifications
알림 목록 조회

**Query Parameters**
| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `unreadOnly` | boolean | false | 읽지 않은 것만 |
| `page` | int | 0 | 페이지 번호 |
| `size` | int | 20 | 페이지 크기 |

**Response** `200 OK`
```json
{
    "success": true,
    "data": {
        "content": [
            {
                "id": "507f1f77bcf86cd799439011",
                "userId": "507f1f77bcf86cd799439012",
                "type": "TASK_REMINDER",
                "title": "할 일 알림",
                "message": "프로젝트 미팅 - 14:00",
                "relatedTaskId": "507f1f77bcf86cd799439013",
                "relatedDate": "2025-01-12",
                "isRead": false,
                "createdAt": "2025-01-12T13:50:00.000Z"
            }
        ],
        "page": 0,
        "size": 20,
        "totalElements": 5,
        "totalPages": 1
    }
}
```

**Notification Types**
- `TASK_REMINDER`: Task 리마인더
- `DAILY_SUMMARY`: 일일 할 일 요약
- `PLANNING_REMINDER`: 계획 수립 알림
- `REVIEW_REMINDER`: 회고 알림

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

**Response** `200 OK`

### PUT /notifications/read-all
전체 알림 읽음 처리

**Response** `200 OK`

---

## 7. Today (편의 API)

### GET /today
오늘 할 일 조회 (현재 주 계획에서 오늘 날짜)

**Response** `200 OK`
```json
{
    "success": true,
    "data": {
        "date": "2025-01-12",
        "weeklyPlan": {
            // WeeklyPlan 객체 (해당 주 계획이 있는 경우)
        },
        "tasks": [
            {
                "id": "507f1f77bcf86cd799439011",
                "title": "프로젝트 미팅",
                "scheduledTime": "14:00",
                "status": "PENDING",
                "priority": "HIGH",
                // ... 나머지 Task 필드
            }
        ]
    }
}
```

주간 계획이 없으면 `weeklyPlan`은 없고 `tasks`는 빈 배열.

---

## Related Documentation

- [Domain Model](./domain-model.md) - Entity definitions and schemas
- [Business Rules](./business-rules.md) - Business logic and validation rules
- [Backend API Reference](../docs-internal/API_REFERENCE.md) - 백엔드 상세 API 문서

---

**Last Updated:** 2025-01-13
