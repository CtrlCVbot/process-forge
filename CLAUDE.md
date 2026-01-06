# CLAUDE.md

## 프로젝트 컨텍스트
이 프로젝트의 규칙은 `.agent/` 폴더에 정의되어 있습니다.

## 필수 참조 파일
- .agent/MASTER.md
- .agent/rules/ownership.md
- .agent/rules/change-control.md

## 스킬 사용
프로세스 분석 요청 시 `.agent/skills/process-analyzer.md` 규칙을 적용하세요.
```

또는 **하위 폴더에 CLAUDE.md를 추가**하는 방법도 있습니다:
```
/.agent
├── CLAUDE.md          # 이 폴더가 규칙 저장소임을 선언
├── MASTER.md
├── /rules
│   └── CLAUDE.md      # (선택) 규칙 폴더 설명
└── /skills
    └── CLAUDE.md      # (선택) 스킬 폴더 설명
```

---

## 장점

| 구조 | 장점 |
|------|------|
| `.agent/` 중앙화 | 도구 변경해도 규칙 재작성 불필요 |
| 진입점 분리 | 각 도구 특성에 맞게 최적화 가능 |
| CLAUDE.md 참조 | Claude Code가 컨텍스트로 로드 |

---

## 주의점

| 항목 | 설명 |
|------|------|
| **명시적 참조 필요** | Claude Code는 "알아서" 폴더를 탐색하지 않음. CLAUDE.md에서 명시해야 함 |
| **파일 크기** | 너무 많은 규칙 파일 → 컨텍스트 초과. 핵심만 참조하고 나머지는 필요 시 로드 |
| **트리거 조건** | 스킬/에이전트에 "언제 적용할지" 트리거 조건 명시 필요 |

---

## 실제 작동 확인용 최소 구조

테스트해보려면 이 구조로 시작:
```
/process-forge
├── CLAUDE.md
├── .cursorrules
└── .agent/
    ├── MASTER.md
    └── rules/
        └── document-schema.md