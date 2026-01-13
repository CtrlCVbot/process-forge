# Notion DB 스펙 - Personal Workspace

---
docId: DBS-notion-personal-001
title: Notion DB 스펙 - Personal Workspace
owner: 박재형
status: Draft
version: 1.0
created: 2026-01-07
updated: 2026-01-07
parent: DBS-notion-dashboard-001
---

## 1. 개요

### 1.1 목적

Personal Workspace는 **개인 작업을 비공개로 관리**하면서 **팀 일정과 비교**할 수 있는 공간이다. 작업이 준비되면 **자동화를 통해 팀 DB로 공개**할 수 있다.

### 1.2 핵심 원칙

| 원칙 | 설명 |
|------|------|
| **완전한 권한 분리** | Personal DB는 본인만 접근 가능 |
| **팀 일정 참조 가능** | Linked DB로 팀 일정을 Read-only로 비교 |
| **자동화 공개** | 공개 예정일 도래 시 Make 자동화로 팀 DB에 복사 |
| **팀 DB 구조 호환** | 공개 시 매핑이 용이하도록 유사한 속성 구조 유지 |

### 1.3 사용 시나리오

```
예시: 2분기 Java 마이그레이션 사전 조사

1. 1분기 중 Personal Phase 생성: "[Research] Java 마이그레이션 조사"
2. 목표 Cycle: 2026 Q2, 공개 예정일: 2026-04-01
3. 개인 Timeline에서 팀 일정과 비교하며 작업
4. 2026-03-29 (D-3): 공개 알림 수신, 공개 상태 → "공개대기"
5. 2026-04-01: 자동화로 팀 Phases DB에 복사, 공개 상태 → "공개완료"
```

---

## 2. 아키텍처

### 2.1 DB 구조

```
Operations Hub/
├── Data/ (팀 공개)
│   ├── Cycles
│   ├── Phases
│   ├── Ops Tasks
│   └── Initiatives
│
└── Personal/ (개인 전용)
    ├── Personal Phases ─────┐
    └── Personal Tasks ──────┴─→ (Make 자동화) → Data/Phases, Ops Tasks
```

### 2.2 권한 설계

| 영역 | 권한 | 접근자 |
|------|------|--------|
| Data/* | Workspace 전체 | 팀 전원 |
| Personal/* | 개인만 | Owner 본인 (또는 지정 멤버) |

### 2.3 데이터 흐름

```
[Personal Phases] ──(공개)──→ [Phases]
       │                          │
       ▼                          ▼
[Personal Tasks] ──(공개)──→ [Ops Tasks]
```

---

## 3. Personal Phases DB

### 3.1 속성 정의

| 속성명 | 타입 | 필수 | 설명 | 예시 |
|--------|------|------|------|------|
| Phase명 | Title | O | [유형] 작업명 - 목적 | [Research] Java 마이그레이션 조사 |
| Phase 유형 | Select | O | 작업 유형 | Spec/Dev/QA/Release/Research |
| 시작일 | Date | O | 예정 시작일 | 2026-01-15 |
| 종료일 | Date | O | 예정 종료일 | 2026-02-28 |
| 상태 | Select | O | 작업 진행 상태 | 미시작/진행중/막힘/완료 |
| 책임자 | Person | O | Owner (본인) | 박재형 |
| **목표 Cycle** | Select | O | 공개 목표 기간 | 2026 Q1/Q2/Q3/Q4 |
| **공개 예정일** | Date | O | 팀 DB 이관 예정일 | 2026-04-01 |
| **공개 상태** | Select | O | 공개 진행 상태 | 준비중/공개대기/공개완료 |
| 산출물 docIds | Text | - | SSOT 연결 키 | PRD-migration-001 |
| 메모 | Text | - | 개인 메모 | 조사 범위: Spring Boot 3.x |

### 3.2 Phase 유형

| 유형 | 설명 | 팀 DB 매핑 |
|------|------|-----------|
| Spec | 기획/요구사항 정의 | Spec |
| Dev | 개발 | Dev |
| QA | 품질 검증 | QA |
| Release | 릴리즈 | Release |
| **Research** | 사전 조사/PoC (개인 전용) | Spec (공개 시) |

### 3.3 공개 상태 정의

| 상태 | 설명 | 전환 조건 | 전환 주체 |
|------|------|----------|----------|
| **준비중** | 개인 작업 진행 중 | 기본값 | - |
| **공개대기** | 공개 준비 완료, 대기 | 공개 예정일 D-3 | 자동화 (Make) |
| **공개완료** | 팀 DB로 이관됨 | 공개 예정일 도래 | 자동화 (Make) |

### 3.4 관계 (Relations)

| 대상 DB | 관계 유형 | 설명 |
|---------|----------|------|
| Personal Tasks | 1:N | 이 Phase의 개인 Task 목록 |

**참고**: Personal Phases는 팀 Cycles DB와 직접 연결하지 않음 (권한 분리). 대신 `목표 Cycle` 속성으로 텍스트 참조.

---

## 4. Personal Tasks DB

### 4.1 속성 정의

| 속성명 | 타입 | 필수 | 설명 | 예시 |
|--------|------|------|------|------|
| Task명 | Title | O | 실행 작업명 | Spring Boot 버전 조사 |
| Personal Phase | Relation | O | 소속 Personal Phase | [Research] Java 마이그레이션 |
| 상태 | Select | O | 진행 상태 | Backlog/In Progress/Done |
| 마감일 | Date | - | 개인 마감일 | 2026-01-20 |
| 메모 | Text | - | 개인 메모 | 공식 문서 링크 정리 |

### 4.2 상태 정의

| 상태 | 설명 |
|------|------|
| **Backlog** | 아직 착수 안 함 |
| **In Progress** | 진행 중 |
| **Done** | 완료 |

**참고**: 팀 Ops Tasks보다 단순한 상태 (This Week, Review Needed 등 생략). 공개 시 상태는 "Backlog"로 초기화.

---

## 5. 표준 뷰

### 5.1 My Workspace 내 개인 영역

```
┌─────────────────────────────────────────────────────────────┐
│ [Personal Planning Zone]                                     │
│                                                              │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ [개인 + 팀 통합 비교]                                    │ │
│ │  ────────────────────────────────────────────────────   │ │
│ │  팀 Phases (Linked, Read-only)                          │ │
│ │  ────────────────────────────────────────────────────   │ │
│ │  Personal Phases (내 개인 일정)                          │ │
│ │  ────────────────────────────────────────────────────   │ │
│ └─────────────────────────────────────────────────────────┘ │
│                                                              │
│ ┌────────────────────────┐  ┌────────────────────────────┐ │
│ │ [공개 대기 목록]         │  │ [Personal Tasks Kanban]   │ │
│ │ Table                   │  │ Board                     │ │
│ └────────────────────────┘  └────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### 5.2 뷰 상세

#### 5.2.1 팀 일정 참조 (Linked DB)

| 항목 | 설정 |
|------|------|
| 소스 DB | Phases (Linked Database) |
| 뷰 타입 | Timeline |
| 필터 | Cycle Status = Active |
| 용도 | 팀 일정과 개인 일정 비교 (Read-only) |

#### 5.2.2 개인 Timeline

| 항목 | 설정 |
|------|------|
| 소스 DB | Personal Phases |
| 뷰 타입 | Timeline |
| 필터 | 공개 상태 ≠ 공개완료 |
| Group By | 목표 Cycle |
| 용도 | 개인 일정 전체 조회 |

#### 5.2.3 공개 대기 목록

| 항목 | 설정 |
|------|------|
| 소스 DB | Personal Phases |
| 뷰 타입 | Table |
| 필터 | 공개 상태 = 공개대기 |
| 정렬 | 공개 예정일 (Asc) |
| 표시 속성 | Phase명, 목표 Cycle, 공개 예정일, 상태 |
| 용도 | 곧 공개될 항목 관리 |

#### 5.2.4 Personal Tasks Kanban

| 항목 | 설정 |
|------|------|
| 소스 DB | Personal Tasks |
| 뷰 타입 | Kanban (Board) |
| 필터 | 상태 ≠ Done |
| Group By | 상태 |
| 용도 | 개인 실행 작업 관리 |

---

## 6. 자동화 설계

### 6.1 자동화 도구 선택

| 도구 | 추천 | 이유 |
|------|------|------|
| **Make (Integromat)** | O | Notion 네이티브 연동, 시각적 워크플로우, 적정 비용 |
| Zapier | - | 비용 높음 |
| n8n | - | Self-hosted 필요 |
| Notion API Script | - | 개발/운영 부담 |

### 6.2 시나리오 1: 공개 예정 알림 (D-3)

**목적**: 공개 3일 전 알림 및 상태 전환

```
트리거: 매일 09:00 (스케줄)

1. Notion: Search Personal Phases
   - 필터: 공개 예정일 = today + 3일 AND 공개 상태 = 준비중

2. Iterator: 각 Phase에 대해

3. Slack: Send Message
   - 채널: DM 또는 개인 채널
   - 메시지: "📢 3일 후 공개 예정: [Phase명]\n목표 Cycle: [목표 Cycle]\n공개 준비를 확인하세요."

4. Notion: Update Page
   - 공개 상태 → "공개대기"
```

**Make 워크플로우 다이어그램**:

```
┌──────────┐   ┌──────────────┐   ┌──────────┐   ┌──────────────┐
│ Schedule │ → │ Notion:      │ → │ Iterator │ → │ Slack: DM    │
│ Daily 9AM│   │ Search DB    │   │          │   └──────────────┘
└──────────┘   │ (D-3 필터)   │   │          │          │
               └──────────────┘   │          │          ▼
                                  │          │   ┌──────────────┐
                                  └──────────┘   │ Notion:      │
                                                 │ Update 상태  │
                                                 └──────────────┘
```

### 6.3 시나리오 2: 자동 공개 실행

**목적**: 공개 예정일 도래 시 팀 DB로 복사

```
트리거: 매일 09:00 (스케줄)

1. Notion: Search Personal Phases
   - 필터: 공개 예정일 = today AND 공개 상태 = 공개대기

2. Iterator: 각 Phase에 대해

3. Notion: Search Cycles
   - 필터: 이름 = [목표 Cycle] (예: "2026 Q2")
   - 결과: Cycle ID 획득

4. Notion: Create Page (팀 Phases DB)
   - Phase명: 직접 복사
   - Phase 유형: 직접 복사 (Research → Spec)
   - 시작일/종료일: 직접 복사
   - 상태: "미시작"
   - 책임자: 직접 복사
   - 사이클: Cycle ID로 Relation 연결
   - Gate 상태: "Not Ready"
   - 산출물 docIds: 직접 복사

5. (선택) Personal Tasks → Ops Tasks 복사
   - 해당 Personal Phase의 모든 Task 조회
   - 각 Task를 팀 Ops Tasks DB에 생성
   - Phase Relation을 새로 생성된 팀 Phase로 연결

6. Notion: Update Personal Phase
   - 공개 상태 → "공개완료"

7. Slack: Send Message
   - "✅ [Phase명]이 팀 일정에 공개되었습니다."
```

**Make 워크플로우 다이어그램**:

```
┌──────────┐   ┌──────────────┐   ┌──────────┐   ┌──────────────┐
│ Schedule │ → │ Notion:      │ → │ Iterator │ → │ Notion:      │
│ Daily 9AM│   │ Search       │   │          │   │ Get Cycle    │
└──────────┘   │ Personal     │   │          │   └──────────────┘
               │ Phases       │   │          │          │
               │ (공개대기)    │   │          │          ▼
               └──────────────┘   │          │   ┌──────────────┐
                                  │          │   │ Notion:      │
                                  │          │   │ Create Team  │
                                  │          │   │ Phase        │
                                  └──────────┘   └──────────────┘
                                                        │
                        ┌───────────────────────────────┘
                        ▼
                 ┌──────────────┐   ┌──────────────┐
                 │ Notion:      │ → │ Slack:       │
                 │ Update       │   │ 공개 알림    │
                 │ → 공개완료   │   └──────────────┘
                 └──────────────┘
```

### 6.4 시나리오 3: 수동 즉시 공개

**목적**: 버튼 클릭으로 즉시 공개

```
트리거: Notion 버튼 → Webhook URL 호출

1. Webhook: Receive (Personal Phase ID 전달)

2. Notion: Get Page (Personal Phase)

3. (시나리오 2의 3~7 단계와 동일)
```

**Notion 버튼 설정**:
- Personal Phase 템플릿에 "즉시 공개" 버튼 추가
- 버튼 액션: Open URL → Make Webhook URL + Page ID 파라미터

### 6.5 속성 매핑 테이블

| Personal Phase 속성 | 팀 Phase 속성 | 매핑 방식 |
|---------------------|---------------|----------|
| Phase명 | Phase명 | 직접 복사 |
| Phase 유형 | Phase 유형 | 직접 복사 (Research → Spec) |
| 시작일 | 시작일 | 직접 복사 |
| 종료일 | 종료일 | 직접 복사 |
| 상태 | 상태 | "미시작"으로 초기화 |
| 책임자 | 책임자 | 직접 복사 |
| 목표 Cycle | 사이클 (Relation) | Cycle명으로 검색 후 연결 |
| 산출물 docIds | 산출물 docIds | 직접 복사 |
| - | Gate 상태 | "Not Ready"로 초기화 |
| - | 이니셔티브 | 비워둠 (수동 연결) |

| Personal Task 속성 | 팀 Ops Task 속성 | 매핑 방식 |
|--------------------|------------------|----------|
| Task명 | Task명 | 직접 복사 |
| Personal Phase | Phase | 새 팀 Phase ID로 연결 |
| 상태 | 상태 | "Backlog"로 초기화 |
| 마감일 | 마감일 | 직접 복사 |
| - | 책임자 | Personal Phase 책임자 복사 |

---

## 7. 운영 규칙

### 7.1 Personal Phase 생성 규칙

1. **목표 Cycle 필수**: 언제 공개할지 명확히 설정
2. **공개 예정일 필수**: 자동화 트리거를 위해 필수
3. **Phase 유형 지정**: Research는 공개 시 Spec으로 매핑됨

### 7.2 공개 전 체크리스트

공개 전 (D-3 알림 수신 시) 확인:
- [ ] 시작일/종료일이 팀 일정과 충돌 없음
- [ ] 산출물 docIds가 정확히 입력됨
- [ ] 필요 시 Initiative 연결 계획 수립
- [ ] 관련 팀원에게 사전 공유 (필요 시)

### 7.3 공개 후 처리

1. **팀 Phase 확인**: 정상 생성 여부 확인
2. **Cycle/Initiative 연결**: 필요 시 수동 조정
3. **Personal Phase 아카이브**: 공개완료 항목은 그대로 유지 (이력 보관)

---

## 8. 관련 문서

### 8.1 상위 문서

| docId | 제목 | 관계 |
|-------|------|------|
| DBS-notion-dashboard-001 | 대시보드 스펙 | 상위 스펙 |
| PRC-planops-001 | 운영/진척 관리 프로세스 | 상위 프로세스 |

### 8.2 참조 문서

| docId | 제목 | 관계 |
|-------|------|------|
| DBS-notion-phases-001 | Phases DB 스펙 | 공개 대상 DB |
| DBS-notion-opstasks-001 | Ops Tasks DB 스펙 | 공개 대상 DB |
| DBS-notion-cycles-001 | Cycles DB 스펙 | 목표 Cycle 참조 |

---

## 변경 이력

| 버전 | 날짜 | 변경 내용 | 작성자 |
|------|------|----------|--------|
| 1.0 | 2026-01-07 | 최초 작성 | 박재형 |
