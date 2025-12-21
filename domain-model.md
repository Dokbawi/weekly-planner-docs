# Domain Model

## 개요
주간 계획 → 일일 TODO → 변경 추적 → 회고 사이클을 지원하는 일정관리 서비스

---

## 1. User (사용자)

```
User {
    id: String (ObjectId)
    email: String (unique)
    password: String (BCrypt encoded)
    name: String
    settings: UserSettings
    createdAt: Instant
    updatedAt: Instant
}

UserSettings {
    planningDay: DayOfWeek = SUNDAY      // 계획 수립 요일
    reviewDay: DayOfWeek = SATURDAY      // 회고 요일
    weekStartDay: DayOfWeek = MONDAY     // 주 시작 요일
    timezone: String = "Asia/Seoul"
    defaultReminderMinutes: Int = 10
    notificationEnabled: Boolean = true
}
```

---

## 2. WeeklyPlan (주간 계획)

```
WeeklyPlan {
    id: String (ObjectId)
    userId: String (User.id 참조)
    weekStartDate: LocalDate             // 주 시작일
    weekEndDate: LocalDate               // 주 종료일
    status: PlanStatus = DRAFT
    dailyPlans: Map<LocalDate, DailyPlan>
    confirmedAt: Instant?                // 확정 시각
    createdAt: Instant
    updatedAt: Instant
}

PlanStatus {
    DRAFT      // 작성 중 (변경 추적 X)
    CONFIRMED  // 확정됨 (변경 추적 O)
    COMPLETED  // 주간 완료
}
```

### 상태 전이
```
DRAFT → CONFIRMED (확정 버튼)
CONFIRMED → COMPLETED (주 종료 시 자동)
```

---

## 3. DailyPlan (일일 계획)

```
DailyPlan {
    date: LocalDate
    dayOfWeek: DayOfWeek
    tasks: List<Task>
    memo: String?
}
```

- WeeklyPlan에 embedded로 저장
- 별도 collection 아님

---

## 4. Task (할 일)

```
Task {
    id: String (ObjectId)
    title: String (required, max 200)
    description: String? (max 1000)
    scheduledTime: LocalTime?            // 선택적 시간 지정
    estimatedMinutes: Int?               // 예상 소요 시간
    reminder: ReminderSettings?
    status: TaskStatus = PENDING
    priority: Priority = MEDIUM
    tags: List<String>
    order: Int = 0                       // 정렬 순서
    createdAt: Instant
    completedAt: Instant?
}

ReminderSettings {
    enabled: Boolean = true
    minutesBefore: Int = 10
    notifiedAt: Instant?                 // 알림 발송 완료 시각
}

TaskStatus {
    PENDING       // 대기
    IN_PROGRESS   // 진행 중
    COMPLETED     // 완료
    CANCELLED     // 취소
    POSTPONED     // 연기됨 (다른 날로 이동)
}

Priority {
    LOW
    MEDIUM
    HIGH
    URGENT
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

## 5. ChangeLog (변경 이력) ⭐

**핵심 기능**: WeeklyPlan이 CONFIRMED 상태일 때만 기록

```
ChangeLog {
    id: String (ObjectId)
    weeklyPlanId: String
    userId: String
    targetDate: LocalDate                // 변경된 Task가 속한 날짜
    taskId: String
    taskTitle: String                    // 조회 편의용 스냅샷
    changeType: ChangeType
    changes: List<FieldChange>
    reason: String?                      // 변경 사유 (선택)
    changedAt: Instant
}

FieldChange {
    field: String                        // 변경된 필드명
    previousValue: String?               // 이전 값 (JSON string)
    newValue: String?                    // 새 값 (JSON string)
}

ChangeType {
    TASK_CREATED            // 확정 후 Task 추가
    TASK_UPDATED            // Task 내용 수정
    TASK_DELETED            // Task 삭제
    STATUS_CHANGED          // 상태 변경
    TIME_CHANGED            // 시간 변경
    MOVED_TO_ANOTHER_DAY    // 다른 날로 이동
    PRIORITY_CHANGED        // 우선순위 변경
}
```

### 기록 규칙
1. `WeeklyPlan.status == CONFIRMED` 일 때만 기록
2. 실제 값이 변경된 필드만 `FieldChange`에 포함
3. `TASK_CREATED`: previousValue = null
4. `TASK_DELETED`: newValue = null
5. `MOVED_TO_ANOTHER_DAY`: 이동 전 날짜의 로그로 기록, previousValue에 원래 날짜

---

## 6. Notification (알림)

```
Notification {
    id: String (ObjectId)
    userId: String
    type: NotificationType
    title: String
    message: String
    relatedTaskId: String?
    relatedDate: LocalDate?
    isRead: Boolean = false
    createdAt: Instant
    readAt: Instant?
}

NotificationType {
    TASK_REMINDER       // Task 시간 알림
    PLANNING_REMINDER   // 계획 수립일 알림 (일요일)
    REVIEW_REMINDER     // 회고일 알림 (토요일)
    DAILY_SUMMARY       // 오늘 할 일 요약
}
```

---

## 7. WeeklyReview (주간 회고)

**주의**: DB에 저장하지 않고 실시간 생성 (ChangeLog 기반 집계)

```
WeeklyReview {
    weeklyPlanId: String
    weekStartDate: LocalDate
    weekEndDate: LocalDate
    statistics: ReviewStatistics
    dailyBreakdown: Map<LocalDate, DailyStatistics>
    changeHistory: List<ChangeLog>
    generatedAt: Instant
}

ReviewStatistics {
    totalPlanned: Int                    // 전체 계획 Task 수
    completed: Int                       // 완료
    cancelled: Int                       // 취소
    postponed: Int                       // 연기
    addedAfterConfirm: Int              // 확정 후 추가된 Task
    completionRate: Double              // 완료율 (%)
    totalChanges: Int                   // 총 변경 횟수
    changesByType: Map<ChangeType, Int> // 유형별 변경 횟수
}

DailyStatistics {
    date: LocalDate
    dayOfWeek: DayOfWeek
    planned: Int
    completed: Int
    completionRate: Double
    changesCount: Int
}
```

---

## MongoDB Collections

| Collection | Document | Index |
|------------|----------|-------|
| `users` | User | `email` (unique) |
| `weekly_plans` | WeeklyPlan | `userId + weekStartDate` (unique), `userId + status` |
| `change_logs` | ChangeLog | `weeklyPlanId + changedAt`, `userId + targetDate` |
| `notifications` | Notification | `userId + isRead + createdAt` |

---

## 시간대 처리

- 모든 `Instant`는 UTC로 저장
- `LocalDate`, `LocalTime`은 사용자 timezone 기준
- API 응답 시 사용자 timezone으로 변환
- 프론트엔드에서 표시 시 로컬 시간으로 변환
