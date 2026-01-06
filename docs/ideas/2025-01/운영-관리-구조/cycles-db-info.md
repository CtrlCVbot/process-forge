---

## docId: DBS-notion-cycles-001
type: DBS
title: Notion DB 템플릿 - Cycles (Active Cycle 기반)
status: Draft
rev: 0.1
owner: (작성자)
reviewers: [(버디/리드)]
effectiveDate: 2026-01-02
links:
notion_db: (Cycles DB URL)
related_docs:
- PRC-planops-001
- POL-docops-001

# 1. 개요

Cycles DB는 “이번에 우리가 운영/집중하는 기간”을 정의하는 **기간 컨테이너**다.

분기(Quarter)든 월(Month)든 스프린트(Sprint)든, 운영 화면/필터는 **Cycle 이름이 아니라 `Status=Active`**를 기준으로 작동한다.

- **핵심 목표:** “마스터 플랜/스펙/개발/주간 실행” 뷰들이 기간 단위가 바뀌어도 깨지지 않게 한다.
- **운영 원칙:** Active Cycle은 원칙적으로 **항상 1개**만 유지한다.

---

# 2. 데이터 모델에서의 위치

## 2.1 관계(권장)

- Cycles (1) ── (N) Initiatives
- Initiatives (1) ── (N) Phases
- Phases (1) ── (N) Ops Tasks

> Note: Initiatives/Phases/Ops Tasks는 모두 Rollup으로 Cycle Status를 가져와 표준 뷰 필터를 Cycle Status = Active로 고정한다.
> 

---

# 3. 레코드 규칙(네이밍/ID)

## 3.1 이름 규칙(Name)

- 예시(분기): `2026 Q1`
- 예시(월): `2026-02`
- 예시(스프린트): `2026 S01 (02/03~02/16)`

## 3.2 cycleId(선택)

- 팀에서 ID 기반 운영을 강하게 하고 싶다면:
    - 예: `CYC-2026Q1`, `CYC-202602`
- 초기에는 Name만으로도 운영 가능. (다만 외부 연동/Jira 연결까지 가면 ID를 추천)

---

# 4. 속성 스키마(필수/권장)

아래는 “최소로 시작 → 확장”이 가능한 안정 스키마.

## 4.1 필수 속성

| 속성명 | 타입 | 필수 | 예시 | 설명 |
| --- | --- | --- | --- | --- |
| Name | Title | ✅ | 2026 Q1 | 기간 이름 |
| Start | Date | ✅ | 2026-01-01 | 기간 시작 |
| End | Date | ✅ | 2026-03-31 | 기간 종료 |
| Status | Select | ✅ | Active | Planned / Active / Done |
| Owner | Person | ✅ | 리드 | 기간 운영 책임 |

### Status 값(고정)

- `Planned` : 계획 수립 중(아직 운영 기준 아님)
- `Active` : **운영 기준(표준 뷰가 바라보는 단 하나의 기간)**
- `Done` : 종료/아카이브

## 4.2 권장 속성(운영 편의)

| 속성명 | 타입 | 필수 | 예시 | 설명 |
| --- | --- | --- | --- | --- |
| Cadence | Select |  | Quarter | Quarter / Month / Sprint |
| Goal | Text |  | “정산 리팩토링 릴리즈” | 기간의 최종 성과(1~2문장) |
| Notes | Text |  |  | 특이사항/운영 메모 |
| Active Guard | Formula(또는 Checkbox) |  | true | “Active 1개만” 유지 점검용(옵션) |

> Formula는 Notion 버전에 따라 문법 차이가 있을 수 있어 옵션 처리로 둔다.
> 
> 
> 운영 룰(사람 프로세스)로도 충분히 강제 가능.
> 

---

# 5. 롤업/지표(선택 확장)

Cycles DB에서 “이번 기간이 얼마나 진행 중인지”를 보고 싶으면 아래를 추가한다.

## 5.1 Initiatives 연결(선택)

- `Initiatives` (Relation → Initiatives)

## 5.2 지표 롤업 예시

| 속성명 | 타입 | 설명 |
| --- | --- | --- |
| #Initiatives | Rollup | Initiatives count |
| #Committed | Rollup | Initiatives 중 Status=Committed count(구현 방식은 DB 설계에 따라) |
| #Released | Rollup | Initiatives 중 Status=Released count |
| Progress(%) | Formula | Released/Committed 비율(옵션) |

> 초기에는 지표 없이 시작하고, 2~3회전 운영 후에 “정말 필요한 지표만” 추가하는 것을 권장.
> 

---

# 6. 표준 뷰(권장 세트)

Cycles DB는 뷰가 많을 필요는 없고, “Active 전환/종료”가 빠르게 되면 된다.

## 6.1 Views

1. **Active Cycle (Table)**
- Filter: `Status = Active`
- Sort: Start desc
- 목적: 지금 운영 기준이 무엇인지 “단 하나”를 보여줌
1. **Planning Queue (Table)**
- Filter: `Status = Planned`
- Sort: Start asc
- 목적: 다음 기간 준비
1. **Archive (Table)**
- Filter: `Status = Done`
- Sort: End desc
- 목적: 종료된 기간 모아보기

---

# 7. 페이지 템플릿(레코드 본문 템플릿)

Cycles 레코드(페이지) 안에는 “운영 체크리스트 + 링크 허브”만 둔다.

(문서 본문 SSOT는 Obsidian이므로 Notion에 상세 문서를 복제하지 않는다)

## 7.1 Cycle Page Template: “Cycle Hub”

### 섹션(권장)

- **Goal / Outcome**
- **Key Dates**
- **Links**
    - Master Plan(Phases Timeline)
    - Spec Pipeline
    - Dev Timeline
    - Weekly Execution(Ops Tasks)
    - Initiative Portfolio
- **Kickoff Checklist**
- **Re-plan Log(링크만)**
- **Close Checklist**

### Kickoff Checklist(예시)

- [x]  Status=Active는 이 Cycle만 존재 → Nope, sprint 단위가 존재할 수 있으니까.
- [ ]  Initiatives(Committed/Stretch) 구분 완료
- [ ]  각 Initiative에 Spec/Dev Phase 생성
- [ ]  Phase Start/End 입력 완료
- [ ]  Master Plan 타임라인에서 전체 일정 노출 확인

### Close Checklist(예시)

- [ ]  Released / Carry-over / Dropped 정리
- [ ]  다음 Cycle Planned 등록
- [ ]  회고 요약(3줄) 링크(SSOT) 연결
- [ ]  Status=Done 전환

---

# 8. 운영 프로세스(이 DB가 관여하는 핵심 이벤트)

## 8.1 Cycle Kickoff(기간 스위치)

**Trigger:** 새 기간 시작(분기/월/스프린트)

**Action:**

1. 이번 기간 Cycle의 `Status`를 `Active`로 변경
2. 기존 Active Cycle은 `Done`으로 변경
3. (자동 효과) Initiatives/Phases/Ops Tasks의 표준 뷰가 전부 새 Active로 전환

**Rule:** Active는 항상 1개

## 8.2 Cycle Close(종료)

**Trigger:** 기간 종료 또는 운영 종료 결정

**Action:**

1. 종료 정리(Released/Carry-over/Dropped)
2. Cycle `Status=Done`
3. 다음 Cycle을 Planned로 유지(또는 바로 Active로 전환)

---

# 9. 품질/가드레일

## 9.1 “Active 2개” 방지

- 가장 강력한 방법: **운영 룰로 금지 + Active Cycle 뷰를 항상 띄워둠**
- (옵션) Active Guard 속성으로 경고 표기(필요 시)

## 9.2 날짜 무결성

- Start/End는 필수
- End < Start 같은 오류는 운영상 바로 병목을 만든다 → 생성 시 점검

---

# 10. 체크리스트(구축 순서)

1. Cycles DB 생성
2. 필수 속성(Name/Start/End/Status/Owner) 추가
3. Status 옵션값(Planned/Active/Done) 고정
4. Views 3개 생성(Active/Planning/Archive)
5. Cycle Hub 템플릿 생성(체크리스트+링크 섹션)
6. (선택) Cadence/Goal 추가

---

# 11. 다음 문서(순차 문서화)

- DBS-notion-initiatives-001 (Initiatives DB 템플릿)
- DBS-notion-phases-001 (Phases DB 템플릿)
- DBS-notion-opstasks-001 (Ops Tasks DB 템플릿)
- (선택) DBS-notion-docsregistry-001 (Docs Registry DB 템플릿)

---