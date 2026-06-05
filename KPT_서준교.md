# UnitConverter_18 — KPT 회고

> 작성일: 2026-06-05  
> 범위: spec → red → green → refactoring 전 과정  
> 워크플로: ECB · Dual-Track TDD

---

## K — Keep (유지할 것)

### 1. 문제를 먼저 고정하고 구현으로 내려간 흐름

- 저는 이번 프로젝트에서 README와 프로토타입 `UnitConverter.py`를 바로 수정하기보다, 먼저 실제 불편함과 사용 행동을 정리하려고 하였습니다. 이 접근은 구현 방향을 잡는 데 효과적이었습니다.
- 저는 *「환산 비율 계산·코드 분기 증식으로 오류와 수정 비용이 커진다」* 라는 한 문장 문제 정의를 먼저 잡았습니다. 그 결과 PRD(FR-CVT/REG/IN), Test Plan, ECB 분해가 한 방향으로 이어질 수 있었습니다.
- Report 01에서 정리한 Mom Test 증거 3건(한 줄 입력, 손 계산 번거로움, cubit 추가 시 if/elif 증식)이 SC-1~SC-3과 OCP 목표의 기준이 되었습니다. 덕분에 구현 중에도 “무엇을 증명해야 하는가”를 잃지 않을 수 있었습니다.

### 2. 문서와 계약을 함께 남긴 점

- 저는 `Report/*.md`에 제가 무엇을 했고, 현재 상태와 리스크가 무엇인지 남기려고 하였습니다.
- `docs/PRD.md`와 `docs/INTERFACES.md`에는 요구사항, 시그니처, 불변식을 구현 계약으로 고정하였습니다.
- `TODO_RED.md`와 `reference.md`를 통해 Test ID와 pytest 경로를 추적하였습니다.
- 문서가 많아진 부담은 있었지만, spec→RED→GREEN→REFACTOR 흐름을 되짚을 때 큰 도움이 되었습니다. 특히 PR 리뷰를 반영할 때 “어떤 계약을 바꾸는지” 확인할 수 있었다는 점이 좋았다고 생각합니다.

### 3. AI와 협업하기 위한 규칙을 먼저 세운 점

- 저는 `.cursorrules`(헌법) → `unit-converter-tdd` Skill(절차) → `/tdd-red` · `/review-ecb` Command(실행 단축) 순으로 AI 협업 규칙을 쌓았습니다. 이 점은 잘한 부분이라고 생각합니다.
- 저장소 루트가 프로덕션인지, UI/Logic Dual-Track을 어떻게 나눌지, RED Harness와 spec RED를 어떻게 구분할지 미리 정리해 둔 덕분에 이후 혼선을 줄일 수 있었습니다.
- PR 리뷰에서 나온 비차단 제안을 Skill § GREEN 예외 규칙으로 다시 반영한 점도 좋았습니다. 단순히 코드만 고친 것이 아니라, 다음 진행 방식까지 함께 보정하였습니다.

### 4. Dual-Track TDD와 Test ID 추적

- 저는 Logic Track(`D-*`: entity·control)과 UI Track(`U-*`: boundary)을 나누어 PRD와 테스트를 연결하였습니다. 이 방식은 다음에도 유지할 만하다고 느꼈습니다.
- `tests/entity/`, `tests/boundary/`, `tests/control/`을 분리하고, Logic Track에서는 domain mock을 금지한 규칙이 ECB 구조를 지키는 데 도움이 되었습니다.
- `test_logic_tests_no_domain_mock`, `test_boundary_does_not_import_entity` 같은 Harness 테스트로 규칙 위반을 자동 검출한 점도 좋았습니다. 사람이 기억하는 규칙보다 테스트로 고정한 규칙이 더 믿을 만하다는 것을 느꼈습니다.

### 5. REFACTOR 전 Golden Master 확보

- 저는 리팩토링 전에 전체 `pytest tests/ -v` 결과를 23 passed로 고정하였습니다. 이는 잘한 판단이었다고 생각하였습니다.
- RED 단계의 `7 failed + 4 passed(harness)`에서 GREEN `20 passed`, 리뷰 반영 후 `23 passed`까지의 타임라인을 명확하게 남길 수 있었습니다.
- 별도의 argv Golden 파일은 없었지만, 이번 프로젝트 규모에서는 전체 pytest 통과를 Golden Master로 삼아도 충분하다고 판단하였습니다. 덕분에 REFACTOR에서 동작을 바꾸지 않았다는 확신을 갖고 구조를 정리할 수 있었습니다.

### 6. 의도에 부합하는 REFACTOR 진행

- `try_parse_and_validate`로 파싱 중복을 줄이고, `DEFAULT_UNITS` 기반으로 registry 초기화를 정리하고, output 포맷터를 분리하였습니다.
- 공개 API(`validate_raw`, `run_conversion`, `format_results`) 시그니처를 유지했고, 테스트 23건도 그대로 통과하였습니다. 이 점은 TDD의 REFACTOR 의도에 맞게 진행한 부분이라고 생각합니다.

### 7. 브랜치와 PR로 단계별 스냅샷을 남긴 점

- 저는 `spec` → `red` → `green` → `refactoring` 브랜치와 PR #1~#4로 Phase별 상태를 남겼습니다.
- 나중에 회고하거나 발표할 때 “어느 시점에 무엇이 완성되었는지”를 설명하기 쉬웠습니다.
- `staging`에 단계별로 통합한 것도 전체 진행 흐름을 보존하는 데 도움이 되었습니다.

---

## P — Problem (문제·아쉬움)

### 1. 정한 규칙을 끝까지 엄격하게 지키지는 못한 점

- 저는 Rule/Skill에서 Test ID 1묶음씩 RED→GREEN을 권장해 두었지만, 실제 진행은 그보다 느슨하게 하였습니다.
- RED는 설계표 기준으로 묶어서 작성했고, GREEN·Harness 보강 과정에서는 D-CVT-02, U-FLOW, U-OUT, Harness 8건 등이 RED 선행 없이 추가된 부분이 있었습니다.
- 결과적으로 테스트는 통과했지만, 이 방식은 GREEN 단계에서 디버깅 범위를 키우고 Phase 경계를 흐리게 만들 수 있었습니다. 제가 속도를 내려고 한 만큼 TDD 리듬은 일부 흔들렸다고 생각합니다.

### 2. 초반 기획과 거버넌스에 비해 구현이 뒤로 밀린 점

- 저는 초반에 Mom Test, PRD, ECB 설계, Harness, Rule, RED 스켈레톤에 많은 시간을 사용하였습니다.
- 이 과정이 필요하긴 했지만, Report 01~03 시점까지 실제 구현은 stub 수준에 머물렀고 핵심 GREEN 구현은 한 세션에 집중되었습니다.
- “인프라 먼저, 코드 나중” 전략은 안정적인 기반을 만들었지만, 중간에는 실제 기능이 얼마나 완성되었는지 체감하기 어려웠습니다. 다음에는 설계와 작은 구현을 더 짧은 주기로 번갈아 진행해 보고 싶습니다.

### 3. 문서 갱신이 코드보다 늦었던 점

- 저는 `INTERFACES.md`의 표 출력 규칙(입력 단위 줄 제외)을 GREEN 후 리뷰에서야 반영하였습니다.
- `reference.md`와 `TODO_RED.md`의 GREEN 체크도 구현보다 늦게 갱신하였습니다.
- `README`의 CLI 실행 경로도 처음에는 `pip install -e`와 `PYTHONPATH` 조건이 명확하지 않았습니다.
- 코드는 맞게 수정하였지만, 문서가 뒤따라오면서 리뷰 포인트가 생겼습니다. 이 부분은 제가 더 꼼꼼하게 챙겼어야 했다고 생각하였습니다.

### 4. Mom Test 검증 표본이 부족했던 점

- 저는 Pain과 페르소나를 학습자 관점으로 정리했지만, 실제 검증은 제한적으로 진행하였습니다.
- cubit 등록, 설정 파일 로드 같은 추가 요구는 README와 과제 맥락에서는 타당했지만, 실제 사용자가 얼마나 필요로 하는지는 아직 가설에 가깝습니다.
- PRD와 테스트 우선순위는 논리적으로 정리했지만, 동료나 실제 사용자에게 한 번 더 물어보는 과정은 부족하였습니다.

### 5. RED Harness 해석을 처음부터 명확히 잡지 못한 점

- 저는 초반에 `NotImplementedError` stub이 RED로 유효한지, skip은 어떤 상태로 봐야 하는지 혼동하였습니다.
- 이후 Report 03과 `.cursorrules` §5에 red Harness skeleton 허용 범위를 명시하면서 해석은 정리되었습니다. 다만 처음부터 더 명확하게 정의했으면 불필요한 확인을 줄일 수 있었을 것입니다.

### 6. 환경과 운영 이슈를 뒤늦게 발견한 점

- Windows 환경에서 `pip install -e` 없이 `python UnitConverter.py`를 실행하면 import가 실패하는 문제를 뒤늦게 확인하였습니다.
- Git push 중 프록시 오류(`Proxy CONNECT aborted`)도 발생하였습니다.
- pre-commit pytest나 Golden 갱신 정책 같은 Hook은 아직 구현하지 못하였습니다. 기능 구현에는 직접 문제가 없었지만, 운영 자동화까지 마무리하지 못한 점은 아쉽습니다.

###  7. Prompting 기록이 부족한 점

- 저는 Report와 git log는 남겼지만, AI에게 어떤 질문을 했고 어떤 판단을 거쳐 절충했는지는 별도 `Prompting/*.md`로 정리하지 못하였습니다.
- 나중에 AI 협업 방식을 재현하려면 결과물뿐 아니라 질문과 판단 과정도 남겨야 한다고 느꼈습니다. 이 부분은 다음 프로젝트에서 보완하고 싶습니다.

---

## T — Try (다음에 시도할 것)

### 1. Test ID 단위 RED→GREEN 리듬을 더 엄격히 지키기

- Harness 일괄 보강 같은 예외는 허용하되, 기능 Test ID는 RED를 먼저 두는 원칙을 더 엄격히 지키겠습니다.
- 다음에는 한 세션에 여러 ID를 한꺼번에 GREEN으로 밀기보다, Phase 또는 Test ID마다 pytest 요약과 짧은 회고를 남기겠습니다.

### 2. Mom Test 2라운드를 실제로 수행하기

- 학습자 페르소나에 가까운 동료 3명에게 `meter:2.5` 입력, cubit 등록 필요성, 출력 포맷 필요성을 짧게 물어보겠습니다.
- AI가 보완해 준 가설과 실제 사용자의 답변이 어디서 다른지 비교하고, 그 차이를 PRD와 P2 backlog에 반영하겠습니다.

### 3. 문서를 코드와 같은 단위로 갱신하기

- GREEN/REFACTOR마다 `INTERFACES.md`, `reference.md`, `TODO_RED.md`, `README.md`를 같은 PR 또는 같은 커밋 묶음에서 함께 확인하겠습니다.
- Phase가 끝날 때마다 pytest 수, 브랜치, PR 번호를 Report에 남겨서 문서와 코드의 시간 차이를 줄이겠습니다.

### 4. Golden Master 정책을 명시하기

- 이번 프로젝트에서는 `pytest tests/ -v`의 N passed를 Golden Master로 삼았습니다.
- 다음에는 이 기준을 Rule 또는 `docs/HARNESS.md`에 짧게 명시하고, REFACTOR 전후 동일 passed 수와 Harness 통과 여부를 체크리스트로 남기겠습니다.
- 필요하다면 pre-push에서 `pytest tests/ -q`를 실행하는 Hook도 사용자 승인 후 도입하겠습니다.

### 5. AI 협업 과정도 기록하기

- Ask로 방향과 계약을 확인하고 Agent로 구현하는 패턴은 계속 사용하겠습니다.
- 다만 결과 Report만 남기지 않고, 중요한 질문·판단·PR 리뷰 대응은 `Prompting/01~NN.md` 형태로 따로 기록하겠습니다.
- 이렇게 하면 다음 프로젝트에서 AI 협업 방식을 더 쉽게 재사용할 수 있을 것입니다.

### 6. P2 backlog를 RED부터 분리하기

- `D-REG-02`, `D-CFG-01`, 추가 output 포맷은 별도 Phase로 분리해 RED부터 시작하겠습니다.
- 특히 `config_loader`는 entity/registry와 boundary 경계를 흐릴 수 있으므로, Report 02의 아키텍처 원칙을 먼저 다시 확인하고 들어가겠습니다.

### 7. 네이밍과 온보딩 정보를 정리하기

- README 상단에 `UnitConverter_18`(repo), `UnitConverter_XX`(과제·Report 코드명), `unit-converter-18`(패키지)의 차이를 짧게 고정하겠습니다.
- 신규 참여자가 처음 실행할 수 있도록 `pip install -e ".[dev]"` → harness 테스트 → RED 1건 실행 흐름도 간단히 남기겠습니다.

---

## 프로젝트 요약

| 단계 | Report / PR | pytest | 핵심 성과 |
|------|-------------|--------|-----------|
| Mom Test·PRD | 01 · PR #1 spec | 테스트 없음 | Pain → 도메인 고정 · FR·SC |
| ECB·Harness | 02 · PR #1 | — | Dual-Track · INTERFACES · Rule |
| 전체 RED | 03 · PR #2 red | 7 failed, 4 passed | 스켈레톤 · D-CVT/U-IN RED |
| GREEN | — · PR #3 green | 20 → 23 passed | converter·input·flow·ECB harness 8건 |
| REFACTOR | 본 문서(04) · PR #4 | 23 passed (불변) | 파싱 DRY · registry SSOT · output 분리 |

---

## 종합 한 줄

**Keep:** 저는 문제 정의를 먼저 고정하고, Dual-Track TDD + Rule/Skill + ECB Harness로 AI와 함께 구현과 회귀를 통제한 점을 유지하고 싶습니다.  
**Problem:** 반면 Test ID 단위 진행을 끝까지 엄격히 지키지 못했고, 문서 갱신·Mom Test 검증·네이밍 정리를 뒤늦게 처리한 점은 아쉽습니다.  
**Try:** 다음에는 Phase 단위 GREEN, 문서 동기화, Prompting 기록, P2 RED 분리, 동료 재검증을 통해 가설을 코드 밖에서도 더 검증해 보겠습니다.
