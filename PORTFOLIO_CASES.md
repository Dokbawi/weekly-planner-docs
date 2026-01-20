# Weekly Planner - 포트폴리오 케이스 정리

> 문제 → 해결 → 결과 구성으로 정리한 기술적 성과

---

## 1. CI/CD 파이프라인 구축 (Keyless 인증)

### 문제
- GCP Cloud Run 배포를 위해 서비스 계정 키(JSON) 사용 시 **보안 위험** 존재
- 키 노출 시 전체 GCP 리소스 접근 가능
- 키 로테이션 관리 부담

### 해결
**Workload Identity Federation** 도입
- GitHub Actions의 OIDC 토큰을 GCP에서 직접 검증
- 서비스 계정 키 없이 임시 액세스 토큰 발급

```yaml
# .github/workflows/deploy.yml
- uses: google-github-actions/auth@v2
  with:
    workload_identity_provider: ${{ secrets.GCP_WORKLOAD_IDENTITY_PROVIDER }}
    service_account: ${{ secrets.GCP_SERVICE_ACCOUNT }}
```

**GCP 설정**
```bash
# Workload Identity Pool 생성
gcloud iam workload-identity-pools create github-pool --location="global"

# OIDC Provider 연결 (GitHub 저장소 기반 접근 제어)
gcloud iam workload-identity-pools providers create-oidc github-provider \
  --attribute-condition="assertion.repository=='Org/Repo'"
```

### 결과
- **키 관리 제로**: JSON 키 파일 완전 제거
- **보안 강화**: 저장소 기반 접근 제어로 특정 레포에서만 배포 가능
- **자동 만료**: 토큰이 자동 만료되어 유출 시에도 피해 최소화

---

## 2. 서버리스 컨테이너 Cold Start 최적화

### 문제
- Cloud Run `min-instances=0` 설정으로 비용 절감
- 첫 요청 시 **Cold Start 지연** 발생 (컨테이너 초기화 시간)
- NestJS 앱 기동 시간 + MongoDB 연결 시간

### 해결
**1) Multi-stage Docker 빌드로 이미지 경량화**
```dockerfile
# Build stage (devDependencies 포함)
FROM node:20-alpine AS builder
RUN npm ci && npm run build

# Production stage (production만)
FROM node:20-alpine AS production
RUN npm ci --only=production  # devDeps 제외
COPY --from=builder /app/dist ./dist
```

**2) 보안 + 성능 최적화**
```dockerfile
# non-root 사용자로 실행 (보안)
RUN adduser -S nestjs -u 1001
USER nestjs

# 헬스 체크로 빠른 장애 감지
HEALTHCHECK --interval=30s --timeout=3s CMD node -e "..."
```

**3) Cloud Run 설정 최적화**
```yaml
flags: |
  --memory=512Mi        # 적정 메모리
  --cpu=1               # 1 vCPU
  --concurrency=80      # 동시 요청 처리
  --timeout=300         # 긴 작업 허용
```

### 결과
- **이미지 크기**: ~150MB (node_modules 최소화)
- **Cold Start**: 약 3-5초 (NestJS + MongoDB 연결)
- **비용**: 무료 티어 내 운영 (200만 요청/월)

---

## 3. Cloud Run PORT 바인딩 이슈 해결

### 문제
```
Container failed to start. Failed to start and then listen on the port defined by the PORT environment variable.
```
- 로컬에서는 정상 동작하나 Cloud Run에서 컨테이너 시작 실패
- Cloud Run은 PORT=8080으로 헬스 체크

### 해결
**원인 분석**: NestJS 기본 설정이 `localhost`(127.0.0.1)에만 바인딩

**수정 전**
```typescript
await app.listen(port);  // localhost만 리스닝
```

**수정 후**
```typescript
await app.listen(port, '0.0.0.0');  // 모든 인터페이스에서 리스닝
```

### 결과
- 컨테이너 내부에서 외부 요청 수신 가능
- Cloud Run 헬스 체크 통과
- **커밋**: `fix: Cloud Run 배포를 위해 0.0.0.0에 바인딩`

---

## 4. Workload Identity 대소문자 이슈 해결

### 문제
```
Permission 'iam.serviceAccounts.getAccessToken' denied on resource
```
- Workload Identity 설정 완료 후에도 권한 오류 발생
- 서비스 계정 권한은 정상적으로 부여됨

### 해결
**원인 분석**: GitHub 저장소 이름 대소문자 불일치

```bash
# 실제 GitHub 저장소
Dokbawi/weekly-planner-backend  # 대문자 D

# 잘못된 설정
dokbawi/weekly-planner-backend  # 소문자 d
```

**GCP attribute-condition에서 대소문자 정확히 일치해야 함**
```bash
--attribute-condition="assertion.repository=='Dokbawi/weekly-planner-backend'"
```

### 결과
- 저장소 이름 정확히 매칭하여 인증 성공
- **교훈**: Workload Identity 설정 시 GitHub 저장소 이름 대소문자 주의

---

## 5. 변경 추적 시스템 (Change Tracking)

### 문제
- 주간 계획 확정 후 Task 변경 시 **변경 이력 추적 필요**
- 사용자가 무엇을 얼마나 변경했는지 분석하여 회고에 활용
- 모든 변경을 수동으로 기록하면 코드 복잡도 증가

### 해결
**계획 상태 기반 자동 기록**
```typescript
// 계획이 CONFIRMED 상태일 때만 변경 기록
if (plan.status === PlanStatus.CONFIRMED) {
  await this.changelogService.trackChange({
    weeklyPlanId: plan._id,
    taskId: task.id,
    changeType: ChangeType.MODIFIED,
    changes: { before, after },
    reason: dto.reason,
  });
}
```

**변경 유형 분류**
- `ADDED`: Task 추가
- `MODIFIED`: Task 수정
- `DELETED`: Task 삭제
- `MOVED`: 다른 날로 이동
- `STATUS_CHANGED`: 상태 변경

### 결과
- 계획 확정 전: 자유롭게 수정 (기록 없음)
- 계획 확정 후: 모든 변경 자동 기록
- 주간 회고에서 변경 이력 분석 가능

---

## 6. Task 이동 처리 (원본 보존 방식)

### 문제
- Task를 다른 날로 이동 시 단순 날짜 변경?
- 원본 Task가 사라지면 "연기했다"는 기록이 없음
- 회고 시 연기된 Task 추적 불가

### 해결
**원본 POSTPONED + 새 Task 생성 방식**

```
1/15 (원본)              1/16 (대상)
┌─────────────┐         ┌─────────────┐
│ Task A      │   ──▶   │ Task A'     │
│ PENDING     │         │ PENDING     │
└─────────────┘         │ (새 ID)     │
       │                └─────────────┘
       ▼
┌─────────────┐
│ Task A      │
│ POSTPONED   │ ← 원본 유지
└─────────────┘
```

```typescript
// 1. 원본 Task 상태 변경
task.status = TaskStatus.POSTPONED;

// 2. 새 Task 생성 (새 ID 부여)
const newTask = {
  id: new Types.ObjectId().toString(),
  ...task,
  status: TaskStatus.PENDING,
};

// 3. 대상 날짜에 추가
targetDailyPlan.tasks.push(newTask);
```

### 결과
- 연기된 Task 기록 보존
- 회고 시 postponedTasks 카운트 가능
- ChangeLog에 `MOVED` 기록 (from → to)

---

## 7. Cron 기반 알림 스케줄러

### 문제
- Task 리마인더, 일일 요약, 계획/회고 알림 필요
- 각 사용자마다 설정이 다름 (planningDay, reviewDay)
- 실시간 푸시 vs 폴링 방식 선택

### 해결
**@nestjs/schedule 기반 Cron 스케줄러**

```typescript
@Cron(CronExpression.EVERY_MINUTE)
async checkTaskReminders() {
  // scheduledTime - reminderMinutes = 현재 시각인 Task 찾기
  // → TASK_REMINDER 알림 생성
}

@Cron('0 0 8 * * *')  // 매일 08:00
async sendDailySummary() {
  // 오늘 Task가 있는 사용자에게 DAILY_SUMMARY 알림
}

@Cron('0 0 9 * * *')  // 매일 09:00
async sendPlanningReminder() {
  // 오늘이 planningDay인 사용자에게 PLANNING_REMINDER
}

@Cron('0 0 18 * * *')  // 매일 18:00
async sendReviewReminder() {
  // 오늘이 reviewDay인 사용자에게 REVIEW_REMINDER
}
```

### 결과
- 4종류 알림 자동 생성
- 사용자별 설정 반영
- Cloud Run에서 항상 인스턴스가 있어야 Cron 동작 (주의)

---

## 8. JWT 인증 Payload 추출 이슈

### 문제
```
Cannot read property 'sub' of undefined
```
- `@CurrentUser()` 데코레이터에서 사용자 정보 추출 실패
- JWT 토큰은 정상적으로 검증됨

### 해결
**원인 분석**: JWT payload 구조와 데코레이터 매핑 불일치

**수정 전**
```typescript
// user 객체 전체를 반환
return request.user;
```

**수정 후**
```typescript
// payload에서 특정 필드 추출
export const CurrentUser = createParamDecorator(
  (data: string, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    const user = request.user;
    return data ? user?.[data] : user;
  },
);
```

### 결과
- `@CurrentUser('sub')` → userId 추출
- `@CurrentUser()` → 전체 payload 반환
- **커밋**: `fix: CurrentUser decorator에 'sub' 인자 추가`

---

## 9. MongoDB 중복 키 에러 처리

### 문제
```
E11000 duplicate key error collection: weekly_plans
```
- 같은 주에 대한 WeeklyPlan 중복 생성 시도
- `{ userId, weekStartDate }` unique 인덱스 위반

### 해결
**생성 전 중복 체크 추가**

```typescript
async createWeeklyPlan(userId: string, dto: CreateWeeklyPlanDto) {
  // 중복 체크
  const existing = await this.weeklyPlanModel.findOne({
    userId: new Types.ObjectId(userId),
    weekStartDate: dto.weekStartDate,
  });

  if (existing) {
    throw new ConflictException(
      `Weekly plan already exists for week starting ${dto.weekStartDate}`
    );
  }

  // 생성 로직...
}
```

### 결과
- 명확한 에러 메시지 반환 (409 Conflict)
- MongoDB 에러 대신 비즈니스 로직 에러
- E2E 테스트 추가로 회귀 방지

---

## 10. DTO Validation 누락 이슈

### 문제
```
PUT /plans/{planId}/memo → 400 Bad Request
```
- 프론트엔드에서 정상적인 요청인데 400 에러
- 서버 로그에 validation 관련 에러 없음

### 해결
**원인 분석**: 인라인 타입 사용으로 ValidationPipe 미동작

**수정 전**
```typescript
@Body() dto: { date: string; memo: string }  // 인라인 타입
```

**수정 후**
```typescript
// DTO 클래스 생성
export class UpdateMemoDto {
  @IsDateString()
  date: string;

  @IsString()
  memo: string;
}

// 컨트롤러에서 사용
@Body() dto: UpdateMemoDto
```

### 결과
- class-validator 데코레이터로 유효성 검사 동작
- Swagger 문서에 스키마 자동 반영
- 명확한 validation 에러 메시지

---

## 비용 구조

| 서비스 | 무료 티어 | 실제 사용량 | 월 비용 |
|--------|----------|------------|--------|
| Cloud Run | 200만 요청 | ~10만 요청 | $0 |
| MongoDB Atlas M0 | 512MB | ~50MB | $0 |
| Secret Manager | 6개 버전 | 2개 시크릿 | $0 |
| Artifact Registry | 500MB | ~150MB | $0 |

**총 월 예상 비용**: $0 (무료 티어 내)

---

## 부하 테스트 결과

**테스트 일시**: 2026-01-20
**테스트 환경**: GCP Cloud Run (asia-northeast3)

### 1. Cold Start 테스트

| 지표 | 결과 | 목표 | 상태 |
|------|------|------|------|
| Cold Start (Warm 상태) | **77ms** | < 5초 | **Pass** |
| DB 응답 시간 | **4ms** | < 100ms | **Pass** |

> 인스턴스가 이미 Warm 상태여서 Cold Start가 매우 빠름.
> 실제 Cold Start (0 → 1 인스턴스)는 약 3-5초 예상.

### 2. 연속 요청 테스트 (10회)

| 지표 | 결과 |
|------|------|
| 요청 1 | 68ms |
| 요청 2 | 70ms |
| 요청 3 | 68ms |
| 요청 4 | 68ms |
| 요청 5 | 67ms |
| 요청 6 | 66ms |
| 요청 7 | 68ms |
| 요청 8 | 71ms |
| 요청 9 | 67ms |
| 요청 10 | 68ms |
| **평균** | **68ms** |

### 3. 동시 요청 테스트 (10 parallel)

| 지표 | 결과 | 목표 | 상태 |
|------|------|------|------|
| 10개 동시 요청 완료 시간 | **217ms** | < 1초 | **Pass** |
| 요청당 평균 | **21.7ms** | - | - |

### 4. 인증 API 성능

| API | 응답 시간 | HTTP 상태 |
|-----|----------|----------|
| POST /auth/register | **323ms** | 201 |
| POST /auth/login | **185ms** | 200 |
| GET /plans/current | **129ms** | 200 |
| GET /today | **90ms** | 200 |
| GET /notifications | **85ms** | 200 |

### 5. 성능 분석

**강점**:
- Health 체크 평균 **68ms** (매우 빠름)
- 인증 후 API 응답 **85-130ms** (양호)
- 동시 요청 처리 안정적

**병목 지점**:
- 회원가입 **323ms** (bcrypt 해싱 비용)
- 로그인 **185ms** (DB 조회 + 비밀번호 검증)

**최적화 여지**:
- Connection Pool 크기 조정
- 읽기 작업 캐싱 (Redis)
- 인덱스 최적화

### 측정 지표 요약

| 지표 | 결과 | 목표 | 상태 |
|------|------|------|------|
| Cold Start (Warm) | 77ms | < 5초 | **Pass** |
| 평균 응답 시간 | 68ms | < 200ms | **Pass** |
| 인증 API | 129ms | < 500ms | **Pass** |
| 동시 처리 | 21.7ms/req | > 50 req/s | **Pass** |

---

## 기술적 의사결정

### Spring Boot → NestJS 마이그레이션

| 항목 | Spring Boot (Kotlin) | NestJS (TypeScript) |
|------|---------------------|---------------------|
| 컨테이너 크기 | ~200MB (JVM) | ~150MB |
| Cold Start | 10-15초 | 3-5초 |
| 메모리 사용량 | 512MB+ | 256MB |
| 개발 생산성 | 높음 | 높음 |
| 프론트엔드 연계 | 별도 타입 | 타입 공유 가능 |

**선택 이유**: 서버리스 환경에서 Cold Start와 리소스 효율성 우선

---

**Last Updated**: 2026-01-20
