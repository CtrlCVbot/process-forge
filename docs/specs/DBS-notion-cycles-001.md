# Notion DB 스펙 - Cycles

---
docId: DBS-notion-cycles-001
title: Notion DB 스펙 - Cycles
owner: 박재형
status: Draft
version: 1.0
created: 2026-01-06
updated: 2026-01-06
parent: PRC-planops-001
---

## 1. 개요

### 1.1 목적

Cycles DB는 **기간 컨테이너**로서 분기/월/스프린트 등의 운영 기간을 관리한다. 모든 표준 뷰의 필터 기준이 되는 핵심 DB이다.

### 1.2 핵심 원칙

- **Active Cycle은 항상 1개만** 유지
- 모든 하위 DB(Phases, Ops Tasks)는 Cycle Status를 Rollup하여 필터링

---

## 2. 속성 정의

### 2.1 필수 속성

| 속성명 | 타입 | 필수 | 설명 | 예시 |
|--------|------|------|------|------|
| 이름 | Title | O | 기간 이름 | 2026 Q1, 2026-01 |
| 시작일 | Date | O | 기간 시작일 | 2026-01-01 |
| 종료일 | Date | O | 기간 종료일 | 2026-03-31 |
| 상태 | Select | O | 기간 상태 | 예정/진행중/완료 |
| 책임자 | Person | O | 기간 운영 책임자 (단일) | 박재형 |

### 2.2 권장 속성

| 속성명 | 타입 | 설명 | 예시 |
|--------|------|------|------|
| 운영 단위 | Select | 기간 유형 | Quarter/Month/Sprint |
| 목표 | Text | 기간의 최종 성과 목표 | "신규 기능 3개 릴리즈" |
| 특이사항 | Text | 특이사항 | "휴가 시즌 고려" |

### 2.3 자동 속성 (Rollup/Formula)

| 속성명 | 타입 | 계산 방식 | 용도 |
|--------|------|----------|------|
| Phase 갯수 | Rollup | Count(Phases) | 기간 내 Phase 수 |
| Initiative 갯수 | Rollup | Count(Initiatives) | 기간 내 Initiative 수 |
| 진행률 | Formula | Done Phases / Total Phases | 진행률 표시 |

---

## 3. 상태 정의

| 상태 | 설명 | 전환 조건 |
|------|------|----------|
| **예정** | 다음 기간으로 예정됨 | 기본 생성 상태 |
| **진행중** | 현재 운영 중 | Cycle Kickoff 시 전환 (1개만 유지) |
| **완료** | 기간 종료 | Cycle Close 시 전환 |

### 3.1 상태 전환 규칙

```
예정 → 진행중 : Cycle Kickoff (기존 진행중을 완료로 먼저 전환)
진행중 → 완료 : Cycle Close
완료 → (변경 불가)
```

---

## 4. 관계 (Relations)

| 대상 DB | 관계 유형 | 설명 |
|---------|----------|------|
| Phases | 1:N | 이 Cycle에 속한 Phase 목록 |
| Initiatives | 1:N | 이 Cycle에 속한 Initiative 목록 |

---

## 5. 표준 뷰

### 5.1 Active Cycle

| 항목 | 설정 |
|------|------|
| 뷰 타입 | Gallery 또는 Table |
| 필터 | 상태 = 진행중 |
| 정렬 | Start (Desc) |
| 목적 | 현재 운영 기준 확인 |

### 5.2 Planning Queue

| 항목 | 설정 |
|------|------|
| 뷰 타입 | Table |
| 필터 | 상태 = 예정 |
| 정렬 | Start (Asc) |
| 목적 | 다음 기간 준비 |

### 5.3 Archive

| 항목 | 설정 |
|------|------|
| 뷰 타입 | Table |
| 필터 | 상태 = 완료 |
| 정렬 | End (Desc) |
| 목적 | 종료된 기간 기록 |

---

## 6. 운영 규칙

### 6.1 필수 규칙

1. **Active Cycle 단일 유지**: 진행중 상태는 항상 1개만 존재
2. **Owner 필수**: 모든 Cycle은 반드시 단일 Owner 지정
3. **기간 명시**: Start/End 날짜 필수 입력

### 6.2 네이밍 규칙

| Cadence | 형식 | 예시 |
|---------|------|------|
| Quarter | YYYY Q# | 2026 Q1 |
| Month | YYYY-MM | 2026-01 |
| Sprint | Sprint #-YYYY | Sprint 1-2026 |

---

## 7. 자동화 (권장)

### 7.1 Notion Automation

| 트리거 | 액션 | 설명 |
|--------|------|------|
| 상태 → 진행중 | 기존 진행중 → 완료 | 중복 진행중 방지 |
| 상태 → 완료 | 알림 발송 | Cycle 종료 안내 |

---

## 8. 관련 문서

| docId | 제목 | 관계 |
|-------|------|------|
| PRC-planops-001 | 운영/진척 관리 프로세스 | 상위 프로세스 |
| DBS-notion-phases-001 | Phases DB 스펙 | 하위 DB |
| DBS-notion-initiatives-001 | Initiatives DB 스펙 | 하위 DB |

---

## 변경 이력

| 버전 | 날짜 | 변경 내용 | 작성자 |
|------|------|----------|--------|
| 1.0 | 2026-01-06 | 최초 작성 | 박재형 |
