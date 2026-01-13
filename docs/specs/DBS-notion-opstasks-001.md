# Notion DB 스펙 - Ops Tasks

---
docId: DBS-notion-opstasks-001
title: Notion DB 스펙 - Ops Tasks
owner: 박재형
status: Draft
version: 1.0
created: 2026-01-06
updated: 2026-01-06
parent: PRC-planops-001
---

## 1. 개요

### 1.1 목적

Ops Tasks DB는 **주간 실행 티켓**을 관리한다. Phase를 실제 수행 가능한 단위로 분해하여 개인이 실행하는 작업을 추적한다.

### 1.2 핵심 원칙

- 모든 Task는 **Phase에 연결** 필수
- 1주일 내 완료 가능한 **실행 가능한 단위**로 분해
- **단일 Owner** 필수

---

## 2. 속성 정의

### 2.1 필수 속성

| 속성명 | 타입 | 필수 | 설명 | 예시 |
|--------|------|------|------|------|
| Task명 | Title | O | 실행 가능한 작업 | PRD 초안 작성, API 엔드포인트 개발 |
| Phase | Relation → Phases | O | 소속 Phase | [Spec] 결제 기능 |
| 상태 | Select | O | 진행 상태 | Backlog/This Week/In Progress/Review Needed/Done/Parking |
| 책임자 | Person | O | 실행자 (단일) | 박재형 |
| 마감일 | Date | O | 완료 목표일 | 2026-01-10 |

### 2.2 권장 속성

| 속성명 | 타입 | 설명 | 예시 |
|--------|------|------|------|
| 산출물 docIds | Text | SSOT 연결 키 | PRD-payment-001 |
| DoD 체크 | Checkbox | 완료 정의 충족 여부 | ✓ |
| SSOT 링크 | URL | Obsidian 문서 링크 | obsidian://... |
| 리뷰어 | Person | 리뷰 담당자 | 김철수 |
| 예상 시간 | Number | 작업 예상 시간(h) | 4 |

### 2.3 Rollup 속성

| 속성명 | 타입 | 계산 경로 | 용도 |
|--------|------|----------|------|
| 사이클 상태 | Rollup | Task → Phase → Cycle → Status | 표준 뷰 필터용 |
| Phase 유형 | Rollup | Task → Phase → Phase 유형 | 그룹핑용 |

---

## 3. 상태 정의

| 상태 | 설명 | 사용 시점 |
|------|------|----------|
| **Backlog** | 대기 중 | 아직 착수 예정 아님 |
| **This Week** | 이번 주 실행 | 이번 주 선정됨 |
| **In Progress** | 진행 중 | 실제 작업 중 |
| **Review Needed** | 리뷰 대기 | 작업 완료, 리뷰 필요 |
| **Done** | 완료 | DoD 충족 + 리뷰 통과 |
| **Parking** | 보류 | 일시 중단 (사유 명시) |

### 3.1 상태 전환 흐름

```
Backlog → This Week : 주간 계획 선정 (월요일)
This Week → In Progress : 작업 착수
In Progress → Review Needed : 작업 완료, 리뷰 요청
Review Needed → Done : 리뷰 통과
Review Needed → In Progress : 리뷰 피드백 반영
* → Parking : 보류 결정
Parking → Backlog/This Week : 보류 해제
```

---

## 4. 관계 (Relations)

| 대상 DB | 관계 유형 | 필수 | 설명 |
|---------|----------|------|------|
| Phases | N:1 | O | 소속 Phase |

### 4.1 Rollup 경로

```
Ops Tasks → Phase → Cycle → Status (3단계)
```

---

## 5. 표준 뷰

### 5.1 주간 실행 (Kanban)

| 항목 | 설정 |
|------|------|
| 뷰 타입 | Kanban (Board) |
| 필터 | 사이클 상태 = Active AND (This Week OR In Progress OR Review Needed) |
| Group By | 상태 |
| 정렬 | 마감일 (Asc) |
| 목적 | 이번 주 집중 작업 |

### 5.2 리뷰 대기 (Table)

| 항목 | 설정 |
|------|------|
| 뷰 타입 | Table |
| 필터 | 상태 = Review Needed |
| 정렬 | 마감일 (Asc) |
| 목적 | 리뷰 처리 대기 목록 |

### 5.3 내 할 일 (Table)

| 항목 | 설정 |
|------|------|
| 뷰 타입 | Table |
| 필터 | 책임자 = Me AND 상태 ≠ Done |
| 정렬 | 마감일 (Asc) |
| 목적 | 개인 작업 목록 |

### 5.4 전체 Backlog (Table)

| 항목 | 설정 |
|------|------|
| 뷰 타입 | Table |
| 필터 | 사이클 상태 = Active AND 상태 = Backlog |
| 정렬 | Phase, 마감일 |
| 목적 | 전체 대기 작업 확인 |

---

## 6. 운영 규칙

### 6.1 필수 규칙

1. **Phase 연결 필수**: 독립 Task 금지, 반드시 Phase에 연결
2. **단일 Owner**: 공동 Owner 금지
3. **마감일 필수**: 완료 목표일 입력 필수
4. **적정 크기**: 1주일 내 완료 가능한 단위로 분해

### 6.2 주간 운영 (Task 선정)

| 요일 | 활동 | 기준 |
|------|------|------|
| 월 | This Week 선정 | 최대 5~7개/인 |
| 수 | Blocked/Review 해소 | In Progress 막힘 확인 |
| 금 | 주간 리뷰 | 다음 주 Top3 확정 |

### 6.3 DoD (완료 정의)

Task를 Done으로 전환하기 전 확인:
- [ ] 산출물 존재 (코드, 문서 등)
- [ ] docId 연결 (해당 시)
- [ ] 리뷰 완료 (해당 시)
- [ ] 다음 단계에 필요한 정보 전달 완료

---

## 7. 자동화 (권장)

### 7.1 Notion Automation

| 트리거 | 액션 | 설명 |
|--------|------|------|
| 상태 → Review Needed | 리뷰어에게 알림 | 리뷰 요청 안내 |
| 마감일 도래 | 책임자에게 알림 | 마감 임박 안내 |
| 상태 → Done | DoD 체크 확인 | 완료 정의 충족 확인 |

---

## 8. 관련 문서

| docId | 제목 | 관계 |
|-------|------|------|
| PRC-planops-001 | 운영/진척 관리 프로세스 | 상위 프로세스 |
| DBS-notion-phases-001 | Phases DB 스펙 | 상위 DB |

---

## 변경 이력

| 버전 | 날짜 | 변경 내용 | 작성자 |
|------|------|----------|--------|
| 1.0 | 2026-01-06 | 최초 작성 | 박재형 |
