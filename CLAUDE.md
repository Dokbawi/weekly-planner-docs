# Weekly Planner - 공유 문서

이 레포는 Backend/Frontend에서 submodule로 참조하는 공유 스펙 문서입니다.

## 레포 구조
```
weekly-planner-docs/
├── CLAUDE.md                      # 이 파일
├── README.md                      # 문서 인덱스 및 가이드
├── domain-model.md                # 도메인 모델 정의
├── api-contract.md                # REST API 스펙
├── business-rules.md              # 비즈니스 규칙
├── ui-spec.md                     # 화면 명세
├── backend-integration-guide.md   # 백엔드 통합 가이드
└── development-workflow.md        # 개발 워크플로우
```

## 문서 수정 규칙

### 1. 문서 업데이트 우선 순위
- API 변경 시: `api-contract.md` 먼저 수정
- 도메인 변경 시: `domain-model.md` 먼저 수정
- 비즈니스 로직 변경 시: `business-rules.md` 먼저 수정
- Backend/Frontend 각각 submodule update 후 반영

### 2. 코드 예시 작성 규칙 ⭐
**중요**: 긴 코드 예시 대신 실제 구현 파일 참조

**좋은 예:**
```markdown
**Implementation:** See `src/plan/plan.service.ts:265` for task move logic

Key points:
- Original task status changes to POSTPONED
- New task created with PENDING status
- ChangeLog entry created automatically
```

**나쁜 예:**
```markdown
```typescript
// 50줄의 전체 함수 코드 복사...
async moveTask(taskId: string, targetDate: string) {
  // ... 긴 코드 ...
}
```
```

### 3. 문서 간 크로스 레퍼런스
모든 문서는 관련 문서로 링크:
```markdown
## Related Documentation

- [Domain Model](./domain-model.md) - Entity definitions
- [API Contract](./api-contract.md) - REST API spec
- [Business Rules](./business-rules.md) - Business logic
- [README](./README.md) - Documentation index
```

### 4. 파일 경로 표준
- 항상 forward slash 사용: `src/path/to/file.ts`
- 상대 경로로 문서 링크: `./domain-model.md`
- 절대 경로로 소스 파일 참조: `src/auth/auth.service.ts`

### 5. 최종 업데이트 날짜
각 문서 하단에 표기:
```markdown
**Last Updated:** 2025-12-29
```

---

## Related Documentation

- [README](./README.md) - Documentation index and navigation
- [Development Workflow](./development-workflow.md) - Submodule management and conventions

---

**Last Updated:** 2024-12-31
