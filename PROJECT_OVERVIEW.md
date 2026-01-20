# Weekly Planner - 프로젝트 개요

> 주간 계획 및 할 일 관리 풀스택 애플리케이션

## 프로젝트 소개

Weekly Planner는 주간 단위로 할 일을 계획하고, 변경 사항을 추적하며, 주간 회고를 통해 생산성을 분석할 수 있는 풀스택 웹 애플리케이션입니다.

### 핵심 기능

| 기능 | 설명 |
|------|------|
| **주간 계획 관리** | 월요일~일요일 단위로 할 일을 계획하고 관리 |
| **계획 확정 및 변경 추적** | 계획 확정 후 모든 변경 사항을 자동 기록 |
| **Task 이동** | 할 일을 다른 날로 이동 시 원본은 POSTPONED 처리 |
| **주간 회고** | 완료율, 변경 이력 등 주간 통계 자동 분석 |
| **알림 시스템** | Task 리마인더, 일일 요약, 계획/회고 알림 |
| **출퇴근 계산기** | 도착 시간 기준 출발 시간 역산 기능 |

---

## 기술 스택

### Backend

| 구분 | 기술 | 설명 |
|------|------|------|
| Framework | **NestJS 10** | TypeScript 기반 Node.js 프레임워크 |
| Language | **TypeScript 5.3+** | 타입 안정성 및 개발 생산성 |
| Database | **MongoDB 8.0** | NoSQL 문서 데이터베이스 |
| ODM | **Mongoose 8.0** | MongoDB 객체 모델링 |
| Auth | **JWT + Passport** | 토큰 기반 인증 |
| Scheduler | **@nestjs/schedule** | Cron 기반 알림 스케줄러 |
| Documentation | **Swagger** | OpenAPI 자동 문서화 |
| Logging | **Winston + Morgan** | 구조화된 로깅 |

### Infrastructure

| 구분 | 기술 | 설명 |
|------|------|------|
| Cloud | **GCP Cloud Run** | 서버리스 컨테이너 호스팅 |
| Database | **MongoDB Atlas** | 관리형 MongoDB 서비스 |
| CI/CD | **GitHub Actions** | 자동 빌드 및 배포 |
| Registry | **Artifact Registry** | Docker 이미지 저장소 |
| Secrets | **Secret Manager** | 환경변수 보안 관리 |
| Auth | **Workload Identity Federation** | 키 없는 GCP 인증 |

### Frontend (예정)

| 구분 | 기술 |
|------|------|
| Framework | Next.js 14 / React 18 |
| UI | Tailwind CSS |
| State | Zustand / React Query |

---

## 아키텍처

### 시스템 구성도

```
┌─────────────────────────────────────────────────────────────────┐
│                        GitHub Actions                            │
│                    (CI/CD Pipeline)                              │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     GCP Cloud Run                                │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                  NestJS Application                      │    │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐   │    │
│  │  │   Auth   │ │   Plan   │ │ Changelog│ │  Review  │   │    │
│  │  │  Module  │ │  Module  │ │  Module  │ │  Module  │   │    │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘   │    │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐               │    │
│  │  │Notification│ │ Commute │ │  Health  │               │    │
│  │  │  Module  │ │  Module  │ │  Module  │               │    │
│  │  └──────────┘ └──────────┘ └──────────┘               │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    MongoDB Atlas (M0)                            │
│              (Managed NoSQL Database)                            │
└─────────────────────────────────────────────────────────────────┘
```

### 모듈 구조

```
src/
├── auth/              # JWT 인증 (Passport)
├── user/              # 사용자 관리
├── plan/              # 주간 계획 및 Task CRUD
├── changelog/         # 변경 추적 시스템
├── notification/      # 알림 및 스케줄러
├── review/            # 주간 회고
├── commute-routine/   # 출퇴근 계산기
├── health/            # 헬스 체크
└── common/            # 공통 모듈
    ├── decorators/    # @CurrentUser, @Public
    ├── filters/       # 전역 예외 필터
    ├── interceptors/  # 로깅, 성능 측정
    └── dto/           # 공통 응답 wrapper
```

---

## CI/CD 파이프라인

### GitHub Actions Workflow

```yaml
# .github/workflows/deploy.yml
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      id-token: write  # Workload Identity Federation

    steps:
      - uses: actions/checkout@v4

      # 1. Workload Identity로 GCP 인증 (키 없음)
      - uses: google-github-actions/auth@v2
        with:
          workload_identity_provider: ${{ secrets.GCP_WORKLOAD_IDENTITY_PROVIDER }}
          service_account: ${{ secrets.GCP_SERVICE_ACCOUNT }}

      # 2. Docker 빌드 & Artifact Registry 푸시
      - run: |
          docker build -t $REGION-docker.pkg.dev/$PROJECT_ID/weekly-planner/backend:$SHA .
          docker push ...

      # 3. Cloud Run 배포
      - uses: google-github-actions/deploy-cloudrun@v2
        with:
          service: weekly-planner-backend
          image: ...
          secrets: |
            MONGODB_URI=MONGODB_URI:latest
            JWT_SECRET=JWT_SECRET:latest
```

### 배포 흐름

```
1. main 브랜치 Push
       │
       ▼
2. GitHub Actions 트리거
       │
       ▼
3. Workload Identity Federation으로 GCP 인증
   (서비스 계정 키 없이 OIDC 토큰으로 인증)
       │
       ▼
4. Docker 이미지 빌드
       │
       ▼
5. Artifact Registry에 푸시
       │
       ▼
6. Cloud Run 배포
   - Secret Manager에서 환경변수 주입
   - min-instances=0 (비용 최적화)
   - max-instances=10 (자동 스케일링)
       │
       ▼
7. 배포 완료 (URL 출력)
```

---

## 핵심 기술 구현

### 1. 변경 추적 시스템

계획 확정(CONFIRMED) 후 모든 Task 변경을 자동으로 기록합니다.

```typescript
// 계획이 CONFIRMED 상태일 때만 변경 기록
if (plan.status === PlanStatus.CONFIRMED) {
  await this.changelogService.trackChange({
    weeklyPlanId: plan._id.toString(),
    taskId: task.id,
    changeType: ChangeType.MODIFIED,
    changes: { before, after },
    reason: dto.reason,
  });
}
```

**변경 유형**: ADDED, MODIFIED, DELETED, MOVED, STATUS_CHANGED

### 2. Task 이동 처리

단순 이동이 아닌, 원본 보존 방식으로 구현했습니다.

```
원본 Task (1/15)          대상 Task (1/16)
┌─────────────────┐       ┌─────────────────┐
│ status: PENDING │  ───▶ │ status: PENDING │
│                 │       │ (새 ID 생성)     │
└─────────────────┘       └─────────────────┘
        │
        ▼
┌─────────────────┐
│status: POSTPONED│
│ (원본 유지)      │
└─────────────────┘
```

### 3. Cron 기반 알림 스케줄러

```typescript
@Cron(CronExpression.EVERY_MINUTE)
async checkTaskReminders() {
  // scheduledTime - reminderMinutes = 현재 시각인 Task 찾기
}

@Cron('0 0 8 * * *')  // 매일 08:00
async sendDailySummary() {
  // 오늘 Task가 있는 사용자에게 요약 알림
}

@Cron('0 0 9 * * *')  // 매일 09:00
async sendPlanningReminder() {
  // planningDay인 사용자에게 계획 수립 알림
}
```

### 4. Workload Identity Federation

서비스 계정 키 없이 GitHub Actions에서 GCP에 인증합니다.

```
GitHub Actions                    GCP
     │                             │
     │  OIDC Token (JWT)           │
     │ ─────────────────────────▶  │
     │                             │
     │  Access Token               │
     │ ◀─────────────────────────  │
     │                             │
     │  API 요청 (Cloud Run 배포)   │
     │ ─────────────────────────▶  │
```

**장점**:
- 키 노출 위험 없음
- 키 로테이션 불필요
- GitHub 저장소 기반 접근 제어

---

## 데이터 모델

### ERD (간략)

```
User (사용자)
├── _id: ObjectId
├── email: string (unique)
├── password: string (bcrypt)
├── planningDay: DayOfWeek
└── reviewDay: DayOfWeek

WeeklyPlan (주간 계획)
├── _id: ObjectId
├── userId: ObjectId → User
├── weekStartDate: string
├── weekEndDate: string
├── status: DRAFT | CONFIRMED
└── dailyPlans: DailyPlan[]
    └── tasks: Task[]

ChangeLog (변경 이력)
├── _id: ObjectId
├── weeklyPlanId: ObjectId → WeeklyPlan
├── changeType: ADDED | MODIFIED | DELETED | MOVED
├── changes: Object
└── reason?: string

CommuteRoutine (출퇴근 루틴)
├── _id: ObjectId
├── userId: ObjectId → User
├── steps: CommuteStep[]
└── totalMinutes: number
```

### MongoDB 인덱스 전략

| Collection | Index | 용도 |
|------------|-------|------|
| users | `{ email: 1 }` unique | 로그인 시 조회 |
| weekly_plans | `{ userId: 1, weekStartDate: -1 }` unique | 사용자별 주간 계획 |
| changelogs | `{ weeklyPlanId: 1, changedAt: -1 }` | 계획별 변경 이력 |
| notifications | `{ userId: 1, isRead: 1, createdAt: -1 }` | 읽지 않은 알림 조회 |

---

## API 엔드포인트 (주요)

| Method | Endpoint | 설명 |
|--------|----------|------|
| POST | `/auth/register` | 회원가입 |
| POST | `/auth/login` | 로그인 |
| GET | `/plans/current` | 현재 주 계획 (자동 생성) |
| POST | `/plans/{id}/confirm` | 계획 확정 |
| POST | `/plans/{id}/tasks` | Task 추가 |
| POST | `/plans/{id}/tasks/{taskId}/move` | Task 이동 |
| PUT | `/plans/{id}/tasks/reorder` | Task 순서 변경 |
| GET | `/plans/{id}/changes` | 변경 이력 |
| GET | `/reviews/{id}` | 주간 회고 |
| GET | `/notifications` | 알림 목록 |
| POST | `/commute-routines` | 출퇴근 루틴 생성 |
| POST | `/commute-routines/{id}/calculate` | 출발 시간 계산 |

**전체 API 문서**: [api-contract.md](./api-contract.md)

---

## 비용 구조

| 서비스 | 무료 티어 | 예상 비용 |
|--------|----------|----------|
| Cloud Run | 200만 요청/월 | $0 |
| MongoDB Atlas M0 | 512MB | $0 |
| Secret Manager | 6개 시크릿 | $0 |
| Artifact Registry | 500MB | $0 |

**총 예상 비용**: 무료 티어 내 운영 가능 (소규모 사용 기준)

---

## 관련 저장소

| 저장소 | 설명 |
|--------|------|
| [weekly-planner-backend](https://github.com/Dokbawi/weekly-planner-backend) | NestJS 백엔드 API |
| [weekly-planner-frontend](https://github.com/Dokbawi/weekly-planner-frontend) | Next.js 프론트엔드 |
| [weekly-planner-docs](https://github.com/Dokbawi/weekly-planner-docs) | 공유 문서 (Submodule) |

---

## 로컬 개발 환경

```bash
# 의존성 설치
npm install

# 환경변수 설정
cp .env.example .env
# MONGODB_URI, JWT_SECRET 설정

# 개발 서버 실행
npm run start:dev

# API 문서 확인
open http://localhost:3000/api-docs
```

---

**Last Updated**: 2026-01-20
