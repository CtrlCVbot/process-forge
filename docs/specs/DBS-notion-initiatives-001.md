# Notion DB 스펙 - Initiatives

---
docId: DBS-notion-initiatives-001
title: Notion DB 스펙 - Initiatives
owner: 박재형
status: Draft
version: 1.0
created: 2026-01-06
updated: 2026-01-06
parent: PRC-planops-001
---

## 1. 개요

### 1.1 목적

Initiatives DB는 **큰 계획 묶음(포트폴리오)**을 관리한다. 여러 Phase를 하나의 목표로 묶어 관리할 때 사용한다.

### 1.2 핵심 원칙

- **선택적 사용**: 작은 작업은 Initiative 없이 Phase만으로 운영 가능
- Phase가 Initiative를 참조하는 구조 (Initiative → Phase 역참조)
- PM Owner, Tech Owner **각각 단일** 지정

---

## 2. 언제 사용하는가?

| 상황 | Initiative 사용 |
|------|-----------------|
| 여러 Phase를 묶어 관리해야 할 때 | O |
| 분기/월 단위 큰 목표가 있을 때 | O |
| 단순 작업, 버그 수정, 작은 개선 | X (Phase만으로 충분) |
| 독립적인 단일 Phase | X |

### 2.1 사용 예시

**Initiative 사용 O**:
- "결제 시스템 개선" - Spec, Dev, QA, Release 4개 Phase 포함
- "신규 대시보드 개발" - 여러 기능 Phase 묶음

**Initiative 사용 X**:
- "버그 수정: 로그인 오류" - 단일 Dev Phase
- "문서 업데이트" - 단일 작업

---

## 3. 속성 정의

### 3.1 필수 속성

| 속성명 | 타입 | 필수 | 설명 | 예시 |
|--------|------|------|------|------|
| 이니셔티브명 | Title | O | 결과가 드러나게 작성 | 정산 수동조정 120→40 감소 |
| 사이클 | Relation → Cycles | O | 소속 기간 | 2026 Q1 |
| 상태 | Select | O | 진행 상태 | 제안됨/확정/진행중/릴리즈됨/제외 |
| 우선순위 | Select | O | 중요도 | P0/P1/P2/P3 |
| 목표(Outcome) | Text | O | 측정 가능한 결과 | "수동조정 건수 60% 감소" |
| 기획 책임자 | Person | O | PM Owner (단일) | 박재형 |
| 기술 책임자 | Person | O | Tech Owner (단일) | 김철수 |
| 메인 docId | Text | O | SSOT 진입점 (PRD) | PRD-payment-001 |

### 3.2 권장 속성

| 속성명 | 타입 | 설명 | 예시 |
|--------|------|------|------|
| 리스크 | Text | 주요 리스크 | "외부 API 의존성" |
| 의존성 | Text | 외부 의존성 | "결제사 연동 완료 필요" |
| 비고 | Text | 특이사항 | "3월 결제사 점검 기간 고려" |
| SSOT 링크 | URL | Obsidian 문서 링크 | obsidian://... |

### 3.3 Rollup 속성

| 속성명 | 타입 | 계산 경로 | 용도 |
|--------|------|----------|------|
| 사이클 상태 | Rollup | Initiative → Cycle → Status | 표준 뷰 필터용 |
| Phase 수 | Rollup | Count(관련 Phases) | 규모 파악 |
| 완료 Phase | Rollup | Count(완료 Phases) | 진행률 |

### 3.4 대시보드용 속성

Executive Dashboard에서 Initiative 기준 현황 표시를 위한 추가 속성.

| 속성명 | 타입 | 설명 | 계산 방식 |
|--------|------|------|----------|
| 막힘 Phase 수 | Rollup | 막힘 상태 Phase 수 | Phases (상태=막힘) → Count |
| 시작일 | Rollup | 최초 Phase 시작일 | Phases → 시작일 → Min |
| 종료일 | Rollup | 최종 Phase 종료일 | Phases → 종료일 → Max |
| 진행률 | Formula | Phase 기준 진행률 | round(완료 Phase / Phase 수 * 100) |
| 진행률 Bar | Formula | Progress Bar 시각화 | (3.4.2 참조) |
| 상태 아이콘 | Formula | 진행 상태 시각화 | (3.4.3 참조) |
| 지연 사유 | Text | At Risk/Delayed 사유 | 수동 입력 |

#### 3.4.1 진행률 Formula

```notion
// 진행률 계산 (숫자)
if(prop("Phase 수") > 0,
  round(prop("완료 Phase") / prop("Phase 수") * 100),
  0)
```

#### 3.4.2 진행률 Bar Formula

```notion
// Progress Bar 시각화
if(prop("Phase 수") > 0,
  slice("██████████", 0, round(prop("진행률") / 10)) +
  slice("░░░░░░░░░░", 0, 10 - round(prop("진행률") / 10)) +
  " " + format(prop("진행률")) + "%",
  "N/A")
```

#### 3.4.3 상태 아이콘 Formula

```notion
// 상태 아이콘 (On Track / At Risk / Delayed / 완료 / 제외 / 예정)
if(prop("상태") == "릴리즈됨", "✅ 완료",
  if(prop("상태") == "제외", "⛔ 제외",
    if(prop("상태") == "제안됨", "⚪ 예정",
      if(or(prop("상태") == "확정", prop("상태") == "진행중"),
        if(prop("막힘 Phase 수") > 0, "🔴 Delayed",
          if(prop("진행률") >= 50, "🟢 On Track", "🟡 At Risk")),
        "⚪ 예정"))))
```

**상태 아이콘 의미**:

| 아이콘 | 상태 | 조건 |
|--------|------|------|
| ✅ 완료 | 릴리즈됨 | 상태 = 릴리즈됨 |
| ⛔ 제외 | 제외 | 상태 = 제외 |
| 🟢 On Track | 순조로움 | 진행률 >= 50% AND 막힘 없음 |
| 🟡 At Risk | 주의 | 진행률 < 50% AND 막힘 없음 |
| 🔴 Delayed | 지연 | 막힘 Phase 존재 |
| ⚪ 예정 | 미시작 | 상태 = 제안됨 또는 기타 |

---

## 4. 상태 정의

| 상태 | 설명 | 사용 시점 |
|------|------|----------|
| **제안됨** (Proposed) | 후보 상태 | 초기 등록 |
| **확정** (Committed) | 이번 사이클 필수 수행 | Kickoff에서 확정 |
| **진행중** (In Progress) | Spec/Dev 실제 진행 | Phase 착수 |
| **릴리즈됨** (Released) | 실제 배포/반영 완료 | Release Gate 통과 |
| **제외** (Dropped) | 이번 사이클 수행 안 함 | Re-plan에서 결정 |

### 4.1 상태 전환 흐름

```
제안됨 → 확정 : Cycle Kickoff에서 결정
제안됨 → 제외 : 이번 사이클 불가 판단
확정 → 진행중 : Spec Phase 착수
진행중 → 릴리즈됨 : Release Gate 통과
진행중 → 제외 : Re-plan에서 Drop 결정
```

---

## 5. 관계 (Relations)

| 대상 DB | 관계 유형 | 필수 | 설명 |
|---------|----------|------|------|
| Cycles | N:1 | O | 소속 기간 |
| Phases | 1:N (역참조) | - | 이 Initiative의 Phase 목록 |

### 5.1 Phase에서 참조

Phases DB에서 Initiative를 **선택적으로** 참조:
- Initiative 연결 시: 포트폴리오 관점 관리
- Initiative 미연결 시: 독립 Phase로 운영

---

## 6. 표준 뷰

### 6.1 포트폴리오 (Table)

| 항목 | 설정 |
|------|------|
| 뷰 타입 | Table |
| 필터 | 사이클 상태 = Active |
| 정렬 | 우선순위 (Asc), 상태 |
| 표시 속성 | 이니셔티브명, 상태, 우선순위, 기획 책임자, 기술 책임자 |
| 목적 | 전체 Initiative 목록 확인 |

### 6.2 상태 보드 (Board)

| 항목 | 설정 |
|------|------|
| 뷰 타입 | Board |
| 필터 | 사이클 상태 = Active |
| Group By | 상태 |
| 목적 | 의사결정 흐름 확인 |

### 6.3 리스크 레지스터 (Table)

| 항목 | 설정 |
|------|------|
| 뷰 타입 | Table |
| 필터 | 리스크 is not empty |
| 정렬 | 우선순위 (Asc) |
| 목적 | 리스크 있는 Initiative 모아보기 |

### 6.4 우선순위별 (Board)

| 항목 | 설정 |
|------|------|
| 뷰 타입 | Board |
| 필터 | 사이클 상태 = Active |
| Group By | 우선순위 |
| 목적 | 우선순위별 배치 확인 |

---

## 7. 운영 규칙

### 7.1 필수 규칙

1. **Cycle 연결 필수**: 모든 Initiative는 Cycle에 연결
2. **이중 Owner**: 기획 책임자 + 기술 책임자 각각 단일 지정
3. **결과 중심 네이밍**: 결과가 드러나게 작성
4. **docId 필수**: 메인 PRD docId 연결

### 7.2 네이밍 규칙

| 좋은 예 | 나쁜 예 |
|---------|---------|
| 정산 수동조정 120→40 감소 | 정산 고도화 |
| 결제 성공률 99%→99.5% 개선 | 결제 개선 |
| 신규 대시보드 출시 | 대시보드 작업 |

### 7.3 우선순위 기준

| 우선순위 | 설명 | 기준 |
|----------|------|------|
| P0 | 필수 | 이번 사이클 반드시 완료 |
| P1 | 높음 | 가능한 완료 목표 |
| P2 | 보통 | 여유 있으면 진행 |
| P3 | 낮음 | 다음 사이클 후보 |

---

## 8. 자동화 (권장)

### 8.1 Notion Automation

| 트리거 | 액션 | 설명 |
|--------|------|------|
| 상태 → 제외 | 알림 발송 | Drop 결정 안내 |
| 상태 → 릴리즈됨 | 알림 발송 | 완료 축하 안내 |
| 새 Initiative 생성 | 기본 상태 = 제안됨 | 초기 상태 설정 |

---

## 9. 관련 문서

| docId | 제목 | 관계 |
|-------|------|------|
| PRC-planops-001 | 운영/진척 관리 프로세스 | 상위 프로세스 |
| DBS-notion-cycles-001 | Cycles DB 스펙 | 상위 DB |
| DBS-notion-phases-001 | Phases DB 스펙 | 참조 DB (역참조) |

---

## 변경 이력

| 버전 | 날짜 | 변경 내용 | 작성자 |
|------|------|----------|--------|
| 1.0 | 2026-01-06 | 최초 작성 | 박재형 |
| 1.1 | 2026-01-07 | 대시보드용 속성 추가 (진행률, 시작일/종료일, 상태 아이콘) | 박재형 |
