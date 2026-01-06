# TEMPLATE-RULES.md - 템플릿 작성/사용 규칙

> **상위 규칙**: MASTER.md
> **버전**: 1.0
> **상태**: Draft

---

## 1. 개요

이 규칙은 Process Forge 프로젝트의 **문서 템플릿 작성 표준**과 **사용 규칙**을 정의합니다.

---

## 2. 템플릿 파일 명명 규칙

### 2.1 형식

```
TPL-{문서유형}.md
```

### 2.2 현재 템플릿 목록

| 파일명 | 대상 문서 유형 | 저장 위치 |
|--------|---------------|----------|
| `TPL-idea.md` | IDEA (아이디어) | `docs/ideas/` |
| `TPL-proposal.md` | PROP (제안서) | `docs/proposals/` |
| `TPL-report.md` | RPT (검증 리포트) | `docs/reports/` |
| `TPL-policy.md` | POL (정책) | `docs/policies/` |
| `TPL-process.md` | PRC (프로세스) | `docs/processes/` |
| `TPL-rule.md` | RUL (규칙) | `docs/rules/` |
| `TPL-change.md` | CHG (변경요청) | `docs/changes/` |

---

## 3. 플레이스홀더 표기법

### 3.1 기본 형식

```
{{변수명}}
```

### 3.2 표준 플레이스홀더

| 플레이스홀더 | 설명 | 예시 값 |
|-------------|------|--------|
| `{{docId}}` | 문서 식별자 | PRC-planops-001 |
| `{{title}}` | 문서 제목 | 계획 운영 프로세스 |
| `{{owner}}` | Owner 이름 | 홍길동 |
| `{{deputy}}` | Deputy 이름 | 김철수 |
| `{{domain}}` | 업무 도메인 | planops |
| `{{YYYY-MM-DD}}` | 날짜 | 2026-01-06 |

### 3.3 작성 시 주의사항

- 플레이스홀더는 **반드시 교체**되어야 함
- 교체되지 않은 `{{}}` 형태가 남아있으면 Draft 상태 유지
- 선택 항목은 해당 없을 시 행/섹션 삭제

---

## 4. 섹션 구분

### 4.1 필수 섹션 (Required)

모든 문서에 반드시 포함되어야 하는 섹션:

| 섹션 | 설명 |
|------|------|
| **메타데이터** | YAML Front Matter (docId, owner, status 등) |
| **목적** | 문서의 존재 이유 |
| **적용 범위** | 적용 대상/조직/상황 |
| **관련 문서** | 상위/하위/참조 문서 |
| **변경 이력** | 버전별 변경 내역 |

### 4.2 선택 섹션 (Optional)

문서 유형에 따라 선택적으로 포함:

| 섹션 | 적용 문서 |
|------|----------|
| 배경/동기 | IDEA |
| 제안 내용 | PROP |
| 검증 항목/전환 판정 | RPT |
| 정책 내용 | POL |
| 프로세스 단계 | PRC |
| 규칙 내용 | RUL |
| 영향도 분석 | CHG |

### 4.3 템플릿 내 표기

```markdown
## 필수: 목적
[이 섹션은 반드시 작성]

## 선택: 참고 사항
[해당 없으면 이 섹션 삭제]
```

---

## 5. 메타데이터 표준

### 5.1 YAML Front Matter 형식

```yaml
---
docId: {{docId}}
title: {{title}}
owner: {{owner}}
deputy: {{deputy}}
status: Draft
version: 1.0
created: {{YYYY-MM-DD}}
updated: {{YYYY-MM-DD}}
---
```

### 5.2 필수 메타데이터 항목

| 항목 | 필수 | 설명 |
|------|------|------|
| `docId` | O | 문서 식별자 |
| `title` | O | 문서 제목 |
| `owner` | O | Owner 이름 |
| `deputy` | X | Deputy 이름 |
| `status` | O | 문서 상태 |
| `version` | O | 버전 번호 |
| `created` | O | 최초 작성일 |
| `updated` | O | 최종 수정일 |

### 5.3 CHG 전용 메타데이터

```yaml
target: {{대상 문서 docId}}
impact: High | Medium | Low
```

---

## 6. 템플릿 사용 절차

### 6.1 새 문서 작성 시

```
1. 해당 유형의 템플릿 파일 복사
2. 파일명을 docId.md로 변경
3. 메타데이터 플레이스홀더 교체
4. 각 섹션 내용 작성
5. 선택 섹션 중 불필요한 항목 삭제
6. 플레이스홀더 잔존 여부 확인
```

### 6.2 저장 위치

| 문서 유형 | 저장 위치 | 파일명 규칙 |
|----------|----------|------------|
| IDEA | `docs/ideas/` | 자유 파일명 |
| PROP | `docs/proposals/` | `PROP-{domain}-{seq}.md` |
| RPT | `docs/reports/` | `RPT-{domain}-{seq}.md` |
| POL | `docs/policies/` | `POL-{domain}-{seq}.md` |
| PRC | `docs/processes/` | `PRC-{domain}-{seq}.md` |
| RUL | `docs/rules/` | `RUL-{domain}-{seq}.md` |
| CHG | `docs/changes/` | `CHG-{domain}-{seq}-{변경seq}.md` |

---

## 7. 템플릿 수정 규칙

### 7.1 템플릿 수정 권한

- `.agent/templates/` 내 파일은 **프로젝트 관리자**만 수정
- 수정 시 모든 기존 문서와의 호환성 검토 필요

### 7.2 템플릿 버전 관리

- 템플릿 수정 시 이 파일(TEMPLATE-RULES.md)에 기록
- 기존 문서는 소급 적용하지 않음 (신규 문서부터 적용)

---

## 8. AI 도구 사용 지침

### 8.1 문서 생성 요청 시

```
1. 문서 유형 확인 (IDEA/PROP/RPT/POL/PRC/RUL/CHG)
2. 해당 템플릿 로드
3. 사용자에게 필수 정보 요청:
   - IDEA: 자유 (파일명, 내용)
   - PROP/RPT/POL/PRC/RUL/CHG: docId (domain, seq), Owner, 핵심 내용
4. 템플릿 기반 문서 생성
5. 플레이스홀더 완전 교체 확인 (IDEA 제외)
```

### 8.2 템플릿 기반 검증

생성된 문서가 템플릿 필수 섹션을 모두 포함하는지 확인:

```
- [ ] 메타데이터 완전성
- [ ] 목적 섹션 존재
- [ ] 적용 범위 섹션 존재
- [ ] 관련 문서 섹션 존재
- [ ] 변경 이력 섹션 존재
- [ ] 플레이스홀더 잔존 없음
```
