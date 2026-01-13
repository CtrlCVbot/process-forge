# Notion 대시보드 스펙 - Operations Hub

---
docId: DBS-notion-dashboard-001
title: Notion 대시보드 스펙 - Operations Hub
owner: 박재형
status: Draft
version: 1.0
created: 2026-01-07
updated: 2026-01-07
parent: PRC-planops-001
---

## 1. 개요

### 1.1 목적

Operations Hub는 PRC-planops-001 (운영/진척 관리 프로세스)를 Notion 대시보드로 구현하여 세 가지 목적을 충족한다:

| 목적 | 대상 사용자 | 핵심 기능 |
|------|------------|----------|
| **실시간 현황 파악** | 전체 팀원 | Phase/Task 진행 상황 즉시 확인 |
| **경영진 보고** | 경영진, 이해관계자 | KPI와 진척률 중심 요약 뷰 |
| **운영자 작업** | Cycle Owner, PM | 일상 운영 및 관리 도구 |

### 1.2 핵심 원칙

- **Active Cycle 중심**: 모든 뷰는 `Cycle Status = Active` 필터 기반
- **Linked Database**: 단일 원본 DB를 여러 뷰로 표현 (데이터 중복 방지)
- **docId 연결**: Notion = 상태/담당, Obsidian = SSOT (docId 링크로 연결)
- **3단계 구조**: Cycle → Phase → Task (Initiative는 선택적)

---

## 2. 아키텍처

### 2.1 페이지 구조

```
Operations Hub (루트)
│
├── Executive Dashboard (경영진 요약)
│   └── KPI, 포트폴리오, 리스크, 마스터 플랜
│
├── Team Status (실시간 현황)
│   └── 주간 실행, Spec 파이프라인, Dev Timeline, Review Queue
│
├── My Workspace (운영자 작업)
│   └── 내 할 일, Gate 현황, Blocked 관리, Cycle 관리
│
└── Data/ (원본 DB)
    ├── Cycles DB
    ├── Phases DB
    ├── Ops Tasks DB
    └── Initiatives DB
```

### 2.2 네비게이션 설계

```
┌─────────────────────────────────────────────────────────────┐
│ [Active Cycle]  2026 Q1  │  상태: 진행중  │  D-45           │
├─────────────────────────────────────────────────────────────┤
│ [Executive] [Team Status] [My Workspace] [Data]             │
└─────────────────────────────────────────────────────────────┘
```

### 2.3 데이터 흐름

```
[Cycles DB]
    │
    ├──→ [Initiatives DB] ──→ 포트폴리오 뷰
    │         │
    │         ▼
    └──→ [Phases DB] ──→ 마스터 플랜, Spec 파이프라인, Dev Timeline
              │
              ▼
         [Ops Tasks DB] ──→ 주간 실행, Review Queue, 내 할 일
```

---

## 3. 페이지별 상세 설계

### 3.1 Executive Dashboard (경영진 보고용)

**목적**: 이번 분기(Cycle)에 무엇을 하고 있고, 얼마나 진행되었는지 한눈에 파악

**핵심 원칙**:
- **Initiative 중심**: 경영진은 비즈니스 목표(Initiative) 단위로 현황 파악
- **운영 지표 제외**: Gate, Blocked, Review SLA 등 운영 레벨 지표는 Team Status/My Workspace로 이동
- **간결한 KPI**: Cycle 진행률 + Initiative 완료율 (2개)

#### 3.1.1 레이아웃

```
┌─────────────────────────────────────────────────────────────┐
│ [Cycle 현황 카드]                                            │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  2026 Q1  │  진행률: ████████░░ 75%  │  D-45           │  │
│  │  Initiative: 8개 (완료 3 / 진행 4 / 예정 1)            │  │
│  │  상태: On Track                                        │  │
│  └───────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────┤
│ [Initiative 포트폴리오]                                      │
│  ┌───────────────────────────────────────────────────────┐  │
│  │ Table View (Initiative 전체 목록)                      │  │
│  │ ┌─────────────┬──────────┬──────────┬────────┐        │  │
│  │ │ Initiative  │ 상태     │ 진행률   │ 책임자 │        │  │
│  │ ├─────────────┼──────────┼──────────┼────────┤        │  │
│  │ │ 결제 개선    │ 진행중   │ ████░░ 60% │ 김OO  │        │  │
│  │ │ 정산 자동화  │ 위험     │ ██░░░░ 30% │ 박OO  │        │  │
│  │ └─────────────┴──────────┴──────────┴────────┘        │  │
│  └───────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────┤
│ [Initiative Timeline]                                        │
│  ┌───────────────────────────────────────────────────────┐  │
│  │ Timeline (Initiative 기준, Sub-item으로 Phase 표시)    │  │
│  │ ════════════════════════════════════════════════════   │  │
│  │ [결제 개선] ────────────────────────                    │  │
│  │    └─[Spec]──[Dev]──────[QA]──[Rel]                    │  │
│  │ [정산 자동화] ──────────────────────────                │  │
│  │    └─[Spec]────[Dev]────────[QA]──                     │  │
│  │ ════════════════════════════════════════════════════   │  │
│  └───────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────┤
│ [주의 필요 항목]                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │ At Risk / Delayed Initiative (상위 3개)                │  │
│  │ - 정산 자동화: 일정 지연 (Dev Phase 2주 초과)           │  │
│  │ - 리포트 개편: 리소스 부족 (담당자 공석)                │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

#### 3.1.2 구성 요소

| 구성 요소 | 소스 DB | 뷰 타입 | 필터 조건 | 정렬 |
|----------|---------|---------|----------|------|
| Cycle 현황 카드 | Cycles (Linked) | Gallery (카드 1개) | Status = Active | - |
| Initiative 포트폴리오 | Initiatives (Linked) | Table | Cycle Status = Active | 우선순위, 상태 |
| Initiative Timeline | Initiatives (Linked) | Timeline | Cycle Status = Active, 상태 in (확정, 진행중, 릴리즈됨) | 시작일 |
| 주의 필요 항목 | Initiatives (Linked) | Table | Cycle Status = Active, 상태 아이콘 in (At Risk, Delayed) | 우선순위 |

#### 3.1.3 뷰 상세 설정

**Cycle 현황 카드**:
- 뷰 타입: Gallery (카드 크기: Large, 1개만 표시)
- 필터: `상태 = 진행중`
- 표시 속성:
  - Cycle명 (예: 2026 Q1)
  - 진행률 Progress Bar (Initiative 기준)
  - D-Day (종료일까지 남은 일수)
  - Initiative 현황 (완료/진행/예정 수)
  - 상태 아이콘 (On Track / At Risk / Off Track)

**Initiative 포트폴리오**:
- 뷰 타입: Table
- 필터: `Cycle Status = Active`
- 정렬: 우선순위 (P0 → P3), 상태
- 표시 속성:
  - Initiative명
  - 상태 아이콘 (On Track / At Risk / Delayed / 완료)
  - 진행률 (Progress Bar)
  - 기획 책임자
  - 기술 책임자
  - 목표 (측정 가능한 결과)

**Initiative Timeline**:
- 뷰 타입: Timeline
- 필터: `Cycle Status = Active AND 상태 in (확정, 진행중, 릴리즈됨)`
- 날짜 범위: Initiative 시작일 ~ 종료일 (Rollup으로 계산)
- Sub-item: Phases (Spec → Dev → QA → Release 순서)
- 표시: Initiative명, 진행률

**주의 필요 항목**:
- 뷰 타입: Table 또는 List
- 필터: `Cycle Status = Active AND 상태 아이콘 in (At Risk, Delayed)`
- 정렬: 우선순위 (P0 먼저)
- 제한: 상위 3~5개
- 표시 속성: Initiative명, 상태 아이콘, 지연 사유

---

### 3.2 Team Status (실시간 현황)

**목적**: 팀원들이 현재 진행 상황을 빠르게 확인

#### 3.2.1 레이아웃

```
┌─────────────────────────────────────────────────────────────┐
│ [Today's Focus]                                              │
│  In Progress: 3  │  Review Needed: 2  │  Blocked: 1         │
├─────────────────────────────────────────────────────────────┤
│ [주간 실행 Kanban]             │ [Spec 파이프라인]           │
│ ┌─────────────────────┐       │ ┌───────────────────────┐   │
│ │ This Week | In Prog │       │ │ Spec Phase Board      │   │
│ │ Review   | Done     │       │ │ (Gate 상태 표시)       │   │
│ └─────────────────────┘       │ └───────────────────────┘   │
├─────────────────────────────────────────────────────────────┤
│                   [Dev Timeline]                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ Dev Phases Timeline (2주 범위)                       │   │
│  └─────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────┤
│ [Review Queue]                                               │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ Review Needed Tasks Table                            │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

#### 3.2.2 구성 요소

| 구성 요소 | 소스 DB | 뷰 타입 | 필터 조건 | 정렬 |
|----------|---------|---------|----------|------|
| Today's Focus | Ops Tasks | Gallery (Count) | Status in (In Progress, Review Needed, Blocked) | - |
| 주간 실행 | Ops Tasks | Kanban | Cycle Status = Active AND Status in (This Week, In Progress, Review Needed, Done) | 마감일 |
| Spec 파이프라인 | Phases | Board | Cycle Status = Active AND Phase Type = Spec | Start Date |
| Dev Timeline | Phases | Timeline | Cycle Status = Active AND Phase Type = Dev | Start Date |
| Review Queue | Ops Tasks | Table | Status = Review Needed | 생성일 Asc |

#### 3.2.3 뷰 상세 설정

**주간 실행 Kanban**:
- Group by: 상태 (This Week | In Progress | Review Needed | Done)
- 필터: `Cycle Status = Active AND 상태 in (This Week, In Progress, Review Needed, Done)`
- 카드 표시: Task명, 책임자, 마감일, Phase 링크

**Spec 파이프라인 Board**:
- Group by: 상태 (미시작 | 진행중 | 막힘 | 완료)
- 필터: `Cycle Status = Active AND Phase 유형 = Spec`
- 카드 표시: Phase명, 책임자, Gate 상태 (Ready/Not Ready)

---

### 3.3 My Workspace (운영자 작업)

**목적**: Cycle Owner/PM의 일상 운영 도구

#### 3.3.1 레이아웃

```
┌─────────────────────────────────────────────────────────────┐
│ [Quick Actions]                                              │
│  [+ New Phase]  [+ New Task]  [Toggle Active Cycle]         │
├─────────────────────────────────────────────────────────────┤
│ [내 할 일]                     │ [Gate 현황]                 │
│ ┌─────────────────────┐       │ ┌───────────────────────┐   │
│ │ My Tasks Table      │       │ │ Not Ready Gates       │   │
│ │ (Owner = Me)        │       │ │ (체크리스트 진행률)    │   │
│ └─────────────────────┘       │ └───────────────────────┘   │
├─────────────────────────────────────────────────────────────┤
│ [Blocked 레지스터]                                           │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ Blocked Phases/Tasks Table                           │   │
│  └─────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────┤
│ [Cycle 관리]                   │ [Planning Queue]           │
│ ┌─────────────────────┐       │ ┌───────────────────────┐   │
│ │ All Cycles Table    │       │ │ 예정 Cycles           │   │
│ └─────────────────────┘       │ └───────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

#### 3.3.2 구성 요소

| 구성 요소 | 소스 DB | 뷰 타입 | 필터 조건 | 정렬 |
|----------|---------|---------|----------|------|
| Quick Actions | - | Button Block | - | - |
| 내 할 일 | Ops Tasks | Table | Owner = Me AND Status != Done | 마감일 Asc |
| Gate 현황 | Phases | Table | Cycle Status = Active AND Gate = Not Ready | Phase 유형 |
| Blocked 레지스터 | Phases | Table | Status = Blocked | 막힘일 Asc |
| Cycle 관리 | Cycles | Table | 전체 | 시작일 Desc |
| Planning Queue | Cycles | Table | Status = 예정 | 시작일 Asc |

#### 3.3.3 뷰 상세 설정

**내 할 일 Table**:
- 필터: `책임자 = Me AND 상태 != Done`
- 표시 속성: Task명, 상태, 마감일, Phase 링크, Priority
- 정렬: 마감일 Asc

**Gate 현황 Table**:
- 필터: `Cycle Status = Active AND Gate 상태 = Not Ready`
- 표시 속성: Phase명, Phase 유형, Gate 상태, 책임자, Gate 미충족 사유
- 정렬: Phase 유형 (Spec → Dev → QA → Release)

---

## 4. Linked Database 설정

### 4.1 원본 DB 위치

```
/Operations Hub/Data/
├── Cycles (원본 DB)
├── Phases (원본 DB)
├── Ops Tasks (원본 DB)
└── Initiatives (원본 DB)
```

### 4.2 Linked DB 규칙

| 원칙 | 설명 |
|------|------|
| **단일 원본** | 모든 데이터는 Data 폴더의 원본 DB에만 저장 |
| **Linked 활용** | 대시보드 페이지에서는 Linked Database로 뷰만 구성 |
| **뷰 네이밍** | `{페이지약어}-{용도}` (예: Exec-마스터플랜, Team-주간실행) |
| **필터 일관성** | 모든 Linked DB에 `Cycle Status = Active` 필터 기본 적용 |

### 4.3 페이지별 Linked DB 목록

| 페이지 | Linked DB | 뷰 이름 | 용도 |
|--------|----------|---------|------|
| Executive | Cycles | Exec-ActiveCycle | KPI 위젯 |
| Executive | Initiatives | Exec-포트폴리오 | 포트폴리오 요약 |
| Executive | Phases | Exec-마스터플랜 | Timeline 뷰 |
| Executive | Phases | Exec-리스크 | Blocked 목록 |
| Team Status | Ops Tasks | Team-주간실행 | Kanban |
| Team Status | Phases | Team-Spec파이프 | Spec Board |
| Team Status | Phases | Team-DevTimeline | Dev Timeline |
| Team Status | Ops Tasks | Team-ReviewQueue | Review 대기 |
| My Workspace | Ops Tasks | My-할일 | 내 할 일 |
| My Workspace | Phases | My-Gate현황 | Not Ready Gate |
| My Workspace | Phases | My-Blocked | Blocked 관리 |
| My Workspace | Cycles | My-Cycles | Cycle 관리 |

---

## 5. KPI 위젯 구현

### 5.1 KPI 정의

| KPI | 정의 | 목표 | 측정 주기 |
|-----|------|------|----------|
| **계획 대비 완료율** | Committed 중 Released 비율 | >= 80% | Cycle 종료 |
| **Carry-over 비율** | 이월된 Initiative 비율 | <= 20% | Cycle 종료 |
| **Spec Gate 리드타임** | 초안 → Ready 평균 일수 | <= 5일 | Phase별 |
| **Blocked 해소 시간** | 막힘 발생 → 해소 평균 일수 | <= 3일 | 주간 |
| **Review SLA** | Review Needed 평균 대기 시간 | <= 1일 | 주간 |

### 5.2 측정 방법

#### 5.2.1 계획 대비 완료율

**계산식**: `Released Initiative 수 / Committed Initiative 수 * 100`

**Committed 정의**: 상태 = 확정 OR 진행중 OR 릴리즈됨

#### 5.2.2 Carry-over 비율

**계산식**: `(Committed - Released - 제외) / Committed * 100`

**이월 정의**: Cycle 종료 시점에 릴리즈됨/제외가 아닌 Committed 항목

### 5.3 Formula/Rollup 설정

#### 5.3.1 Cycles DB 추가 속성

| 속성명 | 타입 | 설정 |
|--------|------|------|
| Phase 수 | Rollup | Phases → Count all |
| 완료 Phase 수 | Rollup | Phases (Status = 완료) → Count all |
| Initiative 수 | Rollup | Initiatives → Count all |
| 완료 Initiative 수 | Rollup | Initiatives (Status = 릴리즈됨) → Count all |
| 진행률 | Formula | `round(prop("완료 Phase 수") / prop("Phase 수") * 100)` |
| D-Day | Formula | `dateBetween(prop("종료일"), now(), "days")` |

#### 5.3.2 Progress Bar Formula

```
// Cycles DB - 진행률 Progress Bar
if(prop("Phase 수") > 0,
  slice("██████████", 0, round(prop("진행률") / 10)) +
  slice("░░░░░░░░░░", 0, 10 - round(prop("진행률") / 10)) +
  " " + format(prop("진행률")) + "%",
  "N/A")
```

#### 5.3.3 KPI 상태 표시

```
// 완료율 상태 판정
if(prop("진행률") >= 80, "🟢 On Track",
  if(prop("진행률") >= 60, "🟡 At Risk", "🔴 Off Track"))
```

### 5.4 시각화 방법

**Callout 블록 + Linked DB 조합**:

```
┌────────────────────────────────────────────┐
│ 📊 완료율                                   │
│    ████████░░ 75%                          │
│    목표: 80% │ 상태: 🟡 At Risk             │
└────────────────────────────────────────────┘
```

**구현 방법**:
1. Callout 블록 생성 (아이콘: 📊)
2. 내부에 Linked DB (Cycles) Gallery 뷰 삽입
3. 카드 1개만 표시 (Active Cycle)
4. 카드에 Progress Bar Formula, 상태 표시

---

## 6. 자동화 설정

### 6.1 Notion 내장 자동화

| 우선순위 | 트리거 | 조건 | 액션 | 대상 DB |
|----------|--------|------|------|---------|
| P0 | 상태 변경 | → 진행중 (Cycle) | 기존 진행중 → 완료 전환 | Cycles |
| P0 | 상태 변경 | → 막힘 (Phase) | Slack 알림 + Owner 멘션 | Phases |
| P1 | 상태 변경 | → Review Needed | 리뷰어 멘션 알림 | Ops Tasks |
| P1 | 날짜 도래 | 마감일 D-1 | 담당자 멘션 알림 | Ops Tasks |
| P2 | Gate 변경 | → Ready | Slack 채널 알림 | Phases |
| P2 | 상태 변경 | → 릴리즈됨 | Slack 채널 알림 | Initiatives |

### 6.2 자동화 상세

#### 6.2.1 Active Cycle 단일 유지 (P0)

```
트리거: Cycles DB, 상태 속성이 "진행중"으로 변경됨
조건: 없음
액션:
  1. 다른 Cycles 중 상태 = "진행중"인 항목 검색
  2. 해당 항목의 상태를 "완료"로 변경
```

#### 6.2.2 막힘 발생 알림 (P0)

```
트리거: Phases DB, 상태 속성이 "막힘"으로 변경됨
조건: Cycle Status = Active
액션:
  1. Slack Webhook 호출
  2. 메시지: "[Phase명] 막힘 발생 - @책임자 확인 필요"
  3. Phase 페이지 링크 포함
```

### 6.3 Slack 연동 설정

**Webhook URL**: Notion 자동화에서 Slack Incoming Webhook 연결

**알림 채널 구분**:

| 알림 유형 | 채널 | 멘션 |
|----------|------|------|
| 막힘 발생 | #ops-alerts | @책임자 |
| Review 요청 | #ops-review | @리뷰어 |
| Gate Ready | #ops-general | @channel |
| 릴리즈 완료 | #ops-general | - |

---

## 7. 권한 및 공유 설정

### 7.1 역할별 권한

| 역할 | Executive Dashboard | Team Status | My Workspace | Data (원본 DB) |
|------|-------------------|-------------|--------------|----------------|
| 경영진 | View | View | - | - |
| Cycle Owner | View | Edit | Full | Full |
| PM/PO | View | Edit | Full (본인 항목) | Edit |
| Tech Owner | View | Edit | Full (본인 항목) | Edit |
| QA Owner | View | Edit | Full (본인 항목) | Edit |
| 팀원 | View | View | View (본인 항목) | View |

### 7.2 공유 범위

| 페이지 | 공유 설정 | 설명 |
|--------|----------|------|
| Operations Hub | Workspace 전체 | 기본 진입점 |
| Executive Dashboard | 경영진 그룹 + Cycle Owner | 민감 지표 포함 |
| Team Status | Workspace 전체 | 실시간 현황 공유 |
| My Workspace | 개인별 (Person = Me 필터) | 개인 작업 공간 |
| Data | Cycle Owner + 편집 권한자 | 원본 데이터 보호 |

---

## 8. 운영 규칙

### 8.1 뷰 추가/수정 규칙

| 규칙 | 설명 |
|------|------|
| **신규 뷰 승인** | Cycle Owner 승인 후 추가 |
| **필터 변경 금지** | Active Cycle 기본 필터 변경 금지 |
| **네이밍 준수** | `{페이지약어}-{용도}` 형식 준수 |
| **문서화** | 뷰 추가 시 본 스펙 문서 업데이트 |

### 8.2 유지보수 체크리스트

**주간 (금요일 리뷰 시)**:
- [ ] KPI 위젯 정상 표시 확인
- [ ] Blocked 레지스터 최신 상태 확인
- [ ] Review Queue 적체 확인

**월간 (첫째 주 금요일)**:
- [ ] 불필요한 뷰 정리
- [ ] 자동화 규칙 동작 확인
- [ ] 권한 설정 검토

**Cycle 종료 시**:
- [ ] 완료율/이월률 KPI 기록
- [ ] Archive 정리
- [ ] 다음 Cycle 대시보드 준비

---

## 9. 개인 작업 영역 (Personal Planning Zone)

### 9.1 개요

My Workspace에 **Personal Planning Zone**을 추가하여 개인 작업을 비공개로 관리한다. 팀 일정과 비교하면서 작업하고, 준비가 되면 자동화를 통해 팀 DB로 공개한다.

### 9.2 아키텍처

```
Operations Hub/
├── Data/ (팀 공개)
│   ├── Cycles, Phases, Ops Tasks, Initiatives
│
└── Personal/ (개인 전용, 권한 분리)
    ├── Personal Phases
    └── Personal Tasks
```

### 9.3 My Workspace 확장 레이아웃

```
┌─────────────────────────────────────────────────────────────┐
│ [기존 My Workspace 영역]                                     │
│  내 할 일, Gate 현황, Blocked 레지스터, Cycle 관리           │
├─────────────────────────────────────────────────────────────┤
│ [Personal Planning Zone] ─── 개인 전용 영역                  │
│                                                              │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ [개인 + 팀 통합 비교]                                    │ │
│ │  ════════════════════════════════════════════════════   │ │
│ │  팀 Phases (Linked, Read-only)                          │ │
│ │  ────────────────────────────────────────────────────   │ │
│ │  Personal Phases (내 개인 일정)                          │ │
│ │  ════════════════════════════════════════════════════   │ │
│ └─────────────────────────────────────────────────────────┘ │
│                                                              │
│ ┌────────────────────────┐  ┌────────────────────────────┐ │
│ │ [공개 대기 목록]         │  │ [Personal Tasks Kanban]   │ │
│ └────────────────────────┘  └────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### 9.4 Personal DB 뷰

| 뷰 이름 | 소스 DB | 필터 | 용도 |
|---------|---------|------|------|
| 팀 일정 (참조) | Phases (Linked) | Cycle Status = Active | 팀 일정 비교 (Read-only) |
| 개인 Timeline | Personal Phases | 공개 상태 ≠ 공개완료 | 개인 일정 전체 |
| 공개 대기 | Personal Phases | 공개 상태 = 공개대기 | 이관 예정 항목 |
| 개인 Tasks | Personal Tasks | 상태 ≠ Done | 개인 실행 작업 |

### 9.5 자동화 (Make)

| 시나리오 | 트리거 | 액션 |
|----------|--------|------|
| 공개 알림 | 매일 09:00 (D-3) | Slack 알림 + 공개대기 전환 |
| 자동 공개 | 공개 예정일 도래 | 팀 Phases DB로 복사 + 공개완료 |
| 수동 공개 | 버튼 클릭 | 즉시 팀 DB로 복사 |

### 9.6 상세 스펙

Personal Phases/Tasks의 상세 속성 및 자동화 설계는 하위 문서 참조:
- **DBS-notion-personal-001**: Personal Workspace DB 스펙

---

## 10. 관련 문서

### 10.1 상위 문서

| docId | 제목 | 관계 |
|-------|------|------|
| PRC-planops-001 | 운영/진척 관리 프로세스 | 상위 프로세스 |
| PROP-planops-001 | 운영/진척 관리 프로세스 도입 제안 | 근거 제안서 |

### 10.2 참조 DB 스펙

| docId | 제목 | 관계 |
|-------|------|------|
| DBS-notion-cycles-001 | Cycles DB 스펙 | 원본 DB |
| DBS-notion-phases-001 | Phases DB 스펙 | 원본 DB |
| DBS-notion-opstasks-001 | Ops Tasks DB 스펙 | 원본 DB |
| DBS-notion-initiatives-001 | Initiatives DB 스펙 | 원본 DB |

### 10.3 하위 문서

| docId | 제목 | 관계 |
|-------|------|------|
| DBS-notion-personal-001 | Personal Workspace DB 스펙 | 개인 영역 상세 |

### 10.4 규칙 문서

| 문서 | 적용 내용 |
|------|----------|
| `.agent/rules/document-schema.md` | docId 네이밍 규칙 |
| `.agent/rules/ownership.md` | Owner 단일 지정 |
| `.agent/rules/change-control.md` | Approved 후 CHG 변경 |

---

## 11. 변경 이력

| 버전 | 날짜 | 변경 내용 | 작성자 |
|------|------|----------|--------|
| 1.0 | 2026-01-07 | 최초 작성 | 박재형 |
| 1.1 | 2026-01-07 | 개인 작업 영역(Personal Planning Zone) 섹션 추가 | 박재형 |
