# Backend API Requests

프론트엔드에서 요청하는 백엔드 미구현 API 목록입니다.
이 문서는 프론트엔드 개발 중 필요한 API를 백엔드 팀에 요청하기 위해 작성되었습니다.

**Last Updated:** 2026-01-19

---

## 요청 상태

| 상태 | 의미 |
|------|------|
| `REQUESTED` | 구현 요청됨 |
| `IN_PROGRESS` | 백엔드 개발 중 |
| `COMPLETED` | 구현 완료 |
| `WONTFIX` | 구현하지 않기로 결정 |

---

## 1. 출퇴근 계산기 API (Commute Routines)

**상태:** `REQUESTED`
**요청일:** 2026-01-19
**우선순위:** HIGH

출퇴근 시간 계산 기능을 위한 API입니다.
현재 프론트엔드는 localStorage에 임시 저장 중입니다.

### 엔드포인트

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/commute-routines` | 루틴 목록 조회 |
| GET | `/commute-routines/{routineId}` | 특정 루틴 조회 |
| POST | `/commute-routines` | 루틴 생성 |
| PUT | `/commute-routines/{routineId}` | 루틴 수정 |
| DELETE | `/commute-routines/{routineId}` | 루틴 삭제 |
| POST | `/commute-routines/{routineId}/calculate` | 출발 시간 계산 |
| POST | `/commute-routines/{routineId}/add-to-tasks` | 루틴을 Task로 추가 |

### 스키마

```typescript
// CommuteRoutine
interface CommuteRoutine {
  id: string
  userId: string
  name: string
  destination: string
  steps: CommuteStep[]
  totalMinutes: number
  defaultArrivalTime?: string  // "HH:mm"
  createdAt: string
  updatedAt: string
}

interface CommuteStep {
  id: string
  label: string
  durationMinutes: number
  type: 'prepare' | 'walk' | 'bus' | 'subway' | 'taxi' | 'car' | 'bike' | 'other'
  order: number
}
```

### Request/Response 예시

#### POST /commute-routines
```json
// Request
{
  "name": "출근 루틴",
  "destination": "회사",
  "steps": [
    { "label": "준비", "durationMinutes": 30, "type": "prepare", "order": 0 },
    { "label": "도보", "durationMinutes": 10, "type": "walk", "order": 1 },
    { "label": "지하철", "durationMinutes": 25, "type": "subway", "order": 2 }
  ],
  "defaultArrivalTime": "09:00"
}

// Response 201 Created
{
  "success": true,
  "data": {
    "id": "routine-id",
    "userId": "user-id",
    "name": "출근 루틴",
    "destination": "회사",
    "steps": [...],
    "totalMinutes": 65,
    "defaultArrivalTime": "09:00",
    "createdAt": "2026-01-19T00:00:00.000Z",
    "updatedAt": "2026-01-19T00:00:00.000Z"
  }
}
```

#### POST /commute-routines/{routineId}/calculate
```json
// Request
{
  "arrivalTime": "09:00",
  "offsetMinutes": 10  // 여유 시간 (선택, 기본: 0)
}

// Response 200 OK
{
  "success": true,
  "data": {
    "routineId": "routine-id",
    "arrivalTime": "09:00",
    "offsetMinutes": 10,
    "departureTime": "07:45",
    "totalMinutes": 75,
    "schedule": [
      { "stepId": "step-1", "label": "준비", "type": "prepare", "startTime": "07:45", "endTime": "08:15", "durationMinutes": 30 },
      { "stepId": "step-2", "label": "도보", "type": "walk", "startTime": "08:15", "endTime": "08:25", "durationMinutes": 10 },
      { "stepId": "step-3", "label": "지하철", "type": "subway", "startTime": "08:25", "endTime": "08:50", "durationMinutes": 25 }
    ]
  }
}
```

---

## 2. Task 순서 변경 API

**상태:** `REQUESTED`
**요청일:** 2026-01-19
**우선순위:** MEDIUM

드래그 앤 드롭으로 Task 순서를 변경하는 기능입니다.

### 엔드포인트

| Method | Endpoint | Description |
|--------|----------|-------------|
| PUT | `/plans/{planId}/tasks/reorder` | Task 순서 변경 |

### Request/Response

```json
// Request
{
  "date": "2026-01-19",
  "taskIds": ["task-3", "task-1", "task-2"]  // 정렬된 순서
}

// Response 200 OK
{
  "success": true,
  "data": null
}
```

---

## 3. 메모 저장 API 확인 요청

**상태:** `REQUESTED`
**요청일:** 2026-01-19
**우선순위:** HIGH

`PUT /plans/{planId}/memo` API 호출 시 400 에러가 발생합니다.
API가 구현되었는지, Request 형식이 올바른지 확인이 필요합니다.

### 현재 프론트엔드 요청

```json
// PUT /plans/{planId}/memo
{
  "date": "2026-01-19",
  "memo": "오늘의 메모 내용"
}
```

### api-contract.md 기준 스펙

```json
// PUT /plans/{planId}/memo
{
  "date": "2025-01-12",
  "memo": "오늘은 집중해서 일하기"
}

// Response: WeeklyPlan 전체 반환
```

### 에러 상황
- URL: `PUT /api/v1/plans/{planId}/memo`
- Status: 400 Bad Request
- 브라우저 콘솔: `Failed to load resource: the server responded with a status of 400`

---

## 구현 완료된 API

다음 API들은 정상 동작 확인되었습니다:

- [x] Authentication (`/auth/*`)
- [x] Weekly Plans (`/plans`, `/plans/{planId}`, `/plans/{planId}/confirm`)
- [x] Tasks (`/plans/{planId}/tasks/*`)
- [x] Task Move (`/plans/{planId}/tasks/{taskId}/move`)
- [x] Change Logs (`/plans/{planId}/changes`)
- [x] Weekly Review (`/reviews/{planId}`)
- [x] Notifications (`/notifications/*`)

---

## 워크어라운드 적용 중인 API

다음 API들은 백엔드에 직접 구현되지 않아 프론트엔드에서 우회 처리 중입니다.
구현되면 성능이 개선됩니다.

| Endpoint | 현재 워크어라운드 | 우선순위 |
|----------|------------------|----------|
| `GET /plans/current` | `/plans` 목록 조회 후 현재 주 필터링 | LOW |
| `GET /today` | `/plans` 목록 조회 후 오늘 날짜 필터링 | LOW |
| `GET /reviews/current` | `/reviews` 목록 조회 후 현재 주 필터링 | LOW |

---

## 변경 이력

| 날짜 | 변경 내용 |
|------|----------|
| 2026-01-19 | 최초 작성: 출퇴근 API, Task 순서, 메모 API |
