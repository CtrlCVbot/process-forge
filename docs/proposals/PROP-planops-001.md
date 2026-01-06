---
docId: PROP-planops-001
title: 운영/진척 관리 프로세스 도입 제안 (Active Cycle 기반)
owner: 박재형
deputy: (부책임자)
status: Proposal
version: 1.0
created: 2026-01-06
updated: 2026-01-06
related_ideas:
  - docs/ideas/2025-01/운영-관리-구조/운영-진척-관리-노션구조.md
  - docs/ideas/2025-01/운영-관리-구조/운영-진척-관리-개선안.md
  - docs/ideas/2025-01/운영-관리-구조/cycles-db-info.md
  - docs/ideas/2025-01/운영-관리-구조/Initiatives-db-info.md
  - docs/ideas/2025-01/운영-관리-구조/Phases-db-info.md
  - docs/ideas/2025-01/운영-관리-구조/ops-tasks-info.md
---

# 운영/진척 관리 프로세스 도입 제안 (Active Cycle 기반)

## 1. 배경 및 목적

### 1.1 배경

현재 팀의 분기/월 계획 운영에서 다음 문제가 발생하고 있다:

| 문제 | 현상 |
|------|------|
| **기획-개발 일정 분리** | 기획 일정과 개발 일정이 별도로 관리되어 전체 그림 파악 어려움 |
| **기간 전환 시 혼선** | 분기→월 전환 시 뷰/필터가 깨져 수동 수정 필요 |
| **품질 관문 부재** | Spec 완료 여부 확인 없이 개발 착수 → 잦은 스펙 변경 |
| **SSOT 분산** | Notion과 Obsidian에 동일 내용 중복 → 불일치 발생 |
| **책임 불명확** | 공동 Owner로 인한 책임 회피 |

### 1.2 목적

**Notion=운영/데이터**, **Obsidian=SSOT/버전** 원칙 아래에서:

1. 기획(Spec) + 개발(Dev) + QA + Release를 **한 장(마스터 플랜)**으로 운영
2. **Active Cycle 기반** 필터로 기간 전환 시에도 뷰가 깨지지 않는 구조 확보
3. **Gate 기반 품질 관문**으로 단계별 품질 확보
4. **Single Owner 원칙**으로 책임 명확화

---

## 2. 제안 내용

### 2.1 핵심 원칙

| 원칙 | 설명 |
|------|------|
| **SSOT 분리** | Notion=레코드(상태/담당/기한), Obsidian=문서 본문 |
| **docId 연결** | 모든 레코드는 docId로 SSOT 연결, docId 불변 |
| **Single Owner** | 모든 Phase/Task는 1명의 Owner (공동 Owner 금지) |
| **Active Cycle 필터** | 모든 뷰는 `Cycle Status = Active` 필터 사용 |
| **Gate 기반 품질** | 다음 단계 진입 전 Gate 조건 충족 필수 |

### 2.2 Notion DB 구조 (4개 DB)

```
Cycles (1) ── (N) Initiatives (1) ── (N) Phases (1) ── (N) Ops Tasks
```

| DB | 역할 | 핵심 속성 |
|----|------|----------|
| **Cycles** | 기간 컨테이너 | Name, Start, End, Status(Planned/Active/Done), Owner |
| **Initiatives** | 큰 계획 단위 | Cycle, Status, Priority, Outcome, PM Owner, Tech Owner, docId |
| **Phases** | 일정의 실체 | Initiative, Type(Spec/Dev/QA/Release), Start, End, Gate, Owner |
| **Ops Tasks** | 주간 실행 | Phase, Status, Owner, Due, DoD |

### 2.3 표준 뷰 (모두 Active Cycle 필터)

| 뷰 | DB | 목적 |
|----|-----|------|
| **마스터 플랜** | Phases (Timeline) | 기획+개발+QA+Release 한 화면 |
| Spec 파이프라인 | Phases (Board) | 기획 진행 현황 |
| Dev 타임라인 | Phases (Timeline) | 개발 일정 |
| 주간 실행 | Ops Tasks (Kanban) | 이번 주 집중 |
| 포트폴리오 | Initiatives (Table) | 전체 계획 목록 |

### 2.4 Gate 운영

#### Spec Gate (Ready for Dev)
- [ ] PRD docId 존재 (SSOT 링크)
- [ ] 상태/권한/예외 시나리오 포함
- [ ] 데이터 영향 명시
- [ ] QA TC/QAP docId 링크 존재
- [ ] Tech Owner 리뷰 완료

#### Release Gate
- [ ] QA 완료 (핵심 TC 통과)
- [ ] 모니터링/로그 포인트 존재
- [ ] 롤백 플랜 존재

### 2.5 운영 루프

| 이벤트 | 주기 | 내용 |
|--------|------|------|
| 주간 리뷰 | 매주 금 30분 | Blocked 확인, Gate 점검, Top3 확정 |
| Re-plan | 월 1회 (첫째 주 금) | 일정 재배치, Drop/Defer 결정 |
| Cycle Kickoff | 기간 시작 시 | Active 전환, Initiative/Phase 생성 |
| Cycle Close | 기간 종료 시 | 상태 정리, 회고, Done 전환 |

---

## 3. 기대 효과

### 3.1 정량적 효과

| 지표 | 현재 (추정) | 목표 |
|------|------------|------|
| 기획-개발 일정 동기화율 | 60% | 90% |
| 기간 전환 시 수동 수정 | 매번 발생 | 0 |
| Spec 변경으로 인한 재작업 | 잦음 | 50% 감소 |

### 3.2 정성적 효과

- **가시성 향상**: 마스터 플랜 한 장으로 전체 상황 파악
- **계획 신뢰도 향상**: Gate 기반 품질 확보, Re-plan 슬롯 고정
- **책임 명확화**: Single Owner로 의사결정 속도 향상
- **온보딩 용이**: 표준화된 뷰/프로세스로 신규 팀원 적응 시간 단축

---

## 4. 예상 영향

### 4.1 기존 프로세스 영향

| 영역 | 영향 | 대응 |
|------|------|------|
| 기존 Notion DB | 새 DB 구조로 마이그레이션 필요 | 파일럿 후 점진 전환 |
| 기존 뷰/필터 | Active Cycle 필터로 변경 | 기존 뷰 병행 운영 후 폐기 |
| 문서 운영 | docId 연결 강제 | document-schema.md 규칙 적용 |

### 4.2 역할별 영향

| 역할 | 변화 |
|------|------|
| Cycle Owner | Active Cycle 관리, Re-plan 승인 책임 |
| PM/PO | Spec Gate 충족 책임, docId 필수 |
| Tech Owner | Gate Ready 전 Dev 착수 금지 |
| QA Owner | Release Gate 조건 정의/검증 |

### 4.3 관련 규칙 연동

| 규칙 | 연동 내용 |
|------|----------|
| `document-schema.md` | docId 네이밍 규칙 준수 |
| `change-control.md` | Approved 문서 CHG 프로세스 적용 |
| `ownership.md` | Single Owner 원칙 적용 |

---

## 5. 목표 문서 유형

- [ ] POL (정책)
- [x] PRC (프로세스) - `PRC-planops-001`
- [ ] RUL (규칙)

### 5.1 승격 시 생성 문서

| docId | 제목 | 유형 |
|-------|------|------|
| `PRC-planops-001` | 운영/진척 관리 프로세스 | 프로세스 |
| `DBS-notion-cycles-001` | Notion DB 템플릿 - Cycles | DB 스펙 |
| `DBS-notion-initiatives-001` | Notion DB 템플릿 - Initiatives | DB 스펙 |
| `DBS-notion-phases-001` | Notion DB 템플릿 - Phases | DB 스펙 |
| `DBS-notion-opstasks-001` | Notion DB 템플릿 - Ops Tasks | DB 스펙 |

---

## 6. 도입 계획

### 6.1 단계별 계획

| 단계 | 내용 | 산출물 |
|------|------|--------|
| **1. 승인** | 제안서 리뷰 및 승인 | 승인된 PROP |
| **2. DB 구축** | Notion DB 4개 생성, 속성/뷰 설정 | DB 템플릿 |
| **3. 파일럿** | 1 Cycle 시범 운영 | 파일럿 결과 리포트 |
| **4. 피드백** | 문제점 수집 및 개선 | 개선안 |
| **5. 정식 전환** | PRC-planops-001 Approved | 정식 프로세스 |

### 6.2 리스크 및 대응

| 리스크 | 대응 |
|--------|------|
| 기존 데이터 마이그레이션 복잡 | 신규 Cycle부터 적용, 기존은 Archive |
| Gate로 인한 속도 저하 우려 | 최소 Gate 조건만 적용, 점진 강화 |
| 팀 저항 | 파일럿으로 효과 검증 후 확대 |

---

## 7. 관련 문서

| 문서 | 관계 |
|------|------|
| `.agent/rules/document-schema.md` | docId 규칙 참조 |
| `.agent/rules/change-control.md` | 변경통제 규칙 참조 |
| `.agent/rules/ownership.md` | Owner 규칙 참조 |
| `docs/ideas/2025-01/운영-관리-구조/운영-진척-관리-통합안.md` | 상세 내용 |

---

## 8. 승인 요청

본 제안서의 검토 및 승인을 요청합니다.

| 항목 | 내용 |
|------|------|
| 제안자 | (작성자) |
| 승인 요청일 | 2026-01-06 |
| 승인권자 | (승인권자) |

### 승인란

- [ ] 승인
- [ ] 조건부 승인 (조건: _____________)
- [ ] 반려 (사유: _____________)

승인일: _______________
승인자: _______________

---

## 변경 이력

| 버전 | 날짜 | 작성자 | 변경 내용 |
|------|------|--------|----------|
| 1.0 | 2026-01-06 | (작성자) | 최초 작성 |
