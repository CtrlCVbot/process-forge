# Notion DB 스펙 - Phases

---
docId: DBS-notion-phases-001
title: Notion DB 스펙 - Phases
owner: 박재형
status: Draft
version: 1.0
created: 2026-01-06
updated: 2026-01-06
parent: PRC-planops-001
---

## 1. 개요

### 1.1 목적

Phases DB는 **마스터 플랜의 본체**로서 Spec/Dev/QA/Release 등 실제 일정이 있는 작업 단계를 관리한다. 기획+개발+QA+릴리즈를 한 화면에서 확인하는 핵심 DB이다.

### 1.2 핵심 원칙

- Phase는 **Cycle에 직접 연결** (3단계 구조)
- Initiative는 **선택적 연결** (큰 계획 묶음이 필요할 때만)
- 모든 Phase는 **단일 Owner** 필수

---

## 2. 속성 정의

### 2.1 필수 속성

| 속성명 | 타입 | 필수 | 설명 | 예시 |
|--------|------|------|------|------|
| Phase명 | Title | O | [유형] 작업명 - 목적 | [Spec] 결제 기능 - PRD 작성 |
| 사이클 | Relation → Cycles | O | 소속 기간 (직접 연결) | 2026 Q1 |
| 이니셔티브 | Relation → Initiatives | - | 소속 계획 **(선택적)** | 결제 시스템 개선 |
| Phase 유형 | Select | O | 단계 유형 | Spec/Dev/QA/Release |
| 시작일 | Date | O | 일정 시작 | 2026-01-06 |
| 종료일 | Date | O | 일정 종료 | 2026-01-20 |
| 상태 | Select | O | 진행 상태 | 미시작/진행중/막힘/완료 |
| 책임자 | Person | O | Phase Owner (단일) | 박재형 |
| Gate 상태 | Select | O | 다음 단계 진입 조건 | Ready/Not Ready |
| 산출물 docIds | Text | O | SSOT 연결 키 목록 | PRD-payment-001, QAP-payment-001 |

### 2.2 Rollup 속성

| 속성명 | 타입 | 계산 경로 | 용도 |
|--------|------|----------|------|
| 사이클 상태 | Rollup | Phase → Cycle → Status | 표준 뷰 필터용 (2단계) |

### 2.3 운영 안정화 속성 (권장)

| 속성명 | 타입 | 설명 |
|--------|------|------|
| 막힘 사유 유형 | Select | 의존성/리소스/외부/기술부채/기타 |
| 막힘 해소 책임자 | Person | 누가 풀지 명확화 |
| 막힘 해소 목표일 | Date | 언제까지 풀지 |
| 의존 Phase | Relation → Phases | 의존성 추적 |
| 리플랜 변경 여부 | Checkbox | Re-plan에서 변경됨 표시 |
| SSOT 링크 | URL | Obsidian 문서 링크 |

---

## 3. 상태 정의

| 상태 | 설명 | 전환 조건 |
|------|------|----------|
| **미시작** (Not Started) | 아직 시작하지 않음 | 기본 생성 상태 |
| **진행중** (In Progress) | 작업 진행 중 | 시작일 도래 + 착수 |
| **막힘** (Blocked) | 진행 불가 상태 | 의존성/리소스 이슈 발생 |
| **완료** (Done) | 작업 완료 | DoD 충족 |

### 3.1 Phase Type별 DoD (완료 정의)

| Phase Type | 완료 조건 |
|------------|----------|
| Spec | Gate Ready 통과 |
| Dev | main merge + feature flag + 로그/모니터링 포인트 |
| QA | 핵심 TC 통과 + Sev1/2 버그 없음 |
| Release | 배포 완료 + 모니터링 확인 + 롤백 방법 확인 |

---

## 4. Gate 상태 정의

| Gate 상태 | 설명 | 적용 Phase |
|-----------|------|-----------|
| **Ready** | 다음 단계 진입 조건 충족 | Spec → Dev 전환 시 |
| **Not Ready** | 조건 미충족 | 기본 상태 |

### 4.1 Spec Gate 체크리스트

- [ ] PRD docId 존재 (SSOT 링크 연결됨)
- [ ] 상태/권한/예외 시나리오 포함
- [ ] 데이터 영향(필드/검증 규칙) 명시
- [ ] QA 핵심 TC 또는 QAP docId 링크 존재
- [ ] Tech Owner 리뷰 완료

---

## 5. 관계 (Relations)

| 대상 DB | 관계 유형 | 필수 | 설명 |
|---------|----------|------|------|
| Cycles | N:1 | O | 소속 기간 (직접 연결) |
| Initiatives | N:1 | - | 소속 계획 (선택적) |
| Ops Tasks | 1:N | - | 이 Phase의 실행 티켓 |
| Phases (Self) | N:N | - | 의존 Phase |

### 5.1 Rollup 경로 (개선됨)

```
AS-IS: Phase → Initiative → Cycle → Status (3단계)
TO-BE: Phase → Cycle → Status (2단계)
```

---

## 6. 표준 뷰

### 6.1 마스터 플랜 (Timeline)

| 항목 | 설정 |
|------|------|
| 뷰 타입 | Timeline |
| 필터 | 사이클 상태 = Active |
| Group By | Phase 유형 (Spec/Dev/QA/Release) |
| 날짜 범위 | 시작일 ~ 종료일 |
| 목적 | 기획+개발+QA+Release 한 화면 확인 |

### 6.2 Spec 파이프라인 (Board)

| 항목 | 설정 |
|------|------|
| 뷰 타입 | Board |
| 필터 | 사이클 상태 = Active AND Phase 유형 = Spec |
| Group By | 상태 |
| 목적 | 기획 진행 현황 |

### 6.3 Dev 타임라인 (Timeline)

| 항목 | 설정 |
|------|------|
| 뷰 타입 | Timeline |
| 필터 | 사이클 상태 = Active AND Phase 유형 = Dev |
| 목적 | 개발 일정 확인 |

### 6.4 막힘 레지스터 (Table)

| 항목 | 설정 |
|------|------|
| 뷰 타입 | Table |
| 필터 | 상태 = 막힘 |
| 정렬 | 막힘 해소 목표일 (Asc) |
| 목적 | 병목 해소 추적 |

---

## 7. 운영 규칙

### 7.1 필수 규칙

1. **Cycle 직접 연결**: 모든 Phase는 Cycle에 직접 연결 필수
2. **단일 Owner**: 공동 Owner 금지, 반드시 1명 지정
3. **일정 필수**: 시작일/종료일 입력 필수 (마스터 플랜 가시화)
4. **Gate 준수**: Spec Gate 미충족 시 Dev 진행중 전환 금지

### 7.2 네이밍 규칙

| 형식 | 예시 |
|------|------|
| [유형] 작업명 - 목적 | [Spec] 결제 기능 - PRD 작성 |
| [유형] 작업명 | [Dev] 결제 API 개발 |

---

## 8. 자동화 (권장)

### 8.1 Notion Automation

| 트리거 | 액션 | 설명 |
|--------|------|------|
| 상태 → 막힘 | 알림 발송 | 막힘 발생 안내 |
| 종료일 도래 | 알림 발송 | 마감 임박 안내 |
| Gate → Ready | 알림 발송 | Dev 착수 가능 안내 |

---

## 9. 관련 문서

| docId | 제목 | 관계 |
|-------|------|------|
| PRC-planops-001 | 운영/진척 관리 프로세스 | 상위 프로세스 |
| DBS-notion-cycles-001 | Cycles DB 스펙 | 상위 DB |
| DBS-notion-opstasks-001 | Ops Tasks DB 스펙 | 하위 DB |
| DBS-notion-initiatives-001 | Initiatives DB 스펙 | 참조 DB |

---

## 변경 이력

| 버전 | 날짜 | 변경 내용 | 작성자 |
|------|------|----------|--------|
| 1.0 | 2026-01-06 | 최초 작성 | 박재형 |
