# Business Rules

## 1. 주간 계획 (Weekly Plan)

### 1.1 주간 계획 생성
- 사용자당 특정 주에 하나의 WeeklyPlan만 존재
- `POST /plans` 호출 시 해당 주에 이미 계획이 있으면 `400 Bad Request` 반환
- `GET /plans/current` 호출 시 현재 주 계획이 없으면 자동 생성
- 주 시작일은 일요일 기준

### 1.2 계획 확정 (Confirm)
- DRAFT 상태에서만 확정 가능
- 확정 시점의 `confirmedAt` 기록
- **확정 전**: 변경사항 추적 안 함 (자유롭게 수정)
- **확정 후**: 모든 변경사항 ChangeLog에 기록
- 이미 CONFIRMED 상태면 `400 Bad Request` 반환

---

## 2. Task

### 2.1 Task 생성
- 최소 `title` 필수
- `scheduledTime` 없으면 시간 미지정 (하루 중 아무 때나)
- `priority` 기본값: MEDIUM
- `reminderMinutesBefore` 기본값: 30
- Task 생성 시 `date`는 query parameter로 전달

### 2.2 Task 상태 변경
| From | To | 조건 |
|------|-----|-----|
| PENDING | IN_PROGRESS | 항상 가능 |
| PENDING | COMPLETED | 항상 가능 |
| PENDING | CANCELLED | 항상 가능 |
| PENDING | POSTPONED | `move` API로만 |
| IN_PROGRESS | COMPLETED | 항상 가능 |
| IN_PROGRESS | CANCELLED | 항상 가능 |
| IN_PROGRESS | POSTPONED | `move` API로만 |
| COMPLETED | PENDING | 되돌리기 가능 |
| CANCELLED | PENDING | 되돌리기 가능 |
| POSTPONED | - | 상태 변경 불가, 새 날짜에서 PENDING으로 시작 |

### 2.3 Task 이동 (Move)
**Implementation:** `src/plan/plan.service.ts`

- 같은 WeeklyPlan 내 다른 날짜로만 이동 가능
- 이동 시 원본 Task의 상태는 POSTPONED로 변경
- 대상 날짜에 복사본 생성 (상태: PENDING, 새 ID 부여)
- ChangeLog에 `MOVED_TO_ANOTHER_DAY` 타입으로 기록

### 2.4 Task 삭제
- 물리적 삭제 (soft delete 아님)
- CONFIRMED 상태면 ChangeLog에 `TASK_DELETED` 기록
- 삭제된 Task는 ChangeLog의 `taskTitle` 스냅샷으로만 확인 가능

---

## 3. 변경 이력 (ChangeLog)

**Implementation:** `src/changelog/changelog.service.ts`

### 3.1 기록 조건
- WeeklyPlan.status == CONFIRMED 일 때만 기록
- DRAFT 상태에서는 기록 안 함

### 3.2 기록 항목
| 변경 유형 | 기록 시점 |
|----------|----------|
| TASK_CREATED | 확정 후 새 Task 추가 |
| TASK_UPDATED | title, description, status, priority, scheduledTime 등 변경 |
| TASK_DELETED | Task 삭제 |
| MOVED_TO_ANOTHER_DAY | 다른 날로 이동 |

### 3.3 FieldChange 기록
- 실제 값이 변경된 필드만 기록
- `oldValue`: 변경 전 값
- `newValue`: 변경 후 값

### 3.4 변경 사유 (reason)
- 선택 사항 (nullable)
- 사용자가 입력하지 않으면 null로 저장
- 회고 시 변경 맥락 파악용

---

## 4. 알림 (Notification)

**Implementation:** `src/notification/notification.scheduler.ts`

### 4.1 Task 알림 (TASK_REMINDER)
- `scheduledTime`이 설정된 Task 대상
- `scheduledTime - reminderMinutesBefore` 시점에 알림 생성

### 4.2 계획 수립 알림 (PLANNING_REMINDER)
- 사용자 설정 `planningDay` 아침에 발송
- "이번 주 계획을 세워보세요!" 메시지

### 4.3 회고 알림 (REVIEW_REMINDER)
- 사용자 설정 `reviewDay` 저녁에 발송
- "이번 주를 돌아보세요!" 메시지

### 4.4 일일 요약 (DAILY_SUMMARY)
- 매일 아침 (기본 08:00) 발송
- 오늘 할 일 개수 및 요약
- 오늘 할 일이 없으면 발송 안 함

---

## 5. 주간 회고 (Weekly Review)

**Implementation:** `src/review/review.service.ts`

**주의**: DB에 저장하지 않고 실시간 생성 (ChangeLog 기반 집계)

### 5.1 통계 계산
```
completionRate = (completedTasks / totalTasks) * 100

// 단, totalTasks가 0이면 completionRate = 0
```

### 5.2 집계 대상
- `totalTasks`: 전체 Task 수
- `completedTasks`: status == COMPLETED인 Task
- `cancelledTasks`: status == CANCELLED인 Task
- `postponedTasks`: status == POSTPONED인 Task (이동된 원본)

### 5.3 일별 통계
- 각 날짜별로 동일한 통계 계산
- 이동된 Task는 원래 날짜에서 POSTPONED로 카운트

---

## 6. 시간대 처리

### 6.1 저장
- 모든 `Date`는 UTC로 저장
- 날짜 문자열 (weekStartDate 등)은 YYYY-MM-DD 형식

### 6.2 API 응답
- Date 필드는 ISO 8601 형식 (UTC)
- 프론트엔드에서 사용자 timezone으로 변환하여 표시

---

## 7. 동시성 처리

### 7.1 WeeklyPlan 수정
- MongoDB의 원자적 업데이트 사용
- `findOneAndUpdate`로 경쟁 조건 방지

---

## 8. 데이터 정합성

### 8.1 WeeklyPlan 삭제 시
- 관련 ChangeLog도 함께 삭제
- 관련 Notification도 함께 삭제

### 8.2 User 삭제 시
- 모든 WeeklyPlan 삭제
- 모든 ChangeLog 삭제
- 모든 Notification 삭제

---

## Related Documentation

- [Domain Model](./domain-model.md) - Entity definitions and relationships
- [API Contract](./api-contract.md) - REST API specification
- [Backend Integration Guide](./backend-integration-guide.md) - Implementation details

---

**Last Updated:** 2025-01-13
