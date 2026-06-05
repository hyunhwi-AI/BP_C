## K — Keep (유지할 것)

### 1. Mom Test → PRD → 구현의 한 줄기
- 기능 명세(FR/OCP/`units.json`)보다 **실제 Job·비용·행동**을 먼저 고정한 흐름이 효과적이었습니다.
- **한 줄 가설** — *“변환기가 아니라 보내기 전 단위 검증”* — 이 PRD 우선순위(FR-06~09), Test Plan, Phase 로드맵까지 일관되게 이어졌습니다.
- Prompting 01에서 AI가 **솔루션 bias를 지적**하고, 가구 오배송(P0)을 끌어올린 것이 이후 모든 결정의 기준이 되었습니다.

### 2. 문서·세션 이중 기록 (Report + Prompting)
- `Report/*.md`: **무엇을 했는지·상태·리스크·다음 단계** (운영/발표용)
- `Prompting/*.md`: **어떤 질문·판단·절충**이 있었는지 (프롬프트·AI 협업 재현용)
- 5개 리포트가 누적 진행률·pytest 타임라인을 남겨, RED→GREEN→REFACTOR 여정을 추적하기 쉬웠습니다.

### 3. AI 거버넌스 스택 (Rule → Skill → Command → Test Loop)
- `.cursorrules`(헌법) → `dual-track-ecb-tdd` Skill(절차) → `/tdd-red` Command(RED 단축) 순으로 쌓은 구조는 Report 02의 핵심 성과입니다.
- **루트만 프로덕션**, **UI/Logic Dual-Track**, **TDD Phase vs PRD Phase** 용어 정합도 이후 혼선을 줄였습니다.

### 4. Dual-Track TDD + manifest 단일 출처
- UI Track 17건(`manifest.json` Given/Then) + Logic Track 10건(D-*)로 PRD §8과 1:1 대응.
- RED에서 `pytest.raises(Exception)` → 구체 예외, NFR-02·D-E01/E02 **오탐 PASS 방지**는 품질을 높였습니다.

### 5. GREEN 직후 Golden Master
- REFACTOR 전 **CLI stdout + `*.argv.json` 쌍**으로 고정한 것은 Report 05에서 **34 passed·Golden 7건 stdout 불변**으로 검증되었습니다.
- manifest `given`(문서용) vs golden `argv`(실행용) 역할 분리도 좋은 패턴입니다.

### 6. REFACTOR에서 구조만 정리, 동작은 불변
- `composition.py` wiring, `resolve_target_units()` 단일화, `registry.canonical_unit()` — **테스트·Golden 변경 없이** NFR-01·02를 충족한 방식은 TDD REFACTOR의 모범 사례에 가깝습니다.

---

## P — Problem (문제·아쉬움)

### 1. Rule 권장과 실제 진행 방식의 괴리
- Rule/Skill: **Test ID 단위 RED→GREEN** 권장.
- 실제: **일괄 RED(27 fail)** → **한 세션에 Phase 1~4 일괄 GREEN**.
- 가능은 했지만, Report 03·04가 지적한 대로 **GREEN 시 디버깅 범위가 커지고**, Phase 경계가 흐려질 위험이 있었습니다.

### 2. 구현 대비 기획·거버넌스 비중이 큼
- Activities 6시간 기준, 초반은 **분석·기획·Harness·Rule**에 시간이 많이 갔고, **실제 GREEN은 Report 04**에 집중되었습니다.
- 01.REPORT 시점: 구현 ~5% → 04까지 ~90%로 점프. **“인프라 먼저, 코드 나중”** 전략의 trade-off입니다.

### 3. 문서·코드 동기화 지연
- 여러 번 반복된 미해결 항목:
  - `docs/HARNESS.md` §6 — Golden 절차 미반영(04·05)
  - `manifest.json`에 D-* formal 미등록(reference.md만)
  - Report 01에 Harness 미반영 → 02에서 보완
- **문서가 코드보다 늦게 따라오는** 패턴이 있습니다.

### 4. Mom Test 검증의 한계
- Pain 상당 부분이 **본인 1명 + AI 보완 답변(2라운드)** 기반.
- Report 01·Prompting 01: *“주변 3명 추가 검증 필요”*, *“일부 답변은 추론”*.
- PRD·테스트 우선순위는 설득력 있으나, **시장 검증은 아직 가설**입니다.

### 5. skip 해석 혼동 (Report 03)
- 스켈레톤 1차: `2 failed, 25 skipped` → 사용자가 RED인지 질문.
- **skip = 대기, fail = RED 완료**라는 TDD Harness 규칙을 처음엔 직관적으로 이해하기 어려웠습니다.

### 6. 환경·운영 이슈
- **Windows cp949**: `⚠`, `≈` → `경고:`, `=` 로 우회 (04).
- **`green` branch push** — 04·05에서 요청됐으나 **세션 중단으로 미완**.
- **Hook 미구현** — Golden 갱신 정책·pre-save pytest 자동화 없음.

### 7. 아키텍처 혼동 가능성
- `unit_converter/` ECB 골격 vs **루트 프로덕션** — 02에서 정합했지만, 신규 참여자 혼동 리스크는 Report 02·03에도 남아 있습니다.

### 8. GREEN 전 `UnitConverter.py` 방치
- Phase 1~4 GREEN까지 **37줄 인터랙티브 CLI**는 그대로 두고, argv CLI는 GREEN에서 전환.
- 프로토타입과 PRD 목표 사이 **중간 상태가 길게** 유지되었습니다.

---

## T — Try (다음에 시도할 것)

### 1. TDD 진행 방식: “일괄 RED + Phase별 GREEN” 절충 유지·명문화
- Report 03 팁을 Rule/Skill에 반영: *전체 RED 후 GREEN은 PRD Phase 1→2→3→4 순*.
- **한 세션에 Phase 1~4 GREEN**은 피하고, Phase마다 Report·pytest 요약을 남기기.

### 2. Mom Test 2라운드 실제 수행
- 가구 ritual, 보고서 Q&A, 10분 입력 — **주변 3명 1문장씩** 재검증.
- AI 보완 답변과 실제 경험 차이를 PRD·우선순위에 반영.

### 3. 문서를 코드 변경과 같은 PR 단위로 갱신
- GREEN/Golden/REFACTOR마다 **`HARNESS.md`, `manifest`(D-*), README** 체크리스트를 Report §6 “미해결”에서 **완료로 넘기기**.
- Golden 추가 시: `*.argv.json` 생성 → `--update-golden` → `HARNESS.md` 절차 기록.

### 4. Hook + git 마무리
- Hook: Golden 갱신은 **사용자 승인 + `--update-golden`만** (Rule에 이미 있음 → 구현).
- **`green` branch** 생성·커밋·push로 RED→GREEN→REFACTOR 스냅샷 보존.

### 5. AI 협업 패턴 재사용
- **Ask로 방향·조건 확인 → Agent로 구현** (04 GREEN, 05 REFACTOR).
- **Rule 리뷰는 Ask, 반영은 Agent** (02) — 비용 대비 품질이 좋았습니다.
- Mom Test는 **멘토 역할 프롬프트 + “질문 1개만”** — Prompting 01에서 pain 품질을 높였습니다.

### 6. 경쟁자(구글) 기준으로 Job A 검증
- Mom Test 결론: 제품은 **ritual 3단계 → 1단계**로 구글을 이겨야 함.
- Golden FR-06·FR-07이 그 UX를 고정 — **실사용 시 ritual 시간 단축**을 정성/정량으로 한 번 측정해 볼 가치가 있습니다.

### 7. 다음 세션 프롬프트 (Report 05 제안)
```
Hook 구현 — Golden 갱신은 사용자 승인 후 --update-golden만
```
또는
```
green branch 생성 후 REFACTOR까지 커밋·push
```

---

## 한눈에 보는 프로젝트 아크

| 단계 | Report | pytest | 핵심 성과 |
|------|--------|--------|-----------|
| Mom Test·PRD | 01 | 테스트 없음 | Pain → Job A~D, FR 재정렬 |
| Harness·Rule | 02 | 17 skipped | AI 거버넌스·Dual-Track 골격 |
| 전체 RED | 03 | 27 failed | Test Plan·스텁·RED 100% |
| GREEN·Golden | 04 | 34 passed | Phase 1~4 구현·Golden 7쌍 |
| REFACTOR | 05 | 34 passed (Golden 불변) | composition·ECB·SRP 정리 |

---

## 종합 한 줄

**Keep:** Mom Test로 Job을 고정하고, Dual-Track TDD + Rule/Skill + Golden으로 AI와 함께 구현·회귀를 통제한 흐름.  
**Problem:** 일괄 RED/GREEN으로 Rule과 어긋남, 문서·git·Hook 미완, Mom Test 표본 1명·AI 보완 의존.  
**Try:** Phase 단위 GREEN·문서 동기화·Hook·green branch·실사용자 3명 재검증으로 **“보내기 전 단위 검증” 가설**을 코드 너머까지 검증하기.
