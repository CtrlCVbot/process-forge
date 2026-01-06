# Process Forge - Claude Code 설정

이 프로젝트는 운영 프로세스를 설계·검증·문서화하는 메타 시스템입니다.

---

## 프로젝트 컨텍스트

이 프로젝트의 규칙은 `.agent/` 폴더에 정의되어 있습니다.

---

## 필수 참조 파일

작업 시 아래 파일들을 항상 참조하세요:

1. `.agent/MASTER.md` - 마스터 규칙 (최상위 원칙)
2. `.agent/rules/document-schema.md` - 문서 체계/네이밍 규칙
3. `.agent/rules/ownership.md` - 소유권/RACI 규칙
4. `.agent/rules/change-control.md` - 변경 통제 규칙

---

## 작업 시 우선순위

1. **MASTER.md의 원칙 준수** - 모든 작업의 기준
2. **문서 생성 시** `.agent/templates/` 참조
3. **프로세스 검증 시** `.agent/skills/governance-auditor.md` 적용
4. **프로세스 분석 시** `.agent/skills/process-analyzer.md` 적용

---

## 핵심 원칙

| 원칙 | 설명 |
|------|------|
| **Adoption First** | 정착 가능성 우선 - 실제 사용될 수 있는 문서 작성 |
| **Single Owner** | 모든 문서는 개인 1명이 책임 |
| **Change Control** | Approved 문서는 CHG(변경요청)로만 변경 |

---

## 폴더 구조

```
.agent/
├── MASTER.md           # 마스터 규칙
├── rules/              # 프로젝트 운영 규칙
├── skills/             # 스킬 정의
├── agents/             # 에이전트 정의
└── templates/          # 문서 템플릿

docs/
├── policies/           # 정책 문서 (POL-*)
├── processes/          # 프로세스 문서 (PRC-*)
├── rules/              # 규칙 문서 (RUL-*)
├── changes/            # 변경 요청 (CHG-*)
└── archive/            # 폐기 문서
```
