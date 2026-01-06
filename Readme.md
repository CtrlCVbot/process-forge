## 도구별 규칙 인식 구조

각 AI 코딩 도구는 서로 다른 진입점을 사용합니다:

| 도구 | 인식 파일/폴더 | 비고 |
|------|---------------|------|
| **Claude Code** | `CLAUDE.md`, `.claude/` | 프로젝트 루트 + 하위 폴더 |
| **Cursor AI** | `.cursorrules`, `.cursor/rules/` | 프로젝트 루트 |
| **Windsurf** | `.windsurfrules`, `.windsurf/rules/` | 프로젝트 루트 |

---

## 제안: 도구 중립적 구조 + 진입점 분리

```
/process-forge
│
├── CLAUDE.md                    # Claude Code 진입점 → .agent/ 참조
├── .cursorrules                 # Cursor 진입점 → .agent/ 참조
├── .windsurfrules               # Windsurf 진입점 → .agent/ 참조
│
└── .agent/                      # 실제 규칙/스킬/에이전트 (도구 중립)
    │
    ├── MASTER.md                # 마스터 규칙
    │
    ├── /rules                   # 프로젝트 메타 규칙
    │   ├── ownership.md
    │   ├── change-control.md
    │   ├── document-schema.md
    │   ├── quality-gate.md
    │   └── lifecycle.md
    │
    ├── /skills                  # 스킬 정의
    │   ├── SKILL-RULES.md
    │   ├── process-analyzer.md
    │   ├── governance-auditor.md
    │   └── ...
    │
    ├── /agents                  # 에이전트 정의
    │   ├── AGENT-RULES.md
    │   ├── process-architect.md
    │   └── ...
    │
    └── /templates               # 문서 템플릿
        ├── TPL-process.md
        └── ...
```

---

## 진입점 파일 예시

### CLAUDE.md (Claude Code용)

```markdown
# Process Forge - Claude Code 설정

이 프로젝트는 운영 프로세스를 설계·검증·문서화하는 메타 시스템입니다.

## 규칙 로드
아래 파일들을 항상 참조하세요:

1. `.agent/MASTER.md` - 마스터 규칙
2. `.agent/rules/` - 프로젝트 규칙
3. `.agent/skills/` - 사용 가능한 스킬
4. `.agent/agents/` - 에이전트 정의

## 작업 시 우선순위
1. MASTER.md의 원칙 준수
2. 문서 생성 시 `.agent/templates/` 참조
3. 프로세스 검증 시 `governance-auditor` 스킬 적용
```

### .cursorrules (Cursor AI용)

```markdown
# Process Forge Rules for Cursor

## 규칙 참조
모든 작업 시 `.agent/` 폴더의 규칙을 따르세요:
- 마스터 규칙: .agent/MASTER.md
- 세부 규칙: .agent/rules/
- 스킬: .agent/skills/
- 에이전트: .agent/agents/

## 핵심 원칙
- Adoption First: 정착 가능성 우선
- Single Owner: 모든 문서는 개인 1명 책임
- Change Control: Approved 문서는 CHG로만 변경

## 문서 작업 시
- 템플릿: .agent/templates/ 참조
- docId 규칙: .agent/rules/document-schema.md 참조
```

---

## Claude Code의 `.agent/` 인식 방식

Claude Code는 **CLAUDE.md에서 명시적으로 참조하면** 하위 폴더를 인식합니다:

```markdown
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
```
---
## Process Forge 프로젝트 폴더 구조

> **목적**: 운영 프로세스를 설계·검증·문서화·개선하는 메타 시스템
> **원칙**: 도구 중립적 규칙 + 도구별 진입점 분리

---

## 전체 구조 개요

```
process-forge/
│
│  ══════════════════════════════════════════════════════════
│  [도구별 진입점 - 각 AI 도구가 읽는 시작점]
│  ══════════════════════════════════════════════════════════
│
├── CLAUDE.md                     # Claude Code 진입점
├── .cursorrules                  # Cursor AI 진입점
├── .windsurfrules                # Windsurf 진입점
│
│  ══════════════════════════════════════════════════════════
│  [규칙/스킬/에이전트 중앙 저장소]
│  ══════════════════════════════════════════════════════════
│
├── .agent/                       # 🎯 핵심: 모든 규칙의 중앙 저장소
│   │
│   ├── MASTER.md                 # 프로젝트 마스터 규칙 (최상위)
│   │
│   ├── rules/                    # 프로젝트 운영 규칙
│   │   ├── ownership.md          #   └─ 소유권/RACI 규칙
│   │   ├── change-control.md     #   └─ 변경 통제 규칙
│   │   ├── document-schema.md    #   └─ 문서 체계/네이밍 규칙
│   │   ├── quality-gate.md       #   └─ 품질/검증 규칙
│   │   └── lifecycle.md          #   └─ 문서 생명주기 규칙
│   │
│   ├── skills/                   # 스킬 정의
│   │   ├── SKILL-RULES.md        #   └─ 스킬 작성 표준
│   │   ├── process-analyzer.md   #   └─ 프로세스 분석 스킬
│   │   ├── governance-auditor.md #   └─ 거버넌스 검증 스킬
│   │   ├── process-designer.md   #   └─ 프로세스 설계 스킬
│   │   └── process-documenter.md #   └─ 문서화 스킬
│   │
│   ├── agents/                   # 에이전트 정의
│   │   ├── AGENT-RULES.md        #   └─ 에이전트 작성 표준
│   │   ├── process-architect.md  #   └─ 프로세스 아키텍트
│   │   ├── documentation.md      #   └─ 문서화 에이전트
│   │   └── improvement.md        #   └─ 개선 에이전트
│   │
│   └── templates/                # 문서 템플릿
│       ├── TEMPLATE-RULES.md     #   └─ 템플릿 작성/사용 규칙
│       ├── TPL-idea.md           #   └─ 아이디어(IDEA) 템플릿
│       ├── TPL-proposal.md       #   └─ 제안서(PROP) 템플릿
│       ├── TPL-report.md         #   └─ 검증리포트(RPT) 템플릿
│       ├── TPL-policy.md         #   └─ 정책(POL) 템플릿
│       ├── TPL-process.md        #   └─ 프로세스(PRC) 템플릿
│       ├── TPL-rule.md           #   └─ 규칙(RUL) 템플릿
│       └── TPL-change.md         #   └─ 변경요청(CHG) 템플릿
│
│  ══════════════════════════════════════════════════════════
│  [실제 산출물 - 프로세스/정책/변경 문서들]
│  ══════════════════════════════════════════════════════════
│
├── docs/                         # 📄 산출물 저장소
│   │
│   ├── ideas/                    # 아이디어 메모 (자유 파일명)
│   │   └── (자유로운 아이디어 문서)
│   │
│   ├── proposals/                # 제안서 (PROP)
│   │   └── PROP-hrops-001.md     #   └─ 예: 인사 운영 제안서
│   │
│   ├── reports/                  # 검증 리포트 (RPT)
│   │   └── RPT-hrops-001.md      #   └─ 예: 전환 검증 리포트
│   │
│   ├── policies/                 # 정책 문서 (POL)
│   │   └── POL-docops-001.md     #   └─ 예: 문서 운영 정책
│   │
│   ├── processes/                # 프로세스 문서 (PRC)
│   │   └── PRC-planops-001.md    #   └─ 예: 운영/진척 관리 프로세스
│   │
│   ├── rules/                    # 개별 규칙 문서 (RUL)
│   │   └── RUL-docid-001.md      #   └─ 예: docId 네이밍 규칙
│   │
│   ├── changes/                  # 변경 요청 문서 (CHG)
│   │   └── CHG-planops-001-001.md#   └─ 예: PRC-planops-001 변경 #1
│   │
│   └── archive/                  # 폐기/아카이브
│       └── (Deprecated 문서들)
│
│  ══════════════════════════════════════════════════════════
│  [프로젝트 메타 정보]
│  ══════════════════════════════════════════════════════════
│
├── README.md                     # 프로젝트 소개
└── CHANGELOG.md                  # 프로젝트 변경 이력
```

---

## 폴더별 상세 설명

### 1. 도구 진입점 파일

| 파일 | 대상 도구 | 역할 |
|------|----------|------|
| `CLAUDE.md` | Claude Code | `.agent/` 참조 지시, 프로젝트 컨텍스트 제공 |
| `.cursorrules` | Cursor AI | `.agent/` 참조 지시, 핵심 원칙 요약 |
| `.windsurfrules` | Windsurf | `.agent/` 참조 지시, 핵심 원칙 요약 |

**왜 분리하는가?**
- 실제 규칙은 `.agent/`에 한 번만 작성
- 도구가 바뀌어도 규칙 재작성 불필요
- 각 도구 특성에 맞게 진입점만 조정

---

### 2. `.agent/` - 규칙 중앙 저장소

#### 2.1 `MASTER.md` - 마스터 규칙

```
역할: 프로젝트 최상위 규칙 (모든 작업의 기준)
내용:
  - 프로젝트 목적과 원칙
  - 용어 정의
  - 문서 유형 정의
  - 하위 규칙 참조 목록
  - 에이전트/스킬 사용 가이드
```

#### 2.2 `rules/` - 프로젝트 운영 규칙

| 파일 | 정의하는 것 |
|------|------------|
| `ownership.md` | Owner 개인 귀속, 대리자 지정, 인수인계, RACI 기준 |
| `change-control.md` | Approved 수정 금지, CHG 프로세스, 영향도 분류, 긴급 경로 |
| `document-schema.md` | docId 네이밍 규칙, 문서 유형(POL/PRC/RUL/CHG), 관계 |
| `quality-gate.md` | 문서 완성 조건, 리뷰 기준, Approved 조건 |
| `lifecycle.md` | 상태 전이(Draft→Approved→Deprecated), 폐기 기준 |

#### 2.3 `skills/` - 스킬 정의

| 파일 | 스킬 역할 |
|------|----------|
| `SKILL-RULES.md` | 스킬 문서 작성 표준 (필수 섹션, 포맷) |
| `process-analyzer.md` | 프로세스 문서의 구조적 문제점 분석 |
| `governance-auditor.md` | RACI, Change Control, Enforcement 완성도 검증 |
| `process-designer.md` | 새 프로세스 설계 시 필수 요소 가이드 |
| `process-documenter.md` | 프로세스를 표준 포맷으로 문서화 |

#### 2.4 `agents/` - 에이전트 정의

| 파일 | 에이전트 역할 |
|------|-------------|
| `AGENT-RULES.md` | 에이전트 문서 작성 표준 |
| `process-architect.md` | 프로세스 분석/설계 총괄 (여러 스킬 조합) |
| `documentation.md` | 문서화/체계 관리 |
| `improvement.md` | 개선 사이클 관리 |

#### 2.5 `templates/` - 문서 템플릿

| 파일 | 용도 |
|------|------|
| `TEMPLATE-RULES.md` | 템플릿 작성/사용 규칙 |
| `TPL-idea.md` | 아이디어 메모 작성 시 사용 |
| `TPL-proposal.md` | 제안서 작성 시 사용 |
| `TPL-report.md` | 검증 리포트 작성 시 사용 |
| `TPL-policy.md` | 정책 문서 작성 시 사용 |
| `TPL-process.md` | 프로세스 문서 작성 시 사용 |
| `TPL-rule.md` | 규칙 문서 작성 시 사용 |
| `TPL-change.md` | 변경 요청 작성 시 사용 |

---

### 3. `docs/` - 산출물 저장소

실제로 작성된 문서들이 저장되는 곳입니다.

#### 3.1 문서 생산 파이프라인

```
ideas/ → proposals/ → [검증 RPT] → Draft → Review → Approved
(자유)    (구조화)      (reports/)
```

| 단계 | 폴더 | 설명 |
|------|------|------|
| 아이디어 | `docs/ideas/` | 자유로운 메모 (파일명 자유) |
| 제안서 | `docs/proposals/` | 아이디어 통합/구조화 (PROP-*) |
| 검증 | `docs/reports/` | 표준문서 전환 적합성 검증 (RPT-*) |
| 표준문서 | `docs/{type}/` | POL/PRC/RUL로 전환 |

#### 3.2 폴더별 문서 유형

| 폴더 | 문서 유형 | docId 접두사 | 예시 |
|------|----------|-------------|------|
| `ideas/` | 아이디어 | 자유 파일명 | `인사개선-아이디어.md` |
| `proposals/` | 제안서 | `PROP-` | `PROP-hrops-001.md` |
| `reports/` | 검증 리포트 | `RPT-` | `RPT-hrops-001.md` |
| `policies/` | 정책 (Why) | `POL-` | `POL-docops-001.md` |
| `processes/` | 프로세스 (How) | `PRC-` | `PRC-planops-001.md` |
| `rules/` | 개별 규칙 | `RUL-` | `RUL-docid-001.md` |
| `changes/` | 변경 요청 | `CHG-` | `CHG-planops-001-001.md` |
| `archive/` | 폐기 문서 | - | Deprecated 문서 이동 |

---

## 문서 흐름 다이어그램

```
┌─────────────────────────────────────────────────────────────┐
│                    작업 요청 발생                             │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│  AI 도구가 진입점 파일 읽음                                   │
│  (CLAUDE.md / .cursorrules / .windsurfrules)                │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│  .agent/MASTER.md 로드 → 프로젝트 원칙 파악                   │
└─────────────────────┬───────────────────────────────────────┘
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
     ┌─────────┐ ┌─────────┐ ┌─────────┐
     │ rules/  │ │ skills/ │ │templates│
     │ 참조    │ │ 적용    │ │ 사용    │
     └────┬────┘ └────┬────┘ └────┬────┘
          │           │           │
          └───────────┼───────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│  docs/ 에 산출물 생성                                        │
│  (ideas/ | proposals/ | reports/ | policies/ | ...)         │
└─────────────────────────────────────────────────────────────┘
```

---

## 작성 우선순위 로드맵

### Phase 1: 부트스트랩 (즉시 필요)

```
✅ 먼저 만들 것:
├── CLAUDE.md                    # 진입점
├── .cursorrules                 # 진입점
└── .agent/
    ├── MASTER.md                # 프로젝트 정의
    └── rules/
        ├── document-schema.md   # docId/문서유형 확정
        ├── ownership.md         # Owner 규칙 확정
        └── change-control.md    # 변경 통제 확정
```

### Phase 2: 템플릿 (문서 생산 기반)

```
├── .agent/templates/
│   ├── TEMPLATE-RULES.md
│   ├── TPL-process.md
│   └── TPL-change.md
```

### Phase 3: 스킬/에이전트 (자동화 기반)

```
├── .agent/skills/
│   ├── SKILL-RULES.md
│   ├── process-analyzer.md
│   └── governance-auditor.md
├── .agent/agents/
│   ├── AGENT-RULES.md
│   └── process-architect.md
```

### Phase 4: 운영 안정화

```
├── .agent/rules/
│   ├── quality-gate.md
│   └── lifecycle.md
```

---
