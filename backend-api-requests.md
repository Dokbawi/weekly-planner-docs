# Backend API Requests

프론트엔드에서 요청하는 백엔드 미구현 API 목록입니다.
이 문서는 프론트엔드 개발 중 필요한 API를 백엔드 팀에 요청하기 위해 작성되었습니다.

**Last Updated:** 2026-01-20

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

**상태:** `COMPLETED`
**요청일:** 2026-01-19
**완료일:** 2026-01-20
**우선순위:** HIGH

출퇴근 시간 계산 기능을 위한 API입니다.

### 엔드포인트

| Method | Endpoint | Description | 상태 |
|--------|----------|-------------|------|
| GET | `/commute-routines` | 루틴 목록 조회 | COMPLETED |
| GET | `/commute-routines/{routineId}` | 특정 루틴 조회 | COMPLETED |
| POST | `/commute-routines` | 루틴 생성 | COMPLETED |
| PUT | `/commute-routines/{routineId}` | 루틴 수정 | COMPLETED |
| DELETE | `/commute-routines/{routineId}` | 루틴 삭제 | COMPLETED |
| POST | `/commute-routines/{routineId}/calculate` | 출발 시간 계산 | COMPLETED |
| POST | `/commute-routines/{routineId}/add-to-tasks` | 루틴을 Task로 추가 | 미구현 (필요시 요청) |

### 구현 파일

- Controller: `src/commute-routine/commute-routine.controller.ts`
- Service: `src/commute-routine/commute-routine.service.ts`
- Schema: `src/commute-routine/schemas/commute-routine.schema.ts`
- DTO: `src/commute-routine/dto/commute-routine.dto.ts`

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

**상태:** `COMPLETED`
**요청일:** 2026-01-19
**완료일:** 2026-01-20
**우선순위:** MEDIUM

드래그 앤 드롭으로 Task 순서를 변경하는 기능입니다.

### 엔드포인트

| Method | Endpoint | Description | 상태 |
|--------|----------|-------------|------|
| PUT | `/plans/{planId}/tasks/reorder` | Task 순서 변경 | COMPLETED |

### 구현 파일

- Controller: `src/plan/plan.controller.ts` (reorderTasks 메서드)
- Service: `src/plan/plan.service.ts` (reorderTasks 메서드)
- DTO: `src/plan/dto/plan.dto.ts` (ReorderTasksDto)

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

## 3. 메모 저장 API

**상태:** `COMPLETED`
**요청일:** 2026-01-19
**완료일:** 2026-01-20
**우선순위:** HIGH

### 원인 분석

기존 컨트롤러에서 인라인 타입 `{ date: string; memo: string }`을 사용하여
class-validator의 ValidationPipe가 제대로 동작하지 않았습니다.

### 해결 방법

`UpdateMemoDto` 클래스를 생성하고 적절한 validation decorator를 추가했습니다.

### 구현 파일

- Controller: `src/plan/plan.controller.ts` (updateDailyMemo 메서드)
- Service: `src/plan/plan.service.ts` (updateDailyMemo 메서드)
- DTO: `src/plan/dto/plan.dto.ts` (UpdateMemoDto)

### Request/Response

```json
// PUT /plans/{planId}/memo
// Request
{
  "date": "2026-01-19",
  "memo": "오늘의 메모 내용"
}

// Response 200 OK
{
  "success": true,
  "data": { /* WeeklyPlan 전체 */ }
}
```

---

## 구현 완료된 API

다음 API들은 정상 동작 확인되었습니다:

- [x] Authentication (`/auth/*`)
- [x] Weekly Plans (`/plans`, `/plans/{planId}`, `/plans/{planId}/confirm`)
- [x] Tasks (`/plans/{planId}/tasks/*`)
- [x] Task Move (`/plans/{planId}/tasks/{taskId}/move`)
- [x] Task Reorder (`/plans/{planId}/tasks/reorder`) - NEW
- [x] Daily Memo (`/plans/{planId}/memo`) - FIXED
- [x] Change Logs (`/plans/{planId}/changes`)
- [x] Weekly Review (`/reviews/{planId}`)
- [x] Notifications (`/notifications/*`)
- [x] Commute Routines (`/commute-routines/*`) - NEW

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
| 2026-01-20 | 출퇴근 API, Task 순서, 메모 API 구현 완료 |
| 2026-01-19 | 최초 작성: 출퇴근 API, Task 순서, 메모 API 요청 |
