# Domain Model

## 개요
주간 계획 → 일일 TODO → 변경 추적 → 회고 사이클을 지원하는 일정관리 서비스

---

## 1. User (사용자)

**Schema:** `src/user/schemas/user.schema.ts`

```
User {
    id: String (ObjectId)
    email: String (unique, required)
    password: String (BCrypt encoded, required)
    name: String (required)
    settings: UserSettings
    createdAt: Date
    updatedAt: Date
}

UserSettings {
    planningDay: DayOfWeek = 0 (SUNDAY)    // 계획 수립 요일 (0-6)
    reviewDay: DayOfWeek = 6 (SATURDAY)    // 회고 요일 (0-6)
    timezone: String = "Asia/Seoul"
    defaultReminderMinutes: Int = 30
    notificationEnabled: Boolean = true
}

DayOfWeek (숫자 enum) {
    SUNDAY = 0
    MONDAY = 1
    TUESDAY = 2
    WEDNESDAY = 3
    THURSDAY = 4
    FRIDAY = 5
    SATURDAY = 6
}
```

---

## 2. WeeklyPlan (주간 계획)

**Schema:** `src/plan/schemas/plan.schema.ts`

```
WeeklyPlan {
    id: String (ObjectId)
    userId: ObjectId (User.id 참조, required)
    weekStartDate: String (YYYY-MM-DD, required)
    weekEndDate: String (YYYY-MM-DD, required)
    status: PlanStatus = DRAFT
    dailyPlans: Array<DailyPlan>
    confirmedAt: Date?                // 확정 시각
    createdAt: Date
    updatedAt: Date
}

PlanStatus {
    DRAFT      // 작성 중 (변경 추적 X)
    CONFIRMED  // 확정됨 (변경 추적 O)
}
```

### 인덱스
- `userId + weekStartDate` (unique) - 사용자당 주별 하나의 계획만 허용

### 상태 전이
```
DRAFT → CONFIRMED (확정 버튼)
```

---

## 3. DailyPlan (일일 계획)

```
DailyPlan {
    date: String (YYYY-MM-DD, required)
    tasks: Array<Task>
    memo: String?
}
```

- WeeklyPlan에 embedded로 저장
- 별도 collection 아님

---

## 4. Task (할 일)

```
Task {
    id: String (ObjectId, auto-generated)
    title: String (required)
    description: String?
    scheduledTime: String?            // "HH:mm" 형식 (선택적 시간 지정)
    reminderMinutesBefore: Int = 30   // 리마인더 (몇 분 전)
    status: TaskStatus = PENDING
    priority: TaskPriority = MEDIUM
    tags: Array<String> = []
    createdAt: Date
    completedAt: Date?
}

TaskStatus {
    PENDING       // 대기
    IN_PROGRESS   // 진행 중
    COMPLETED     // 완료
    CANCELLED     // 취소
    POSTPONED     // 연기됨 (다른 날로 이동)
}

TaskPriority {
    LOW
    MEDIUM
    HIGH
}
```

### 상태 전이
```
PENDING → IN_PROGRESS → COMPLETED
PENDING → CANCELLED
PENDING → POSTPONED (다른 날로 이동 시)
IN_PROGRESS → CANCELLED
IN_PROGRESS → POSTPONED
```

---

## 5. ChangeLog (변경 이력)

**핵심 기능**: WeeklyPlan이 CONFIRMED 상태일 때만 기록

**Schema:** `src/changelog/schemas/changelog.schema.ts`

**Service:** `src/changelog/changelog.service.ts`

```
ChangeLog {
    id: String (ObjectId)
    weeklyPlanId: String (required)
    userId: String (required)
    targetDate: String (YYYY-MM-DD)   // 변경된 Task가 속한 날짜
    taskId: String (required)
    taskTitle: String                 // 조회 편의용 스냅샷
    changeType: ChangeType (required)
    changes: Array<FieldChange>
    reason: String?                   // 변경 사유 (선택)
    createdAt: Date
}

FieldChange {
    field: String                     // 변경된 필드명
    oldValue: Any?                    // 이전 값
    newValue: Any?                    // 새 값
}

ChangeType {
    TASK_CREATED            // 확정 후 Task 추가
    TASK_UPDATED            // Task 내용 수정
    TASK_DELETED            // Task 삭제
    MOVED_TO_ANOTHER_DAY    // 다른 날로 이동
}
```

### 기록 규칙
1. `WeeklyPlan.status == CONFIRMED` 일 때만 기록
2. 실제 값이 변경된 필드만 `changes`에 포함
3. `TASK_CREATED`: 새 Task 추가 시
4. `TASK_DELETED`: Task 삭제 시
5. `MOVED_TO_ANOTHER_DAY`: 이동 시 movedTo 정보 포함

---

## 6. Notification (알림)

**Schema:** `src/notification/schemas/notification.schema.ts`

**Scheduler:** `src/notification/notification.scheduler.ts`

```
Notification {
    id: String (ObjectId)
    userId: ObjectId (required)
    type: NotificationType (required)
    title: String (required)
    message: String (required)
    relatedTaskId: String?
    relatedDate: String?              // YYYY-MM-DD
    isRead: Boolean = false
    createdAt: Date
}

NotificationType {
    TASK_REMINDER       // Task 시간 알림
    PLANNING_REMINDER   // 계획 수립일 알림
    REVIEW_REMINDER     // 회고일 알림
    DAILY_SUMMARY       // 오늘 할 일 요약
}
```

---

## 7. WeeklyReview (주간 회고)

**주의**: DB에 저장하지 않고 실시간 생성 (ChangeLog 기반 집계)

**Service:** `src/review/review.service.ts`

```
WeeklyReview {
    weeklyPlanId: String
    weekStartDate: String
    weekEndDate: String
    statistics: ReviewStatistics
    dailyBreakdown: Array<DailyStatistics>
    changeHistory: Array<ChangeLog>
}

ReviewStatistics {
    totalTasks: Int                   // 전체 Task 수
    completedTasks: Int               // 완료된 Task
    cancelledTasks: Int               // 취소된 Task
    postponedTasks: Int               // 연기된 Task
    completionRate: Float             // 완료율 (%)
}

DailyStatistics {
    date: String
    totalTasks: Int
    completedTasks: Int
    completionRate: Float
}
```

---

## MongoDB Collections

| Collection | Document | Index |
|------------|----------|-------|
| `users` | User | `email` (unique) |
| `weekly_plans` | WeeklyPlan | `userId + weekStartDate` (unique) |
| `change_logs` | ChangeLog | `weeklyPlanId`, `userId` |
| `notifications` | Notification | `userId + isRead`, `userId + createdAt` |

**Schemas:** `src/*/schemas/` 디렉토리 참조

---

## 시간대 처리

- 모든 `Date`는 UTC로 저장
- `weekStartDate`, `weekEndDate`, `targetDate`는 YYYY-MM-DD 문자열 (사용자 timezone 기준)
- API 응답 시 Date는 ISO 8601 형식
- 프론트엔드에서 사용자 timezone으로 변환하여 표시

---

## Related Documentation

- [API Contract](./api-contract.md) - REST API specification using these models
- [Business Rules](./business-rules.md) - Business logic and validation rules
- [Backend Integration Guide](./backend-integration-guide.md) - Implementation details

---

**Last Updated:** 2025-01-13
