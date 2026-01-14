# Development Workflow Guide

**Last Updated:** 2025-01-13

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
# 1. docs 폴더로 이동
cd docs/
git checkout main
git pull

# 2. 문서 수정 후 커밋
git add .
git commit -m "docs: update API contract"
git push

# 3. 상위 레포로 돌아가서 submodule 참조 업데이트
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

## 개발 환경 설정

### Backend
```bash
# 의존성 설치
npm install

# 환경 변수 설정 (.env)
MONGODB_URI=mongodb://localhost:27017/weekly_planner
JWT_SECRET=your-secret-key

# 개발 서버 실행
npm run start:dev
```

### 테스트
```bash
npm run test          # 유닛 테스트
npm run test:e2e      # E2E 테스트
npm run test:cov      # 커버리지
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

## Related Documentation

- [API Contract](./api-contract.md)
- [Domain Model](./domain-model.md)
- [Business Rules](./business-rules.md)
- [Backend Integration Guide](./backend-integration-guide.md)

---

**Last Updated:** 2025-01-13
