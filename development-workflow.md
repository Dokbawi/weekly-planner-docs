# Development Workflow Guide

**Last Updated:** 2026-01-20

---

## 프로젝트 구조

```
weekly-planner/
├── weekly-planner-frontend/     # React + TypeScript
│   └── docs/                    # Git submodule → weekly-planner-docs
├── weekly-planner-backend/      # NestJS + TypeScript
│   └── docs/                    # Git submodule → weekly-planner-docs
└── weekly-planner-docs/         # 공유 문서 (이 저장소)
```

---

## Git Submodule 관리

### 문서 수정 시
```bash
cd docs/
git checkout main && git pull
# 문서 수정 후
git add . && git commit -m "docs: update API contract"
git push

# 상위 레포 업데이트
cd ..
git add docs
git commit -m "chore: update docs submodule"
git push
```

### 다른 사람의 변경사항 가져오기
```bash
git pull
git submodule update --remote docs
```

---

## 커밋 메시지 컨벤션

```
<type>(<scope>): <subject>

type: feat, fix, docs, style, refactor, test, chore
scope: auth, plan, task, notification, review
```

### 예시
```
feat(task): add task move functionality
fix(auth): correct JWT token validation
docs: update API contract
chore: update docs submodule
```

---

## CI/CD 파이프라인

### Backend (GitHub Actions → GCP Cloud Run)
```yaml
trigger: push to main branch
         ↓
1. Checkout + Node.js 설정
2. npm install + npm run build
3. Docker 빌드 (Multi-stage)
4. GCP Artifact Registry 푸시
5. Cloud Run 배포
```

### Docker Multi-stage 빌드
```dockerfile
# Build stage
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/main.js"]
```

---

## 개발 환경 설정

### Frontend
```bash
npm install
npm run dev      # http://localhost:3000

# 환경 변수 (.env)
VITE_API_URL=http://localhost:8080/api/v1
```

### Backend
```bash
npm install
npm run start:dev

# 환경 변수 (.env)
MONGODB_URI=mongodb://localhost:27017/weekly_planner
JWT_SECRET=your-secret-key
```

### 테스트
```bash
npm run test          # 유닛 테스트
npm run test:e2e      # E2E 테스트
npm run test:cov      # 커버리지
```

---

## 관련 문서

- [API Contract](./api-contract.md)
- [Domain Model](./domain-model.md)
- [Portfolio Summary](./portfolio-summary.md)
