# Weekly Planner - Shared Documentation

**Last Updated:** 2026-01-19

프론트엔드/백엔드 공유 문서 저장소 (Git Submodule)

---

## 문서 목록

### API 및 도메인

| 문서 | 설명 | 대상 |
|------|------|------|
| [api-contract.md](./api-contract.md) | REST API 명세 | Frontend/Backend |
| [domain-model.md](./domain-model.md) | 엔티티 정의 및 스키마 | Frontend/Backend |
| [business-rules.md](./business-rules.md) | 비즈니스 로직 규칙 | Frontend/Backend |

### 프론트엔드

| 문서 | 설명 |
|------|------|
| [backend-integration-guide.md](./backend-integration-guide.md) | API 연동 가이드 |
| [ui-spec.md](./ui-spec.md) | UI/UX 명세 |
| [backend-api-requests.md](./backend-api-requests.md) | 백엔드 미구현 API 요청 목록 |

### 개발 가이드

| 문서 | 설명 |
|------|------|
| [development-workflow.md](./development-workflow.md) | Git Submodule 관리, 커밋 컨벤션 |

---

## 인프라 현황

| 서비스 | 플랫폼 | 상태 |
|--------|--------|------|
| Backend API | GCP Cloud Run | 운영 중 |
| Database | MongoDB Atlas (M0) | 운영 중 |
| Frontend | (예정) | - |
| CI/CD | GitHub Actions | 자동 배포 |

---

## Submodule 사용법

### 문서 수정
```bash
cd docs/
git checkout main && git pull
# 문서 수정
git add . && git commit -m "docs: ..." && git push
```

### 상위 레포 업데이트
```bash
cd ..
git add docs
git commit -m "chore: update docs submodule"
git push
```

### 최신 문서 가져오기
```bash
git submodule update --remote docs
```

---

## 관련 저장소

- [Backend](https://github.com/Dokbawi/weekly-planner-backend)
- [Frontend](https://github.com/Dokbawi/weekly-planner-frontend)
