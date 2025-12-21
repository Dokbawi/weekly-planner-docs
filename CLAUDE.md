# Weekly Planner - 공유 문서

이 레포는 Backend/Frontend에서 submodule로 참조하는 공유 스펙 문서입니다.

## 레포 구조
```
weekly-planner-docs/
├── CLAUDE.md           # 이 파일
├── domain-model.md     # 도메인 모델 정의
├── api-contract.md     # REST API 스펙
├── business-rules.md   # 비즈니스 규칙
└── ui-spec.md          # 화면 명세
```

## 문서 수정 규칙
1. API 변경 시 `api-contract.md` 먼저 수정
2. 도메인 변경 시 `domain-model.md` 먼저 수정
3. Backend/Frontend 각각 submodule update 후 반영
