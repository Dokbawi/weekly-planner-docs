# Backend Integration Guide

**Last Updated:** 2026-01-20

프론트엔드 개발자를 위한 백엔드 API 연동 가이드입니다.

---

## 배포 환경

| 서비스 | 플랫폼 | 설정 |
|--------|--------|------|
| Backend | GCP Cloud Run | 서울 리전, 자동 스케일링 |
| Database | MongoDB Atlas | M0 무료 티어 |
| CI/CD | GitHub Actions | main 브랜치 푸시 시 자동 배포 |

---

## API 현황

### 구현 완료
- `POST /auth/register`, `POST /auth/login`, `GET /auth/me`
- `GET /plans`, `GET /plans/current`, `GET /plans/{planId}`
- `POST /plans/{planId}/tasks`, `PUT /plans/{planId}/tasks/{taskId}`
- `POST /plans/{planId}/tasks/{taskId}/move`
- `GET /reviews/{planId}`
- `GET /notifications`, `PUT /notifications/{id}/read`

전체 API 목록은 [api-contract.md](./api-contract.md) 참조

---

## 응답 형식

```json
// 성공
{ "success": true, "data": { ... } }

// 실패
{ "success": false, "error": { "code": "...", "message": "..." } }

// 페이지네이션
{ "success": true, "data": { "content": [...], "page": 0, "size": 10, "totalElements": 100 } }
```

---

## 인증

### 로그인 응답
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIs...",
  "tokenType": "Bearer",
  "expiresIn": 86400
}
```

### 요청 헤더
```
Authorization: Bearer {accessToken}
```

---

## 주요 API 사용법

### Task 생성
```
POST /plans/{planId}/tasks?date=2026-01-20
{
  "title": "회의 참석",
  "scheduledTime": "14:00",
  "reminderMinutesBefore": 30,
  "priority": "HIGH"
}
```
**주의**: `date`는 query parameter로 전달

### Task 이동
```
POST /plans/{planId}/tasks/{taskId}/move
{
  "targetDate": "2026-01-21",
  "reason": "시간 부족"
}
```
- 원본 Task는 POSTPONED 상태로 변경
- 새 날짜에 복사본 생성 (PENDING)

---

## 데이터 정규화

프론트엔드는 `dailyPlans`를 객체(Record)로 기대하지만, 백엔드는 배열로 반환합니다.

```typescript
// 백엔드 응답
dailyPlans: [{ date: '2026-01-12', tasks: [] }, ...]

// 프론트엔드 기대
dailyPlans: { '2026-01-12': { date: '...', tasks: [] }, ... }

// 변환 함수
function normalizeDailyPlans(plans) {
  return plans.reduce((acc, plan) => ({ ...acc, [plan.date]: plan }), {})
}
```

**구현 위치**: `src/api/plans.ts`, `src/api/reviews.ts`

---

## 자주 발생하는 문제

### 1. Task 날짜 불일치
- **원인**: date가 body에 있음
- **해결**: `POST /tasks?date=yyyy-MM-dd`

### 2. 401 Unauthorized
- **확인**: localStorage의 토큰 존재 여부
- **확인**: Authorization 헤더 형식 (`Bearer {token}`)

### 3. 400 Bad Request (계획 생성)
- **원인**: 해당 주 계획이 이미 존재
- **해결**: `GET /plans/current` 사용 (없으면 자동 생성)

---

## 관련 문서

- [API Contract](./api-contract.md) - 전체 API 명세
- [Domain Model](./domain-model.md) - 엔티티 정의
- [Business Rules](./business-rules.md) - 비즈니스 로직
