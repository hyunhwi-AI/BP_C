# 팀 KPT 회고 — UnitConverter Dual-Track TDD 실습

**작성일:** 2026-06-05  
**범위:** UnitConverter 13 · 14 · 15 · 16 · 17 · 18  
**공통 프레임:** Mom Test → SPEC/Harness → RED → GREEN → REFACTOR (+ Golden Master)

---

## K — Keep (잘한 것, 계속할 것)

### 1. TDD + ECB를 "말"이 아니라 "게이트"로 운영

대부분 팀이 RED(tests만) / GREEN(src만) / REFACTOR(동작 불변)를 Rule·Skill·Command에 명시했습니다.

- _13: 34 passed + Golden 7
- _15: 12 passed + Golden
- _17: 25 passed + GM 7 (REFACTOR 후에도 회귀 유지)

### 2. Mom Test → 요구/테스트로 이어지는 설계

- _13: 해외 주문·재검증 비용
- _14: C2C·CONV-04 meter 경유
- _16: 입력 오타·조용한 실패

"진짜 문제"가 Test ID·검증 규칙에 연결된 사례가 많습니다.

### 3. Cursor Harness 삼각 구조

`.cursorrules`(헌법) → Skill(전체 사이클) → Command(RED/GREEN/REFACTOR 단축)

_13, _14, _15, _16, _17에서 반복되며 Agent scope creep를 줄이는 데 효과가 있었습니다.

### 4. Report + Prompting Transcript 쌍

- _14: 02 SPEC Report/Transcript
- _15: Report README에 Part A~M 규칙
- _16: STEP 1~5 Report/Transcript

세션 재현 경로를 남긴 팀이 많고, C2C 추적의 핵심 자산입니다.

### 5. 명세·추적 문서의 깊이 (팀별 강점)

| 팀 | 강점 |
|----|------|
| _14 | docs/spec 00~12, 추적 매트릭스, 품질 체크리스트 |
| _15 | TEST_PLAN, Dual-Track RED ID (U-IN/D-CNV) |
| _17 | TRACEABILITY, Golden Master, Command 8개 |
| _16 | README Phase 대시보드, PRD 역할 분리 (README=진행, PRD=계약) |

### 6. 상호 리뷰로 Prompting 품질을 보는 문화

spec PR에서 구현 정답보다 Transcript·Report·Phase 선언을 보는 리뷰 축이 생겼습니다. PR description·pytest N/A 명시 같은 좋은 사례(_14)가 팀 학습으로 이어졌습니다.

### 7. REFACTOR를 "스멜 → 안전 변경"으로 쪼갠 팀

- _14: refactor-smell / refactor-safe
- _15: Report 12 → 13
- _16: Change Budget 2턴
- _17: RS-01~03

assert/Golden 불변을 전제로 구조만 개선한 흐름이 모범입니다.

---

## P — Problem (문제점, 개선 필요)

### 1. 팀 간 "같은 개념, 다른 이름·경로"

| 축 | 예시 |
|----|------|
| Test ID | D-CNV(_15,_16) vs CONV(_14) vs D-CVT(_18) vs FR-01(_13) |
| 코드 위치 | 루트 ECB(_13) vs src/unit_converter/(_14,_17) vs src/entity/(_16,_18) |
| 문서 | docs/PRD.md vs docs/spec/ vs TEST_PLAN.md |

조원끼리 리뷰·회고 시 번역 비용이 큽니다.

### 2. 진행 SSOT가 README에 없거나 약한 경우

_13, _15는 Report 중심이고 README 상단이 시드 템플릿에 가깝습니다. _14, _16은 Current Phase 배너가 있어 "지금 어디인지"가 바로 보입니다. 팀 전체 기준이 갈립니다.

### 3. PR 본문·세션 재현 경로 공백

_13, _15 spec PR은 description이 비어 있거나 약해 Report 없이는 맥락 복원이 어렵습니다. 반면 _14는 포함/제외·다음 RED 순서까지 PR에 있어 대비됩니다.

### 4. spec 단계에서 문서·수치 불일치

- _15: OQ-1 자릿수(4 vs 5), 프로젝트명(01 vs 15)
- _14: factor 근사 vs 정밀, TODO 백로그 orphan

첫 GREEN 전에 수정하지 않으면 assert·Golden에서 시간을 잃습니다.

### 5. Hook·자동 게이트는 공통으로 미착수

Rule에 "RED 중 src 수정 금지"는 있으나 Hook/CCTV는 _13~_18 모두 미구현 또는 전략만 있습니다. Phase 혼선은 여전히 사람·리뷰에 의존합니다.

### 6. Golden Master·E2E 타이밍 불균형

_13, _15, _17은 Golden까지 갔지만 _16은 Entity Logic만 REFACTOR 완료(6/6)이고 Control/Boundary·CLI·Golden은 미착수입니다. 레이어·트랙 진행 속도가 팀 내에서 크게 다릅니다.

### 7. 6시간 Activities와 spec 깊이의 긴장

명세·Harness·Command까지 쌓는 데 시간을 쓰면 구현·TC 본문 비중이 줄어듭니다. _18은 RED 설계·스켈레톤 중심, _13/_17은 end-to-end에 가깝게 진행 — 목표 달성 정의가 팀마다 다릅니다.

---

## T — Try (다음에 시도할 것)

### 1. 팀 공통 PR 템플릿 (spec/red/green/refactor 공용)

매 PR 본문에 최소 4줄 고정:

- Phase / Layer / Track
- 포함·제외 범위
- pytest 스냅샷 (또는 N/A)
- 다음 1 Test ID + Command

### 2. README 상단 "Current Phase" 배너 의무화

_14/_16 형식을 팀 규약으로: 브랜치, 허용/금지, 다음 Test ID, 마지막 pytest 결과 한 줄.

### 3. Test ID·경로 "로스터 1페이지" 공유

수업/팀 위키에 팀별 ID 매핑만 적기 (예: D-CNV-01 ≈ CONV-01 ≈ Logic entity RED). 코드 통일이 아니라 리뷰·회고용 번역표.

### 4. spec merge 전 체크 3항 (각자 로컬)

- 자릿수·Golden 값 한 줄 통일
- Rule/Skill 안 프로젝트명 = repo명
- Report 02 + Prompting 02 쌍 + README 링크 일치

### 5. Transcript 품질 규칙 (red부터)

매 턴: 사용자 프롬프트 원문 1블록 + 실행한 pytest 명령 + passed/failed/skipped. spec은 요지 요약만이어도 red부터는 원문+결과 고정.

### 6. 리뷰 코멘트 출력 규칙

`/review-pr-spec` 결과는 코드 블록 전체로 붙이기, 본문에서 `~` 금지 (8-9로 표기). 팀 리뷰어 공통 룰로 공유.

### 7. Golden Master 착수 기준 합의

"Boundary 1건 GREEN + CLI 스모크" 이후 Golden 1쌍 추가 — _13/_17 사례를 팀 기준으로 문서화.

### 8. 본인(_16) 다음 스프린트

- Control RED: E001~E003, `/tdd-red`
- Entity `/review-ecb`로 Mom Test SC 회귀 확인
- Boundary(U-*) 착수 전 README Phase 갱신
- Golden은 ECB wiring 후 `tests/golden/`

### 9. 회고 발표용 개인 1장 (각 조원)

Keep 1 / Problem 1 / Try 1 + pytest 숫자 한 줄 — KPT 전체를 이 1장으로 모아 발표하면 1시간 Activities에 맞습니다.

---

## 팀 진행 스냅샷 (회고 참고용)

| 프로젝트 | 대략 위치 | pytest 규모 (Report 기준) | 특징 |
|----------|-----------|---------------------------|------|
| _13 | REFACTOR 완료 | 34 passed (+Golden 7) | 루트 ECB, manifest 17+10, Golden |
| _14 | Entity REFACTOR 중 | 3 passed (CONV) | docs/spec·C2C, Command 6개 |
| _15 | REFACTOR 완료 | 12 passed (+Golden) | TEST_PLAN, GUI 스모크까지 |
| _16 | Entity REFACTOR 완료 | 6/6 entity | STEP 1~5, Control/Boundary 대기 |
| _17 | REFACTOR 완료 | 25 passed (+GM 7) | 패키지 구조, Command 8개 |
| _18 | RED→GREEN 진행 | harness+다수 ID | INTERFACES, ECB src/ |

---

## 발표용 한 줄 요약

**Keep:** AI Harness + Dual-Track TDD로 "실패 테스트 → 최소 구현 → 불변 리팩터"를 팀 전체가 같은 언어로 시도했다.

**Problem:** 이름·경로·문서 SSOT가 팀마다 달라 협업·리뷰 비용이 크고, README/PR 재현성·자동 게이트는 아직 약하다.

**Try:** PR/README 최소 템플릿 + red부터 Transcript에 pytest 스냅샷 + Golden 착수 기준을 팀 규약으로 맞추고, 각자 다음 레이어(본인은 Control RED)로 넓힌다.
