---

## docId: DBS-notion-initiatives-001
type: DBS
title: Notion DB 템플릿 - Initiatives (이니셔티브 / 분기·월 큰 계획 단위)
status: Draft
rev: 0.1
owner: (작성자)
reviewers: [(버디/리드)]
effectiveDate: 2026-01-02
links:
notion_db: (Initiatives DB URL)
related_docs:
- PRC-planops-001
- DBS-notion-cycles-001
- POL-docops-001

# 1. 개요

Initiatives DB는 **사이클(분기/월) 안에서 “무엇을 달성할 것인가”**를 정의하는 상위 계획 단위다.

Ops Tasks(주간 할 일)와 Phases(스펙/개발/QA/릴리즈 일정)가 흔들리지 않도록, Initiative는 아래를 고정한다.

- **범위(Scope)**: 이번 사이클에 어디까지 할지
- **우선순위(Priority)**: 무엇이 먼저인지
- **책임(Owner)**: 누가 판단하고 조율하는지
- **성과(Outcome)**: 무엇이 끝났다고 말할 수 있는지

> Notion의 Initiative 페이지는 “허브(Hub)”다.
> 
> 
> 상세 본문(기획/정책/프로세스/데이터)은 Obsidian(SSOT)로만 관리하고, Notion에는 docId+링크만 둔다.
> 

---

# 2. 데이터 모델에서의 위치

## 2.1 관계(권장)

- Cycles (1) ── (N) Initiatives
- Initiatives (1) ── (N) Phases
- Phases (1) ── (N) Ops Tasks

## 2.2 Active Cycle 기반 운영을 위한 핵심

- Initiatives는 `사이클` Relation을 반드시 가진다.
- 표준 뷰 필터는 **사이클명**이 아니라 `사이클 상태(rollup) = 운영중`을 사용한다.

---

# 3. 레코드 규칙(네이밍/ID)

## 3.1 이니셔티브명(Title) 규칙

- 결과가 드러나게 작성한다(“논의/개선” 금지)
    - ✅ “정산 수동조정 120→40 감소”
    - ✅ “배차 확정 리드타임 15분→7분”
    - ❌ “정산 고도화”
    - ❌ “배차 화면 개선 논의”

## 3.2 initiativeId(권장)

- 예: `INI-2026Q1-001`, `INI-202602-003`
- 외부 연동(Jira/GitHub)이나 리포팅을 고려하면 초기에 두는 편이 안정적

---

# 4. 속성 스키마(한글화 표준 포함)

아래는 “최소 운영 세트(필수) → 확장 세트(권장)”로 구성된다.

## 4.1 필수 속성(최소 운영 세트)

| 한글 속성명 | 영문 권장키(선택) | 타입 | 필수 | 예시 | 설명 |
| --- | --- | --- | --- | --- | --- |
| 이니셔티브명 | Name | Title | ✅ | 정산 예외 자동처리 커버율 70% | Initiative 제목 |
| 사이클 | Cycle | Relation → Cycles | ✅ | 2026 Q1 | 소속 기간 |
| 이니셔티브 상태 | Status | Select | ✅ | 확정 | 의사결정 단계 |
| 우선순위 | Priority | Select/Number | ✅ | P0 | 상대 우선순위 |
| 목표/성과(Outcome) | Outcome | Text | ✅ | 수동조정 120→40 | 측정 가능한 결과 |
| 기획 책임자 | PM Owner | Person | ✅ | 홍길동 | 범위/정책/일정 조율 |
| 기술 책임자 | Tech Owner | Person | ✅ | 김개발 | 기술 판단/리소스 조율 |
| 메인 docId(PRD) | Main docId | Text | ✅ | PRD-mm-0012 | SSOT 진입점 |
| 사이클 상태(롤업) | Cycle Status | Rollup | ✅ | 운영중 | 표준 뷰 필터용 |

### 이니셔티브 상태(Select 옵션) - 한글 표준

- `제안됨` (Proposed): 후보
- `확정` (Committed): 이번 사이클에 반드시 수행
- `진행중` (In Progress): Spec/Dev가 실제 진행
- `릴리즈됨` (Released): 사용자/운영에 실제 반영됨
- `제외` (Dropped): 이번 사이클에서 수행하지 않음

> 주의: Phase의 진행상태(진행/완료)와 Initiative 상태를 섞지 않는다.
> 
> 
> “릴리즈됨”은 **실제 배포/반영**이 된 경우에만 사용한다.
> 

### 우선순위(Select 옵션) - 권장

- `P0`(최상): 반드시
- `P1`(상): 가능하면
- `P2`(중): 여력 시
- `P3`(하): 옵션/실험

---

## 4.2 권장 속성(운영 고도화 세트)

| 한글 속성명 | 타입 | 예시 | 설명 |
| --- | --- | --- | --- |
| 범위(이번/다음) | Select | 이번 | 범위 컷 라인 |
| 리스크/의존성 | Text | 현장 협조 필요 | 병목 요인 |
| 영향 범위 | Multi-select | 정책/데이터/상태 | 설계 영향 표시 |
| 기대효과(정성) | Text | 분쟁 감소 | 정량 외 효과 |
| KPI/지표 링크 | URL | (대시보드) | 성공 기준 추적 |
| 이해관계자 | People/Multi-person | CS, 정산팀 | 커뮤니케이션 대상 |
| 관련 문서(docIds) | Text/Multi-select | POL..., DIC... | 연결 문서 목록 |
| Phase 목록 | Relation → Phases |  | 일정 자동 연결 |
| Ops Task 목록 | Relation → Ops Tasks(옵션) |  | 실행 티켓 추적(선택) |

---

# 5. 필수 Rollup/Relation 설정(표준 뷰를 살리는 핵심)

## 5.1 사이클 상태(롤업)

- 속성명: `사이클 상태(롤업)`
- 타입: Rollup
- 경로: `사이클` → `사이클 상태`
- 집계: Show original (또는 단일 값)

> 이 속성이 있어야 “Active Cycle 기반” 필터가 Initiatives에서도 동일하게 작동한다.
> 

---

# 6. 표준 뷰(권장 세트)

모든 표준 뷰는 아래 공통 필터를 사용한다.

- 공통 필터: `사이클 상태(롤업) = 운영중`

## 6.1 이니셔티브 포트폴리오(운영중)

- View: Table
- Filter: `사이클 상태(롤업) = 운영중`
- Sort: `우선순위(P0→P3)`, `이니셔티브 상태`, (선택) Start/Impact
- 표시 컬럼(권장):
    - 우선순위, 상태, 기획 책임자, 기술 책임자, Outcome, 리스크/의존성, 메인 docId(PRD)

## 6.2 상태 보드(의사결정 흐름)

- View: Board (Group by `이니셔티브 상태`)
- Filter: `사이클 상태(롤업) = 운영중`
- 목적: Proposed→Committed→In Progress→Released 흐름 관리

## 6.3 Committed vs Stretch(선택: 범위 속성 사용 시)

- View: Table
- Filter: `사이클 상태(롤업) = 운영중`
- Group: `범위(이번/다음)` 또는 `Committed/Stretch` 속성
- 목적: “확정”과 “옵션” 분리

## 6.4 리스크 레지스터(운영중)

- View: Table
- Filter:
    - `사이클 상태(롤업) = 운영중`
    - `리스크/의존성 is not empty`
- 목적: 병목만 모아 보기

---

# 7. 페이지 템플릿(레코드 본문 템플릿)

Initiative 페이지는 “본문 작성”이 아니라 “링크/체크/결정의 허브”로 고정한다.

## 7.1 Initiative Hub 템플릿(권장 본문)

### 섹션 구성

- 🎯 목표/성과(Outcome)
- 📌 범위(이번/다음) / 제외 범위(비목표)
- 🔗 SSOT 링크(Obsidian)
    - PRD(메인 docId)
    - Policy/State/Data/QA 관련 docId 링크
- 📅 일정(Phases 뷰 임베드)
    - Spec / Dev / QA / Release
- ✅ Gate 체크(요약)
    - Ready for Dev 조건 충족 여부(체크)
- ⚠️ 리스크/의존성 요약
- 🧾 결정 로그(ADR 링크 모음)
- 📣 커뮤니케이션(공지/공유 링크)

### 금지 규칙

- Notion에 PRD/정책/프로세스 본문 복제 금지
- 본문은 Obsidian SSOT로만 유지

---

# 8. 운영 규칙(가드레일)

## 8.1 Initiative 크기 규칙

- Initiative 1개는 최소 아래가 나와야 정상:
    - Phase가 최소 2개 이상(Spec + Dev)
- “기능 하나=Initiative”로 너무 잘게 쪼개면 Task처럼 변질된다.
- 반대로 “정산 고도화”처럼 너무 크면 끝이 없다 → Outcome 1개로 자른다.

## 8.2 상태 사용 규칙

- `릴리즈됨`은 **사용자/운영에 실제 반영**된 경우만 사용한다.
- Spec이 완료됐다고 Released로 바꾸지 않는다.
- Drop/Defer는 Initiative 단위로 결정하고, Re-plan 슬롯에서만 변경한다.

## 8.3 우선순위 상속

- Phase/Ops Tasks는 가능한 한 Initiative 우선순위를 참고하도록 운영한다.
- 밀릴 때는 “Priority 낮은 Initiative부터 멈춘다”가 가능해야 한다.

## 8.4 SSOT 연결 규칙

- `메인 docId(PRD)`는 필수.
- 관련 문서 docId(Policy/State/Data/QA)가 없으면:
    - 진행은 가능하되, Spec Gate(Ready for Dev)에서 통과하지 못하도록 운영(권장)

---

# 9. 구축 순서(세팅 체크리스트)

1. Initiatives DB 생성
2. 속성 추가(필수 속성 8개)
    - 이니셔티브명, 사이클, 상태, 우선순위, Outcome, 기획책임자, 기술책임자, 메인 docId
3. `사이클 상태(롤업)` 설정(사이클→사이클 상태)
4. 상태/우선순위 옵션 값 고정
5. 표준 뷰 4개 생성(포트폴리오/상태보드/범위분리/리스크)
6. Initiative Hub 템플릿 생성(본문 규칙 포함)
7. (선택) initiativeId 및 확장 속성 추가

---

# 10. 다음 문서(연결되는 DB)

- DBS-notion-phases-001 (Phases DB 템플릿) ← 마스터 플랜의 본체
- DBS-notion-opstasks-001 (Ops Tasks DB 템플릿)
- DBS-notion-docsregistry-001 (Docs Registry DB 템플릿, 거버넌스)

---