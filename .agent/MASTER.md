# MASTER.md - Process Forge 마스터 규칙

> **버전**: 1.0
> **상태**: Draft
> **최종 수정**: 2026-01-06

---

## 1. 프로젝트 목적

Process Forge는 **운영 프로세스를 설계·검증·문서화·개선하는 메타 시스템**입니다.

### 목표
- 조직의 운영 프로세스를 체계적으로 문서화
- 프로세스 간 일관성과 품질 보장
- 변경 이력 추적 및 통제
- AI 도구를 활용한 프로세스 분석/개선 자동화

---

## 2. 핵심 원칙

### 2.1 Adoption First (정착 가능성 우선)
- 완벽한 문서보다 **실제 사용될 수 있는 문서** 우선
- 복잡한 프로세스보다 **따르기 쉬운 프로세스** 설계
- 이론보다 **실무 적용성** 중시

### 2.2 Single Owner (단일 소유자)
- 모든 문서는 **개인 1명**이 책임
- 팀/부서가 아닌 **특정인에게 귀속**
- Owner 부재 시 반드시 **대리자(Deputy)** 지정

### 2.3 Change Control (변경 통제)
- **Approved 상태 문서는 직접 수정 금지**
- 변경 시 반드시 **CHG(변경요청) 문서 생성**
- 모든 변경은 **영향도 분석** 후 승인

---

## 3. 용어 정의

| 용어 | 정의 |
|------|------|
| **프로세스 (Process)** | 특정 목표를 달성하기 위한 일련의 활동 순서 |
| **정책 (Policy)** | 조직의 의사결정 기준과 방향 (Why) |
| **규칙 (Rule)** | 특정 상황에서 따라야 할 구체적 지침 (What) |
| **변경요청 (CHG)** | Approved 문서 수정을 위한 공식 요청 |
| **Owner** | 문서의 최종 책임자 (개인 1명) |
| **Deputy** | Owner 부재 시 대리 책임자 |

---

## 4. 문서 유형

| 접두사 | 유형 | 설명 | 예시 |
|--------|------|------|------|
| **POL** | 정책 | 의사결정 기준, 방향성 (Why) | `POL-docops-001` |
| **PRC** | 프로세스 | 업무 수행 절차 (How) | `PRC-planops-001` |
| **RUL** | 규칙 | 구체적 지침, 기준 (What) | `RUL-docid-001` |
| **CHG** | 변경요청 | 기존 문서 수정 요청 | `CHG-planops-001-001` |

---

## 5. 문서 상태 (Lifecycle)

```
Draft → Review → Approved → Deprecated
  ↑                           ↓
  └───────── Archive ←────────┘
```

| 상태 | 설명 |
|------|------|
| **Draft** | 작성 중, 자유롭게 수정 가능 |
| **Review** | 검토 중, Owner 승인 대기 |
| **Approved** | 승인됨, CHG 없이 수정 불가 |
| **Deprecated** | 폐기 예정, 더 이상 유효하지 않음 |
| **Archive** | 보관, 참조용으로만 유지 |

---

## 6. 하위 규칙 참조

### 필수 규칙 (항상 적용)
| 파일 | 내용 |
|------|------|
| `rules/document-schema.md` | docId 네이밍, 문서 유형 체계 |
| `rules/ownership.md` | 소유권, RACI, 인수인계 |
| `rules/change-control.md` | 변경 통제 프로세스 |

### 선택 규칙 (상황별 적용)
| 파일 | 적용 시점 |
|------|----------|
| `rules/quality-gate.md` | 문서 승인 전 품질 검증 시 |
| `rules/lifecycle.md` | 문서 상태 전환 시 |

---

## 7. 스킬/에이전트 사용

### 스킬 (단일 작업)
| 스킬 | 사용 시점 |
|------|----------|
| `skills/process-analyzer.md` | 프로세스 문서 분석 요청 시 |
| `skills/governance-auditor.md` | RACI/변경통제 검증 요청 시 |
| `skills/process-designer.md` | 새 프로세스 설계 요청 시 |
| `skills/process-documenter.md` | 프로세스 문서화 요청 시 |

### 에이전트 (복합 작업)
| 에이전트 | 사용 시점 |
|----------|----------|
| `agents/process-architect.md` | 프로세스 분석/설계 총괄 |
| `agents/documentation.md` | 문서화/체계 관리 |
| `agents/improvement.md` | 개선 사이클 관리 |

---

## 8. 산출물 저장 위치

```
docs/
├── policies/      # POL-* 문서
├── processes/     # PRC-* 문서
├── rules/         # RUL-* 문서
├── changes/       # CHG-* 문서
└── archive/       # 폐기/보관 문서
```

---

## 9. AI 도구 작업 지침

1. **문서 생성 전** - 반드시 `templates/` 폴더의 해당 템플릿 확인
2. **문서 수정 전** - 문서 상태 확인 (Approved면 CHG 생성)
3. **프로세스 분석** - `process-analyzer` 스킬 규칙 적용
4. **검증 요청** - `governance-auditor` 스킬로 완성도 확인
5. **docId 부여** - `document-schema.md` 규칙 준수
