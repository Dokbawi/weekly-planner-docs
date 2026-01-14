# Weekly Planner - Documentation Index

**Last Updated:** 2025-01-13

Weekly Planner 프로젝트의 공유 문서 저장소입니다.

---

## 문서 구조

### 핵심 문서

| 문서 | 설명 |
|------|------|
| [API Contract](./api-contract.md) | REST API 명세 |
| [Domain Model](./domain-model.md) | 엔티티 정의 및 스키마 |
| [Business Rules](./business-rules.md) | 비즈니스 로직 규칙 |

### 개발 가이드

| 문서 | 설명 |
|------|------|
| [Backend Integration Guide](./backend-integration-guide.md) | 프론트엔드 개발자를 위한 API 연동 가이드 |

---

## 기술 스택

### Backend
- **Framework:** NestJS 10.x
- **Language:** TypeScript 5.x
- **Database:** MongoDB + Mongoose
- **Auth:** JWT (Passport)

### Frontend
- **Framework:** React + TypeScript
- **State Management:** Zustand
- **UI:** shadcn/ui

---

## 빠른 시작

### API 서버
```bash
cd weekly-planner-backend
npm install
npm run start:dev
```
- API: http://localhost:3000
- Swagger: http://localhost:3000/api-docs

---

## 문서 업데이트 가이드

이 저장소는 git submodule로 frontend/backend 레포에서 공유됩니다.

### 문서 수정 후
```bash
# docs 폴더에서
cd docs/
git add .
git commit -m "docs: update API contract"
git push

# 상위 레포에서
cd ..
git add docs
git commit -m "chore: update docs submodule"
git push
```

---

## Related

- [Backend Repository](https://github.com/Dokbawi/weekly-planner-backend)
- [Frontend Repository](https://github.com/Dokbawi/weekly-planner-frontend)

---

**Last Updated:** 2025-01-13
