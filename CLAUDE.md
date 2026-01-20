# Weekly Planner - 공유 문서

이 레포는 Backend/Frontend에서 submodule로 참조하는 공유 스펙 문서입니다.

## 레포 구조
```
weekly-planner-docs/
├── CLAUDE.md                      # 이 파일 (문서 작성 규칙)
├── README.md                      # 문서 인덱스
├── portfolio-summary.md           # 프로젝트 기술 요약 (포트폴리오)
├── domain-model.md                # 도메인 모델 정의
├── api-contract.md                # REST API 스펙
├── business-rules.md              # 비즈니스 규칙
├── ui-spec.md                     # 화면 명세
├── backend-integration-guide.md   # 백엔드 통합 가이드
├── backend-api-requests.md        # 백엔드 미구현 API 요청
└── development-workflow.md        # 개발 워크플로우
```

## 문서 수정 규칙

### 1. 문서 업데이트 우선 순위
- API 변경 시: `api-contract.md` 먼저 수정
- 도메인 변경 시: `domain-model.md` 먼저 수정
- 비즈니스 로직 변경 시: `business-rules.md` 먼저 수정

### 2. 코드 예시 작성 규칙
긴 코드 예시 대신 실제 구현 파일 참조:

**좋은 예:**
```markdown
**Implementation:** See `src/plan/plan.service.ts:265`

Key points:
- Original task status changes to POSTPONED
- New task created with PENDING status
```

**나쁜 예:**
50줄 이상의 전체 함수 코드 복사

### 3. 최종 업데이트 날짜
각 문서 하단에 표기:
```markdown
**Last Updated:** 2026-01-20
```

---

**Last Updated:** 2026-01-20
