# 운영/진척 관리 프로세스 (Active Cycle 기반)

---
docId: PRC-planops-001
title: 운영/진척 관리 프로세스
owner: 박재형
deputy: (부책임자)
status: Draft
version: 1.0
created: 2026-01-06
updated: 2026-01-06
---

## 1. 목적

이 프로세스의 목적은 **Notion=운영/데이터**, **Obsidian=SSOT/버전** 원칙 아래에서 팀의 분기/월 계획을 **마스터 플랜 한 장**으로 운영하고, 기획(Spec) + 개발(Dev) + QA + Release 일정을 **Active Cycle 기반**으로 일관되게 관리하는 것입니다.

---

## 2. 적용 범위

### 2.1 적용 대상

| 구분 | 대상 |
|------|------|
| 조직 | 전체 프로젝트 팀 |
| 역할 | Cycle Owner, PM/PO, Tech Owner, QA Owner, Buddy/Reviewer |
| 상황 | 분기/월/스프린트 등 기간(Cycle) 기반 운영 전반 |

### 2.2 적용 제외

- 문서 본문(정책/PRD/상세 프로세스)의 Notion 복제
- 비정기적 긴급 운영 대응 (별도 에스컬레이션 프로세스 적용)

---

## 3. 입력과 출력

### 3.1 입력 (Input)

| 입력 항목 | 설명 | 제공자 |
|----------|------|--------|
| 기간 계획 | 분기/월 단위 계획 정보 | Cycle Owner |
| Initiative 제안 | 큰 계획 항목 (선택적) | PM/PO |
| Spec 산출물 | PRD, QAP 등 docId 연결 문서 | PM/PO |

### 3.2 출력 (Output)

| 출력 항목 | 설명 | 수신자 |
|----------|------|--------|
| 마스터 플랜 | 전체 Phase 일정 Timeline | 전체 팀 |
| 주간 실행 티켓 | Ops Tasks Kanban | 실행 담당자 |
| 릴리즈 완료 | Release Gate 통과 결과 | 이해관계자 |

---

## 4. 역할 (RACI)

| 활동 | R (실행) | A (책임) | C (협의) | I (통보) |
|------|---------|---------|---------|---------|
| Cycle Kickoff | Cycle Owner | Cycle Owner | PM/PO, Tech Owner | 전체 |
| Initiative 정의 | PM/PO | PM/PO | Tech Owner | Cycle Owner |
| Phase 생성/관리 | Phase Owner | Phase Owner | 관련 역할 | Cycle Owner |
| Spec Gate 점검 | PM/PO | Tech Owner | QA Owner | Cycle Owner |
| 주간 리뷰 | Cycle Owner | Cycle Owner | 전체 | - |
| Re-plan | Cycle Owner | Cycle Owner | PM/PO, Tech Owner | 전체 |
| Release Gate 검증 | QA Owner | QA Owner | Tech Owner | 전체 |

**역할 정의**:
- **R (Responsible)**: 실제 작업 수행자
- **A (Accountable)**: 최종 책임자 (1명만)
- **C (Consulted)**: 사전 협의 대상
- **I (Informed)**: 결과 통보 대상

---

## 5. 프로세스 흐름

### 5.1 개요 다이어그램

```
[Cycle Kickoff] → [계획 편성] → [Spec Phase] → [Spec Gate] → [Dev Phase]
                                                    ↓
[Cycle Close] ← [Release Gate] ← [QA Phase] ← [Dev 완료]
       ↓
   [Re-plan] (월 1회)
```

### 5.2 상세 단계

#### 단계 1: Cycle Kickoff (기간 시작)

| 항목 | 내용 |
|------|------|
| **수행자** | Cycle Owner |
| **트리거** | 새 기간(분기/월) 시작 |
| **활동** | Active Cycle 전환, Initiative/Phase 생성 |
| **산출물** | Active Cycle 1개 확정, 마스터 플랜 |
| **완료 조건** | Active Cycle 1개만 존재 확인 |

**상세 절차**:
1. Cycles DB에서 이번 기간 `Status = Active` 설정
2. 기존 Active는 `Done`으로 전환
3. Initiatives 등록 (확정/옵션 구분)
4. 각 Initiative/독립 작업에 Phase 생성 (Spec/Dev/QA/Release)
5. Phase Start/End 입력 완료
6. 마스터 플랜 Timeline 노출 확인

#### 단계 2: Spec Phase (기획 단계)

| 항목 | 내용 |
|------|------|
| **수행자** | PM/PO |
| **트리거** | Cycle Kickoff 완료 |
| **활동** | PRD 작성, 시나리오 정의, 데이터 영향 분석 |
| **산출물** | PRD(docId), QAP(docId) |
| **완료 조건** | Spec Gate Ready |

**상세 절차**:
1. PRD 작성 및 docId 부여
2. 상태/권한/예외 시나리오 포함
3. 데이터 영향(필드/검증 규칙) 명시
4. QA 핵심 TC 또는 QAP docId 링크
5. Tech Owner 리뷰 요청

#### 단계 3: Spec Gate (품질 관문)

| 항목 | 내용 |
|------|------|
| **수행자** | Tech Owner |
| **트리거** | Spec Phase 완료 요청 |
| **활동** | Gate 조건 점검, Ready 판정 |
| **산출물** | Gate 상태 = Ready |
| **완료 조건** | 모든 Gate 체크리스트 충족 |

**Spec Gate 체크리스트**:
- [ ] PRD docId 존재 (SSOT 링크 연결됨)
- [ ] 상태/권한/예외 시나리오 포함
- [ ] 데이터 영향(필드/검증 규칙) 명시
- [ ] QA 핵심 TC 또는 QAP docId 링크 존재
- [ ] Tech Owner 리뷰 완료

#### 단계 4: Dev Phase (개발 단계)

| 항목 | 내용 |
|------|------|
| **수행자** | Tech Owner/개발자 |
| **트리거** | Spec Gate Ready |
| **활동** | 개발, 코드 리뷰, 머지 |
| **산출물** | main merge, feature flag, 로그/모니터링 포인트 |
| **완료 조건** | Dev DoD 충족 |

**상세 절차**:
1. Gate Ready 확인 후 Dev Phase 진행중 전환
2. 개발 수행
3. 코드 리뷰 완료
4. main branch merge
5. feature flag 설정 (필요 시)
6. 로그/모니터링 포인트 추가

#### 단계 5: QA Phase (품질 검증)

| 항목 | 내용 |
|------|------|
| **수행자** | QA Owner |
| **트리거** | Dev Phase 완료 |
| **활동** | TC 실행, 버그 리포트, 재검증 |
| **산출물** | QA 리포트, 버그 목록 |
| **완료 조건** | 핵심 TC 통과, Sev1/2 버그 없음 |

#### 단계 6: Release Gate (릴리즈 관문)

| 항목 | 내용 |
|------|------|
| **수행자** | QA Owner |
| **트리거** | QA Phase 완료 |
| **활동** | Release 조건 점검 |
| **산출물** | Release 승인/보류 결정 |
| **완료 조건** | Release Gate 체크리스트 충족 |

**Release Gate 체크리스트**:
- [ ] QA 완료 (핵심 TC 통과, Sev1/2 없음)
- [ ] 모니터링/로그 포인트 존재
- [ ] 롤백 플랜 존재
- [ ] 릴리즈 노트(docId) 링크

#### 단계 7: Cycle Close (기간 종료)

| 항목 | 내용 |
|------|------|
| **수행자** | Cycle Owner |
| **트리거** | 기간 종료 |
| **활동** | 상태 정리, 회고, Done 전환 |
| **산출물** | 회고 요약, 다음 Cycle 후보 |
| **완료 조건** | Cycle Status = Done |

**상세 절차**:
1. Initiative 상태 정리: 릴리즈됨/이월/제외
2. Carry-over 항목은 다음 Cycle 후보로 이관
3. Cycle Status를 Done으로 전환
4. 회고 요약 (잘된 점/막힌 점/다음 개선)

---

## 6. 예외 처리

### 6.1 예외 상황

| 예외 상황 | 발생 조건 | 처리 방법 |
|----------|----------|----------|
| Gate 미충족 Dev 착수 요청 | 긴급 일정 압박 | Cycle Owner 승인 + ADR 기록, 기술 부채 등록 |
| Active Cycle 2개 이상 | 기간 전환 누락 | 즉시 정리, 1개만 Active 유지 |
| Spec 변경 (Dev 진행 중) | 요구사항 변경 | CHG 프로세스로 처리, Re-plan 슬롯에서 일정 조정 |

### 6.2 에스컬레이션

| 수준 | 조건 | 대상 | 대응 시간 |
|------|------|------|----------|
| Level 1 | Phase Blocked 3일 이상 | Phase Owner | 1일 내 |
| Level 2 | Phase Blocked 1주 이상 | Cycle Owner | 2일 내 |
| Level 3 | Cycle 전체 위험 | 상위 의사결정자 | 즉시 |

---

## 7. 운영 루프

### 7.1 주간 운영

| 요일 | 시간 | 활동 |
|------|------|------|
| 월 | 15분 | 이번 주 Ops Tasks 선정 (최대 5~7개) |
| 수 | 15분 | Blocked/Review Needed 해소 |
| 금 | 30분 | 주간 리뷰 (고정) |

### 7.2 금요일 리뷰 아젠다

1. **마스터 플랜**: Blocked Phase 확인
2. **Spec 파이프라인**: Gate 미충족 원인/지원 필요 확인
3. **Dev 타임라인**: 일정 충돌/과부하 확인
4. **Review Needed**: 리뷰 대기 티켓 처리
5. **다음 주 Top3** 확정 + 리스크/의존성 업데이트

### 7.3 Re-plan (월 1회, 첫째 주 금요일)

**수행 내용**:
- Dropped/Defer 결정
- Phase 일정 재배치 (Start/End 수정)
- 확정 ↔ 옵션 이동
- 주요 의사결정 ADR 기록

---

## 8. 측정 지표 (KPI)

| 지표 | 정의 | 목표 | 측정 주기 |
|------|------|------|----------|
| 계획 대비 완료율 | Committed 중 Released 비율 | 80% 이상 | Cycle 종료 |
| Carry-over 비율 | 이월된 Initiative 비율 | 20% 이하 | Cycle 종료 |
| Spec Gate 리드타임 | 초안 → Ready 소요 시간 | 5일 이내 | Phase별 |
| Blocked 해소 시간 | 막힘 발생 → 해소 소요 | 3일 이내 | 주간 |
| Review SLA | Review Needed 평균 대기 | 1일 이내 | 주간 |

---

## 9. 관련 문서

### 9.1 상위 문서

| 유형 | docId | 제목 |
|------|-------|------|
| 제안서 | PROP-planops-001 | 운영/진척 관리 프로세스 도입 제안 |

### 9.2 하위 문서

| 유형 | docId | 제목 |
|------|-------|------|
| DB 스펙 | DBS-notion-cycles-001 | Notion DB 스펙 - Cycles |
| DB 스펙 | DBS-notion-phases-001 | Notion DB 스펙 - Phases |
| DB 스펙 | DBS-notion-opstasks-001 | Notion DB 스펙 - Ops Tasks |
| DB 스펙 | DBS-notion-initiatives-001 | Notion DB 스펙 - Initiatives |

### 9.3 참조 문서

- `.agent/rules/document-schema.md` - docId 네이밍 규칙
- `.agent/rules/change-control.md` - 변경통제 규칙
- `.agent/rules/ownership.md` - Owner 규칙

---

## 10. 용어 정의

| 용어 | 정의 |
|------|------|
| Cycle | 분기/월/스프린트 등 기간 단위 컨테이너 |
| Initiative | 하나의 큰 계획(에픽) 단위 (선택적 사용) |
| Phase | 실제 일정이 있는 작업 단계 (Spec/Dev/QA/Release) |
| Gate | 다음 단계 진입 조건 (품질 관문) |
| Active Cycle | 현재 운영 중인 기간 (Status = Active) |
| docId | 문서 고유 식별자 |
| SSOT | Single Source of Truth (유일한 진실의 원천) |
| DoD | Definition of Done (완료 정의) |

---

## 11. 변경 이력

| 버전 | 날짜 | 변경 내용 | CHG | 작성자 |
|------|------|----------|-----|--------|
| 1.0 | 2026-01-06 | 최초 작성 | - | 박재형 |
