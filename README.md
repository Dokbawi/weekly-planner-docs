# Weekly Planner - Documentation

**Last Updated:** 2026-01-20

프론트엔드/백엔드 공유 문서 저장소 (Git Submodule)

---

## 프로젝트 개요

| 문서 | 설명 | 대상 |
|------|------|------|
| [PROJECT_OVERVIEW.md](./PROJECT_OVERVIEW.md) | **프로젝트 전체 개요 (포트폴리오용)** | 모든 개발자 |

---

## API 및 도메인

| 문서 | 설명 | 대상 |
|------|------|------|
| [api-contract.md](./api-contract.md) | REST API 명세 | Frontend/Backend |
| [domain-model.md](./domain-model.md) | 엔티티 정의 및 스키마 | Frontend/Backend |
| [business-rules.md](./business-rules.md) | 비즈니스 로직 규칙 | Frontend/Backend |

---

## 프론트엔드

| 문서 | 설명 |
|------|------|
| [backend-integration-guide.md](./backend-integration-guide.md) | API 연동 가이드 |
| [ui-spec.md](./ui-spec.md) | UI/UX 명세 |
| [backend-api-requests.md](./backend-api-requests.md) | 백엔드 API 요청/완료 목록 |

---

## 개발 가이드

| 문서 | 설명 |
|------|------|
| [development-workflow.md](./development-workflow.md) | Git Submodule 관리, 커밋 컨벤션 |

---

## 기술 스택 요약

### Backend
| 기술 | 용도 |
|------|------|
| NestJS 10 | Node.js 프레임워크 |
| TypeScript 5.3+ | 타입 안정성 |
| MongoDB + Mongoose | 데이터베이스 |
| JWT + Passport | 인증 |
| @nestjs/schedule | Cron 스케줄러 |
| Swagger | API 문서화 |

### Infrastructure
| 기술 | 용도 |
|------|------|
| GCP Cloud Run | 서버리스 호스팅 |
| MongoDB Atlas | 관리형 DB |
| GitHub Actions | CI/CD |
| Workload Identity Federation | 키 없는 GCP 인증 |

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

```bash
# 문서 수정
cd docs/
git checkout main && git pull
# 수정 후
git add . && git commit -m "docs: ..." && git push

# 상위 레포 업데이트
cd ..
git add docs
git commit -m "chore: update docs submodule"
git push

# 최신 문서 가져오기
git submodule update --remote docs
```

---

## 관련 저장소

- [Backend](https://github.com/Dokbawi/weekly-planner-backend)
- [Frontend](https://github.com/Dokbawi/weekly-planner-frontend)
