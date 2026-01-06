---

## docId: DBS-notion-phases-001
type: DBS
title: Notion DB 템플릿 - Phases (마스터 플랜의 본체: Spec/Dev/QA/Release)
status: Draft
rev: 0.1
owner: (작성자)
reviewers: [(버디/리드)]
effectiveDate: 2026-01-02
links:
notion_db: (Phases DB URL)
related_docs:
- PRC-planops-001
- DBS-notion-cycles-001
- DBS-notion-initiatives-001
- POL-docops-001

# 1. 개요

Phases DB는 **마스터 플랜(한 장)**을 구성하는 “일정의 실체”다.

Initiatives가 “무엇(What)”이라면, Phases는 **언제/어떤 성격의 일(How by time)**을 정의한다.

- 같은 Timeline 위에서 **기획(Spec) 일정과 개발(Dev) 일정**을 함께 본다.
- Phase는 단순 일정 블록이 아니라 **Gate(착수 조건) + Blocked(막힘 원인) + DoD(완료 정의)**를 포함한 운영 단위다.
- 표준 뷰는 Cycle 이름이 아니라 **`사이클 상태(롤업) = 운영중`*을 기준으로 동작한다.

---

# 2. 데이터 모델에서의 위치

## 2.1 관계(권장)

- Cycles (1) ── (N) Initiatives
- Initiatives (1) ── (N) Phases
- Phases (1) ── (N) Ops Tasks

## 2.2 원칙

- Phase는 반드시 하나의 Initiative에 속한다. (고아 Phase 금지)
- Ops Tasks는 Phase에 연결되어 주간 실행을 추적한다.

---

# 3. 레코드 규칙(네이밍/ID)

## 3.1 Phase명(Title) 규칙

Phase는 “의미 단위”로 작성한다. Task처럼 쪼개지 않는다.

- 권장 형식: `[유형] 이니셔티브명 - 목적 요약`
    - 예) `[Spec] 정산 예외 자동처리 - 정책/상태/데이터 고정`
    - 예) `[Dev] 정산 예외 자동처리 - API/정산로직 구현`
    - 예) `[QA] 정산 예외 자동처리 - 회귀/권한/예외 TC`
    - 예) `[Release] 정산 예외 자동처리 - 점진배포/모니터링`

## 3.2 phaseId(선택)

- 예: `PHS-2026Q1-0012`
- 외부 연동/리포팅을 고려하면 권장

---

# 4. 속성 스키마(한글화 표준 포함)

아래는 “필수(최소 운영) → 확장(운영 안정화)” 순서로 구성한다.

## 4.1 필수 속성(최소 운영 세트)

| 한글 속성명 | 영문 권장키(선택) | 타입 | 필수 | 예시 | 설명 |
| --- | --- | --- | --- | --- | --- |
| Phase명 | Name | Title | ✅ | [Dev] … | Phase 제목 |
| 이니셔티브 | Initiative | Relation → Initiatives | ✅ | INI-… | 소속 이니셔티브 |
| Phase 유형 | Phase Type | Select | ✅ | Dev | Spec/Dev/QA/Release |
| 시작일 | Start | Date | ✅ | 2026-01-15 | 일정 시작 |
| 종료일 | End | Date | ✅ | 2026-02-05 | 일정 종료 |
| Phase 상태 | Status | Select | ✅ | 진행중 | Not Started/In Progress/Blocked/Done |
| 책임자 | Owner | Person | ✅ | 김개발 | 해당 Phase 오너 |
| Gate 상태 | Gate | Select | ✅ | Not Ready | Ready/Not Ready |
| 산출물 docId 목록 | DocIds | Text | ✅ | PRD… POL… | SSOT 연결 키 |
| 사이클 | Cycle | Rollup | ✅ | 2026 Q1 | Initiative→Cycle |
| 사이클 상태(롤업) | Cycle Status | Rollup | ✅ | 운영중 | 표준 뷰 필터용 |

### Phase 유형(Select 옵션) - 한글 표준

- `Spec(기획)`
- `Dev(개발)`
- `QA(검증)`
- `Release(배포)`

### Phase 상태(Select 옵션) - 한글 표준

- `미시작` (Not Started)
- `진행중` (In Progress)
- `막힘` (Blocked)
- `완료` (Done)

### Gate 상태(Select 옵션) - 최소 표준

- `Not Ready` (준비안됨)
- `Ready` (준비완료)

> Tip: Gate는 특히 Spec→Dev 구간에서 강제력이 크다.
> 
> 
> Dev Phase는 Spec Gate가 Ready가 되기 전까지 “진행중”으로 올리지 않는 것을 원칙으로 한다.
> 

---

## 4.2 운영 안정화 속성(권장 확장 세트)

아래 6종은 Phase 운영에서 가장 자주 터지는 구멍을 막는 “운영 필수 확장”이다.

| 한글 속성명 | 타입 | 예시 | 목적 |
| --- | --- | --- | --- |
| 예상 공수(인일) 또는 사이즈 | Number 또는 Select(S/M/L) | 10, M | 리소스 충돌 조기 감지 |
| Gate 근거 링크(SSOT) | URL | (빌드 URL) | Ready 판단 근거 남김 |
| 막힘 사유 유형 | Select | 권한/외부팀/결정/데이터 | Blocked를 ‘해결 가능’하게 |
| 막힘 해소 책임자 | Person | 리드 | 누가 풀지 명확화 |
| 막힘 해소 목표일 | Date | 2026-01-20 | 언제까지 풀지 |
| 의존 Phase | Relation → Phases | PHS-… | Cross-phase 의존성 추적 |
| 일정 변경 사유(ADR/CHG) | Text 또는 URL | ADR-… | 왜 바뀌었는지 기록 |
| 리플랜 변경 여부 | Checkbox | ✅ | 리플랜에서 바뀐 항목 표시 |

### 막힘 사유 유형(Select 옵션) - 권장

- `권한/계정`
- `외부팀 의존`
- `정책 결정 필요`
- `데이터/정합성`
- `리소스 부족`
- `버그/장애`
- `기타`

---

# 5. Rollup/Relation 설정(Active Cycle 기반 필터 유지)

## 5.1 사이클/사이클 상태(롤업) 표준

- `사이클`: Rollup
    - 경로: `이니셔티브` → `사이클`
- `사이클 상태(롤업)`: Rollup
    - 경로: `이니셔티브` → `사이클` → `사이클 상태`

> 이 두 속성 덕분에 Phase 뷰는 “Cycle 이름”이 바뀌어도 운영중만 따라가며 깨지지 않는다.
> 

---

# 6. 표준 뷰(마스터 플랜/기획/개발/릴리즈)

모든 표준 뷰는 공통 필터를 사용한다.

- 공통 필터: `사이클 상태(롤업) = 운영중`

## 6.1 마스터 플랜(한 장)

- View: Timeline
- DB: Phases
- Filter: `사이클 상태(롤업) = 운영중`
- Group: `Phase 유형`
- Sort/표시(권장):
    - 카드 표시: 이니셔티브, 책임자, Phase 상태, Gate 상태
- 목적: **Spec/Dev/QA/Release를 한 화면에서 동기화**

## 6.2 Spec 파이프라인(기획 관점)

- View: Board by `Phase 상태` + (보조) Timeline
- Filter:
    - `사이클 상태(롤업) = 운영중`
    - `Phase 유형 = Spec(기획)`

## 6.3 Dev 타임라인(개발 관점)

- View: Timeline
- Filter:
    - `사이클 상태(롤업) = 운영중`
    - `Phase 유형 = Dev(개발)`

## 6.4 QA/Release 캘린더(선택)

- View: Calendar 또는 Timeline
- Filter:
    - `Phase 유형 in (QA, Release)`
    - `사이클 상태(롤업) = 운영중`
- 목적: 테스트/배포 몰림 방지

## 6.5 막힘(Blocked) 레지스터

- View: Table
- Filter:
    - `사이클 상태(롤업) = 운영중`
    - `Phase 상태 = 막힘`
- 표시 컬럼(권장):
    - 막힘 사유 유형, 해소 책임자, 해소 목표일, 의존 Phase

---

# 7. Gate 운영 규칙(핵심)

## 7.1 Spec → Dev (Ready for Dev) 최소 조건(권장)

Spec Phase의 Gate를 Ready로 만들기 위한 최소 조건을 “템플릿”으로 고정한다.

- [ ]  PRD docId 존재(SSOT 링크)
- [ ]  상태/권한/예외 고려 완료
- [ ]  데이터 영향(필드/검증) 명시
- [ ]  QA 핵심 TC(QAP) 링크 존재

운영 룰:

- Spec Gate가 Ready가 되기 전까지 Dev Phase를 `진행중`으로 올리지 않는다.
- Dev 진행 중 스펙 변경은 CHG로만 반영한다. (POL-docops-001 준수)

## 7.2 QA/Release Gate(선택)

- QA Done 기준: 핵심 TC 통과 + 치명 결함 없음
- Release Done 기준: 배포 완료 + 모니터링 OK + 롤백 준비 확인

---

# 8. Phase Type별 DoD(완료 정의) 표준

Phase “완료”는 팀 공통 정의가 있어야 분쟁이 없다.

- Spec 완료 = Gate Ready(Ready for Dev) 통과
- Dev 완료 = main merge + feature flag 준비 + 로그/모니터링 포인트 포함(팀 합의 기준)
- QA 완료 = 핵심 TC 통과 + Sev1/2 없음(팀 합의 기준)
- Release 완료 = 배포/전환 완료 + 모니터링 확인 + 롤백 방법 확인

> 이 DoD는 Phase 페이지 템플릿에 박아두고, 금요일 리뷰에서 “Gate/DoD”만 확인해도 운영이 된다.
> 

---

# 9. 페이지 템플릿(레코드 본문 템플릿)

Phase 페이지는 “상세 문서”가 아니라 “운영 체크 + 링크 허브”다.

## 9.1 Phase Hub 템플릿(권장)

### 섹션

- 🎯 목적(이 Phase에서 끝내야 할 것 1~2줄)
- 🔗 SSOT 링크
    - PRD/Policy/State/Data/QAP (docId + 빌드 URL)
    - Gate 근거 링크
- ✅ Gate 체크(Ready 조건 체크리스트)
- ⚠️ Blocked 정보
    - 사유 유형 / 해소 책임자 / 목표일
- 🔁 일정 변경 기록
    - Replan Flag / ADR·CHG 링크
- 📌 연결된 Ops Tasks(임베드 뷰)
    - This Week / Review Needed

### 금지

- PRD/정책/프로세스 본문을 Notion에 복제하지 않는다(SSOT 위반)

---

# 10. 운영 가드레일(실패 방지)

## 10.1 Phase를 Task처럼 쪼개지 않는다

- Phase는 “의미 단위”
- 세부 실행은 Ops Tasks로만 관리한다.

## 10.2 일정 변경은 Re-plan 슬롯에서만

- 주간 회의: Status/Gate/Blocked만 업데이트
- Re-plan: Start/End 변경 + Replan Flag 체크 + ADR/CHG 링크 남김

## 10.3 Blocked는 원인 데이터 없으면 금지

- Phase 상태를 막힘으로 바꿀 때는 최소 3개를 채운다:
    - 막힘 사유 유형 / 해소 책임자 / 해소 목표일

---

# 11. 구축 순서(세팅 체크리스트)

1. Phases DB 생성
2. 필수 속성 추가(이니셔티브/유형/시작일/종료일/상태/책임자/Gate/docIds)
3. Rollup 추가
    - 사이클(이니셔티브→사이클)
    - 사이클 상태(이니셔티브→사이클→사이클 상태)
4. Select 옵션값 고정(유형/상태/Gate/막힘사유)
5. 표준 뷰 생성(마스터플랜/Spec/Dev/Blocked/QA-Release)
6. Phase Hub 템플릿 생성(DoD/Gate/Blocked 섹션 포함)
7. (선택) 운영 안정화 속성(공수/의존/변경사유/Replan Flag) 추가

---

# 12. 다음 문서(연결되는 DB)

- DBS-notion-opstasks-001 (Ops Tasks DB 템플릿)
- DBS-notion-docsregistry-001 (Docs Registry DB 템플릿)
- (선택) PRC-changecontrol-001 (CHG/ADR 기반 변경통제 프로세스)

---