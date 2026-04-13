---
title: "하네스 종합 분석 v1 — 수렴, 발산, 공백"
date: 2026-04-14
categories: [AI 시스템 리서치, 인사이트 종합]
tags: [하네스엔지니어링, 종합분석, ClaudeCode, 패턴분석, 에이전트시스템]
---

이 글은 각 하네스의 요약이 아니다. 9개 노트를 함께 볼 때만 드러나는 구조적 패턴을 추출한다. 단일 노트의 주장은 "[하네스명]" 귀속으로, 3자 관찰은 "[관찰자명]" 귀속으로, 여기서의 추론은 "노트 추론"으로 표기한다.

분석 대상: Ralph Wiggum, Superpowers, GSD, Ouroboros, gstack, ECC, Compound Engineering, OpenSpec, revfactory/harness — 9개.

&nbsp;

## 1. 축별 분포 (Axis distribution)

### Seed 축 12개

모든 하네스 노트가 축 1–12를 사용했으나 항목 밀도가 다르다.

**밀도 상위**: 축 4(State & context), 축 5(Prompt strategy), 축 11(Transferable primitives). 설계 핵심이 이 세 축에 집중된다.

**밀도 하위**: 축 9(Empirical claims). 9개 하네스 전체에서 통제된 벤치마크는 revfactory/harness가 유일하다(자기수행, 15 태스크, 10차원 평가). 나머지 8개는 일화·스타수·스크린샷 의존.

&nbsp;

### 후보 축 A–M 분포

| 축 | 이름 | 하네스 수 | 사용 하네스 | 1줄 관찰 |
|---|---|---|---|---|
| **A** | Iteration-boundary semantics | 4 | Ralph, GSD, Ouroboros, Compound-Engineering | 매체만 다름(파일/스폰/이벤트/학습문서). 경계 처리 자체는 보편 |
| **B** | Backpressure mechanism | 2 | Ralph(N reader:1 writer), Ouroboros(Dialectic Rhythm Guard) | 희귀. 병렬화를 적극 쓰는 두 곳만 명시적 백프레셔 규칙 갖춤 |
| **C** | Mode splitting | 9 | 전체 | **유일하게 9개 모두 해당.** 단 2-way(Ralph)에서 7-way(Superpowers)까지 폭이 큼 |
| **D** | Gate mechanism syntax | 7 | Ralph, Superpowers, GSD, Ouroboros, gstack, OpenSpec, revfactory | 게이트 형태만 다름: CAPS호통/XML태그/숫자임계값/페이즈enum/allow-list |
| **E** | Authoritative process medium | 4 | Superpowers(DOT), OpenSpec(RFC2119+Gherkin), revfactory(SKILL.md 명령형), gstack(자연어 numbered steps) | 매체 선택이 Claude vs 다른 모델 범용성에 영향 가능성 |
| **F** | Skill as unit of discipline | 5 | Superpowers, gstack, ECC, Compound-Engineering, revfactory | 빠른 수렴. Anthropic Skills 기판 위 컨벤션 레이어로 5개가 독립 구현 |
| **G** | Execution environment as constraint surface | 3 | GSD, Ouroboros(입력측), ECC(hook profile) | tentenco의 3-축 프레임(Process/Environment/Perspective)에서 "Environment" 포지션 |
| **H** | Artifact naming schema as protocol | 2 | GSD(`{PHASE}-{WAVE}-{TYPE}.md`), gstack(`.tmpl` vs `.md` 역할 인코딩) | GSD만 강한 사용 |
| **I** | Ambiguity-as-numeric-gate | 1 | Ouroboros 전용 | 정량 게이트는 현재 유일. 이식 시 측정 함수가 블랙박스임이 문제 |
| **J** | Deferred-tool loading protocol | 1 | Ouroboros 전용 | MCP late-binding 컨벤션. Claude Code 생태계 특이적 |
| **K** | Role perspective as constraint surface | 4 | gstack, ECC, Compound-Engineering, revfactory | tentenco 프레임의 "Perspective" 포지션. 빠른 수렴 |
| **L** | Instinct learning as harness layer | 2 | ECC(`/learn`→`/evolve`), Compound-Engineering(`/ce:compound`→genetic search) | 구현 매체 다름(신뢰도 수치 vs YAML 마크다운)이지만 메타루프 패턴 동일 |
| **M** | Meta-skill bootstrapping | 1 | revfactory/harness 전용 | 하네스가 하네스를 낳는 패턴. 현재 단독 |

**관찰**: C는 전체 커버, F·K는 5개 이상으로 빠른 수렴, A·L은 4·2개로 승격 임계값 충족. I·J·M은 단독으로 고유성 높은 실험.

&nbsp;

## 2. 설계 공간의 주요 축 (Key design dimensions)

하네스 저자들이 실제로 선택을 해야 했던 설계 긴장점 5개.

&nbsp;

### D1. Fresh context 확보 방식: 시간 vs 공간

"컨텍스트 오염을 피하기 위해 fresh window를 언제 어떻게 만드는가"라는 선택.

**시간 극 (Ralph)**: `while true` bash 루프로 매 이터레이션 컨텍스트를 완전 와이프한다. 이터레이션은 시간축에서 직렬로 온다.
- 얻는 것: 극단적 단순성. bash 스크립트 1개면 충분. 어떤 런타임에도 이식.
- 잃는 것: 병렬화 불가. 진행이 느리고 비용이 선형 증가.

**공간 극 (GSD, Superpowers, Compound-Engineering)**: `Task()` 스폰으로 독립 컨텍스트를 공간적으로 분기한다. 병렬 웨이브가 가능하다.
- 얻는 것: 속도, 병렬화, 오케스트레이터 컨텍스트 보존.
- 잃는 것: 런타임 의존성(Task API, subagent 비용), 오케스트레이터-서브에이전트 통신 복잡도.

**중간 위치 (Ouroboros, OpenSpec)**: Ouroboros는 MCP 서버 EventStore를 통해 stateless evolve_step을 반복하는 혼합. OpenSpec은 change 폴더 단위로 격리하되 단일 에이전트 모델.

**노트 추론**: 두 극은 실제로 다른 문제를 풀고 있다. Ralph의 "시간 직렬"은 간단한 그린필드 반복에 최적. GSD의 "공간 분기"는 복잡한 단일 프로젝트의 병렬 진행에 최적. 스타 수 기준 양쪽 모두 대규모 채택이 공존하는 것이 이를 뒷받침한다.

&nbsp;

### D2. HITL 위치: 세션 외부 vs 내부 강제 게이트 vs 내부 유연

"사람의 판단이 에이전트 루프와 어떻게 맞물리는가"라는 선택.

**외부 극 (Ralph)**: HITL은 세션 사이에서만 일어난다. 실행 중 인간 개입 없음. "Wake up to a broken codebase"가 인정된 실패 모드다.

**내부 강제 극 (Superpowers, Ouroboros)**: Superpowers는 `<HARD-GATE>`로 디자인 승인 없이 구현을 물리적으로 차단한다. Ouroboros는 3연속 코드확인 후 강제로 사람에게 질문을 돌리는 Dialectic Rhythm Guard를 갖는다.

**내부 유연 극 (GSD, gstack, OpenSpec)**: GSD는 `AskUserQuestion` 툴로 결정 지점에서 질문하되 자동 차단은 아니다. gstack은 "User Sovereignty" 원칙을 선언하지만 각 단계 호출 자체는 사람이 수동으로 한다. OpenSpec은 propose → apply → archive 각 전환을 사람이 리뷰 후 결정한다.

**노트 추론**: HITL 위치는 "얼마나 자율화할 것인가"보다 "언제 사람의 판단이 가장 비싼가"의 문제다. Superpowers의 강제 게이트는 "설계 실수를 구현 후에 잡으면 비용이 기하급수적"이라는 판단을 반영한다. Ralph의 외부 HITL은 "반복 비용이 싸다"는 판단. 용도에 따라 맞는 극이 다르다.

&nbsp;

### D3. 상태 지속 매체: 파일 하나 vs 구조화 파일 vs 이벤트 스토어

**최소 극 (Ralph)**: `IMPLEMENTATION_PLAN.md` 1개 + `AGENTS.md` 1개. 단순하나 파일이 비대해지면 컨텍스트 오염.

**구조화 파일 극 (GSD, OpenSpec, Compound-Engineering)**: GSD는 `.planning/` 아래 7+ 파일로 역할 분리. OpenSpec은 `specs/` + `changes/` 이중 레이어. Compound-Engineering은 `docs/solutions/` YAML 태깅 + `CLAUDE.md` 갱신. 공통점은 "파일명 자체가 상태 머신 노드"인 것.

**이벤트 스토어 극 (Ouroboros)**: 파일 대신 MCP 서버 EventStore. evolve_step은 stateless이고 상태는 이벤트 리플레이로 재구성. 운영 복잡도 높음.

**혼합 (ECC)**: SQLite + instinct store + session adapter 중첩.

**노트 추론**: 구조화 파일이 현재 가장 많이 채택된 중간값이다. GSD의 단순한 디렉토리 구조가 넓은 채택을 얻은 것이 이 중간값의 우위를 암시한다.

&nbsp;

### D4. 프로세스 권위 표현: 자연어 vs 구조화 형식

**자연어 극 (Ralph, gstack)**: Ralph는 Markdown 프로즈. gstack은 "Express conditionals as English."

**구조화 형식 극 (Superpowers, OpenSpec, Ouroboros)**: Superpowers는 GraphViz DOT을 프로세스의 권위적 표현으로 채택. "Claude is particularly good at following processes written in dot." OpenSpec은 RFC 2119 키워드 + Gherkin. Ouroboros는 SKILL.md frontmatter가 MCP 툴 콜 시그니처를 선언.

**노트 추론**: Superpowers의 DOT 선택은 "Claude 계열에서만 검증됨"이라는 명시적 제약을 안고 있다. 자연어는 모델 애그노스틱하지만 에이전트가 지시를 창의적으로 해석할 여지가 크다.

&nbsp;

### D5. 하네스 범위: 단일 문제 집중 vs 전방위 번들

**집중 극 (Ralph, Ouroboros, OpenSpec)**: 각자 명확한 하나의 문제를 정의하고 그것만 푼다.
- 얻는 것: 선명한 포지셔닝, 이식성 높음, 다른 하네스와 스택 가능.

**번들 극 (ECC, gstack)**: ECC는 47개 에이전트, 181개 스킬, 72개 규칙. gstack은 23+ 슬래시 커맨드로 Think→Reflect 7단계를 커버.
- 잃는 것: 복잡도, 디버깅 불투명, 토큰 bloat.

**노트 추론**: 스타 수만 보면 번들이 승리한 것처럼 보인다(ECC 140k, gstack 71k). 그러나 비판도 번들에 집중된다. 집중형이 스택의 레이어로 들어가는 패턴이 tentenco/imaginex 등 실무자들의 권고 패턴으로 수렴한다.

&nbsp;

## 3. 수렴하는 패턴 (Convergent patterns)

3개 이상의 하네스에서 독립적으로 나타난 패턴.

&nbsp;

### CP1. 파일시스템 기반 이터레이션 간 기억

**패턴**: 에이전트 컨텍스트가 리셋되더라도 파일이 기억을 이어받는다.

| 하네스 | 구현 |
|--------|------|
| Ralph | `IMPLEMENTATION_PLAN.md` + `AGENTS.md` |
| GSD | `.planning/` 디렉토리 7+ 파일. STATE.md가 "지금 어디에 있는가"만 담당 |
| OpenSpec | `specs/` + `changes/` + `archive/`. 변경 히스토리의 "왜"까지 보존 |
| Compound-Engineering | `docs/solutions/` + `CLAUDE.md`. 매 이터레이션 학습 추출 |
| revfactory | `.claude/agents/` + `.claude/skills/` + 최소 CLAUDE.md |

**수렴이 놀라운가**: 아니다. 당연한 해법이다. 놀라운 것은 매체 선택의 다양성 — 단일 파일(Ralph), 역할별 분리 디렉토리(GSD), 이중 레이어(OpenSpec), 학습 누적(Compound-Engineering) — 이 모두 파일시스템이라는 동일 기반 위에 세워졌다는 것이다.

&nbsp;

### CP2. 모드 분리 (Plan/Build/Review를 다른 에이전트 컨텍스트로)

**패턴**: "기획"과 "구현"(그리고 "리뷰")을 같은 컨텍스트에서 혼합하지 않는다.

9개 하네스 모두 이 패턴을 독립적으로 재발견했다. Ralph의 2-way부터 Superpowers의 7-phase DAG까지 구현 형태만 다를 뿐이다.

**수렴이 놀라운가**: 패턴 자체보다 **9개 하네스 모두**가 독립적으로 재발견했다는 것이 중요하다. 이것은 모드 분리가 "유행하는 아이디어"가 아니라 실제 실패를 막는 load-bearing primitive라는 강한 수렴 증거다.

&nbsp;

### CP3. 서브에이전트에 "필요한 것만" 전달

**패턴**: 상위 오케스트레이터의 히스토리를 서브에이전트에게 상속시키지 않는다. 서브에이전트는 해당 태스크를 실행하기 위한 최소 컨텍스트만 받는다.

- Superpowers: "Each receives 'exactly what they need' without inheriting session history."
- GSD: executor 서브에이전트는 `N-M-PLAN.md`만 읽고 `N-M-SUMMARY.md`만 쓴다. 오케스트레이터는 서머리만 읽는다.
- Compound-Engineering: 14개 리뷰 에이전트 각각이 역할 범위만 리뷰.
- revfactory: "소규모 데이터: Task/Message, 대형 아티팩트: 파일, Subagent 결과: return value."

**수렴이 놀라운가**: 이 패턴은 직관에 반한다 — 더 많은 히스토리가 더 나은 성능을 줄 것이라는 가정을 뒤집는다. 4개 하네스가 독립적으로 "히스토리 없이 보내는 것이 낫다"는 결론에 도달했다는 것은 컨텍스트 오염 문제가 실질적이고 재현 가능하다는 것을 시사한다.

&nbsp;

### CP4. SKILL.md 컨벤션 레이어

**패턴**: Anthropic Agent Skills 기판 위에 자체 SKILL.md 포맷 컨벤션을 얹는다. YAML frontmatter + when-only description + 본문이 기본 구조.

5개 하네스(Superpowers, gstack, ECC, Compound-Engineering, revfactory)가 같은 기판 위에 서로 다른 컨벤션을 얹었다. 이 컨벤션들의 공통 방향 — "when-only description", "body on-demand", "trigger precision" — 이 있다. SKILL.md는 Claude Code 생태계의 사실상 표준이 되어가고 있다.

&nbsp;

### CP5. 역할 페르소나로 LLM 시선 고정

**패턴**: LLM에게 "일반 조수"가 아니라 특정 역할(CEO, security reviewer, QA 등)을 부여해 판단의 시선을 고정한다.

- gstack: 23+ 슬래시 커맨드, 각각 하나의 엔지니어링 역할에 결박
- ECC: 47개 전문화 서브에이전트
- Compound-Engineering: 14개 병렬 리뷰 에이전트 — security-sentinel, performance-oracle, dhh-rails-reviewer
- revfactory: harness-100의 978개 에이전트 정의
- Ouroboros: 5 lateral thinking personas

**수렴이 놀라운가**: 패턴 자체보다 규모가 놀랍다. gstack의 CEO부터 Compound-Engineering의 DHH까지, **실제 인물/직함을 페르소나로 사용**하는 구체적 접근이 수렴한다는 것이 흥미롭다.

&nbsp;

### CP6. 학습의 아티팩트화 (Compound 루프)

**패턴**: 실행 후 "무엇이 작동했고 무엇이 안 됐는가"를 다음 이터레이션이 읽을 수 있는 파일로 추출한다.

| 하네스 | 구현 | 자동화 수준 |
|--------|------|------------|
| Ralph | `AGENTS.md`에 운영 학습 기록 | 수동 |
| ECC | `/learn` → `/evolve` → instinct store | 반자동 |
| Compound-Engineering | `/ce:compound` → `docs/solutions/` YAML 태깅 | 에이전트 검색 기반 재주입 |
| revfactory | Phase 7 "Harness Evolution" | 피드백 루프 제공 |

&nbsp;

## 4. 발산하는 패턴 (Divergent patterns)

&nbsp;

### DV1. 퍼미션 모델: YOLO vs Least-Privilege

**YOLO 베팅 (Ralph, Compound-Engineering)**: `--dangerously-skip-permissions` 글로벌 bypass. "샌드박스는 운영자 책임."

**Least-Privilege 베팅 (GSD, gstack, Superpowers, Ouroboros, OpenSpec)**: `allowed-tools:` 페이즈 프론트매터 선언 + `/careful`, `/freeze`, `/guard` 방어벽.

**증거**: YOLO 쪽의 실패 사례는 이름 붙여진 실제 사건이다 — Anthropic 공식 Ralph 플러그인이 state 파일을 삭제해 레포 내 Claude를 영구 브릭한 "Hook-brick" 사건. Devon의 "A numerical limit does not prevent an agent from deleting a database in the second iteration." Least-privilege 쪽의 구체적 실패 사례는 노트에서 발견되지 않았다.

**노트 추론**: 증거의 비대칭이 뚜렷하다. 단, YOLO는 그린필드 + 빠른 반복 컨텍스트에서 합리적 선택일 수 있다.

&nbsp;

### DV2. 스펙 위치: 코드 위 vs 코드와 동등 vs 코드 아래

**스펙 우위 (OpenSpec, Ouroboros, GSD)**: specs가 소스 오브 트루스. 모호성 해소 전까지 코드 진입 금지.

**코드가 1차 아티팩트 (Ralph, gstack 일부)**: "완벽한 스펙을 쓰는 것보다 루프를 돌려 빠르게 발견하는 것"이 가치.

**노트 추론**: 스펙 우위 vs 코드 우위의 선택은 프로젝트 복잡도와 관련 있다. OpenSpec이 "브라운필드 + 팀"을 타겟으로 하고, Ralph가 "그린필드 + 솔로"를 타겟으로 한다는 것이 이 분기를 설명한다.

&nbsp;

### DV3. 하네스 학습 자동화 수준

**수동 (Ralph, OpenSpec, gstack)**: 운영자가 실패를 보고 프롬프트를 수정. 에이전트가 학습을 추출하지 않는다.

**자동 (ECC, Compound-Engineering)**: 5-layer observer loop 또는 genetic search가 반복 패턴을 자동 추출.

**노트 추론**: 자동 베팅 쪽의 실패 모드가 더 극적이다 — ECC의 NanoClaw v2 내부 불투명, memory explosion 위험. 자동화가 높을수록 디버깅이 어렵다.

&nbsp;

## 5. 공백 (Gaps)

9개 하네스 어디도 커버하지 못하는 문제 공간.

**GAP1. 멀티-사람 협업 하에서의 에이전트 조율**: 9개 하네스는 모두 솔로 개발자 또는 작은 팀을 암묵적으로 가정한다. 10인 팀, 기능 브랜치 10개, PR 리뷰 관행을 대상으로 설계된 하네스는 없다.

**GAP2. 레거시 코드베이스 대규모 리팩터링**: Ralph는 "There's no way in heck would I use Ralph in an existing codebase"라고 명시한다. OpenSpec이 브라운필드를 타겟으로 하지만 100만 줄 레거시 + 테스트 커버리지 0%에 대한 설계는 없다.

**GAP3. 에이전트 비용 예산 관리**: 멀티-이터레이션 예산("이 세션에서 최대 $X 쓸 것")을 강제하는 하네스는 없다. GSD가 Max 플랜을 권장하지만 이것은 가이드라인이지 메커니즘이 아니다.

**GAP4. 에이전트 행동의 감사 추적 및 설명가능성**: "왜 에이전트가 그 코드를 생성했는가"를 나중에 재현 가능하게 추적하는 감사 인프라를 갖춘 하네스는 없다. 규제 환경(금융, 의료)에서 필수인 것이 모두 미구현 상태다.

**GAP5. 실패 감지 및 자동 롤백**: 루프가 발산하거나 코드베이스를 손상시키는 것을 자동으로 감지하고 알려진 양호 상태로 롤백하는 메커니즘이 9개 하네스 중 어디에도 완전히 구현되지 않았다.

&nbsp;

## 6. 이식 우선순위 매트릭스

전체 9개 노트의 "Transferable primitives" 섹션에서 수렴도·독립성·시도 비용 기준 상위 10개.

| Rank | Primitive | Source harness(es) | Try-cost |
|---|---|---|---|
| 1 | **Fresh context per iteration + file-mediated memory** | Ralph, GSD, Superpowers, Ouroboros | 매우 낮음 — bash 루프 또는 슬래시 커맨드 1개 + PLAN 파일 1개 |
| 2 | **Mode splitting (plan vs build 최소 2-way)** | 9개 전체 | 매우 낮음 — 프롬프트 2개 파일 분리 즉시 적용 가능 |
| 3 | **Design approval gate before any implementation** | Superpowers, OpenSpec, GSD, gstack, Ouroboros | 매우 낮음 — 시스템 프롬프트 1줄 추가 |
| 4 | **Subagent receives only what it needs (carve-off)** | Ralph, GSD, Superpowers, CE | 낮음 — Task() API 있는 런타임에서 즉시 |
| 5 | **CLAUDE.md / AGENTS.md 역할 분리** | Ralph, revfactory, CE | 매우 낮음 — 파일 2개 만들고 분리 기준만 정하면 됨 |
| 6 | **When-only skill description** | Superpowers, revfactory, gstack | 매우 낮음 — 기존 SKILL.md description 재작성만으로 |
| 7 | **Role-scoped review** (페르소나 고정 리뷰어) | gstack, ECC, CE, Ouroboros | 낮음 — 페르소나 정의 1개 + 프롬프트 수정 |
| 8 | **Compound step** (이터레이션 후 학습 아티팩트화) | CE, ECC, Ralph, revfactory | 낮음 — `docs/solutions/` 폴더 + 작성 규약만 정하면 됨 |
| 9 | **`allowed-tools:` frontmatter 페이즈별 권한 선언** | GSD, gstack | 낮음 — SKILL.md에 필드 1개 추가 |
| 10 | **Delta-marker spec format** (ADDED/MODIFIED/REMOVED) | OpenSpec | 매우 낮음 — 스펙 파일 작성 컨벤션 추가만으로 |

**이식 권장 안 함**: `--dangerously-skip-permissions`(실패 사례 다수), 서브에이전트 리뷰 mandatory(Superpowers v5.0.6 인라인 롤백이 비용 문제 증명), NanoClaw v2 블랙박스 채택(내부 불투명).

&nbsp;

## 7. 메타 관찰 (Meta observations)

&nbsp;

### MO1. 저자 배경이 하네스 포지션을 거의 완벽하게 예측한다

Ralph (Huntley — 독립 개발자, "I don't code, I vibe") → 최소주의, YOLO, 그린필드 반복.
Superpowers (Vincent — 오픈소스 개발자, chardet 저자) → TDD 강제, 프로세스 디시플린.
gstack (Garry Tan — YC CEO) → 역할 분리, 의사결정 레이어, "스타트업 팀처럼 운영하라".
Ouroboros (Q00 — 서울, ZEP 테크리드) → 입력 모호성 우선, 소크라테스적 면접, 정량 임계값.
revfactory (황민호 — 카카오 AI Native Strategy 팀 리더) → 메타-스킬, 팀 아키텍처 자동화, 논문 수준 실험.
Compound-Engineering (Klaassen/Shipper — Every.to) → 학습 복리, 장기 프로덕트 운영.

**노트 추론**: 이것은 "어떤 하네스가 더 좋은가"라는 질문이 잘못 설정되어 있다는 것을 시사한다 — 어떤 컨텍스트에 있는 누가 쓰는가가 더 적절한 질문이다.

&nbsp;

### MO2. 증거 품질이 채택 규모와 반비례하는 경향

채택 규모 순: ECC(140k stars) → gstack(71k) → GSD(51k) → OpenSpec(39k) → Compound-Engineering(14k) → revfactory(2.4k) → Ouroboros(2.3k) → Ralph(1.6k playbook fork).

통제된 벤치마크 존재 여부: **revfactory만 존재**(15 태스크, 논문화). 나머지 8개는 일화 + 스타수 + 스크린샷.

**노트 추론**: 커뮤니티가 실증보다 저자 플랫폼(YC 대표, X 바이럴)과 설치 편의성에 더 강하게 반응한다는 것을 보여준다. 가장 증거가 약한 하네스가 가장 많이 채택된다는 역설이 지속되고 있다.

&nbsp;

### MO3. 한국 저자 2명이 가장 독창적인 포지션을 차지

9개 하네스 중 한국 저자는 Ouroboros(Q00, 서울, ZEP)와 revfactory(황민호, 제주, 카카오) 2개다. 이 둘이 다른 7개 하네스와 가장 다른 포지션을 차지한다.

- Ouroboros: 다른 8개 하네스 중 어느 것도 "입력 모호성을 숫자로 스코어링해 임계값 이하면 코드 진입 금지"를 1차 문제로 설정하지 않는다.
- revfactory: "하네스를 생성하는 하네스"는 다른 8개 어디에도 없다. 15 태스크 통제 실험 + 논문화도 유일.

**노트 추론**: 가장 혁신적인 접근이 가장 낮은 가시성을 가진다는 패턴이 반복된다.

&nbsp;

### MO4. 타이밍 패턴: 2025 하반기 폭발, 2026 수렴

- 2025-07: Ralph — 원시적 while loop
- 2025-10: Superpowers — SKILL.md 컨벤션 확립
- 2025-12: Compound-Engineering — 학습 루프 개념
- 2026-01: ECC, OpenSpec, Ouroboros — 다층 번들/스펙 관리/소크라테스 게이트
- 2026-02~03: gstack, GSD — 역할 페르소나/컨텍스트 관리 대중화
- 2026-04 (분석 시점): revfactory — 메타-스킬 + 논문화

**노트 추론**: Ralph가 "최소주의 가능성"을 증명하자 6개월 내에 복잡도가 급증했다. Superpowers의 SKILL.md 컨벤션이 사실상 표준으로 자리 잡은 것이 2025-10 이후 5개 하네스가 같은 포맷을 채택한 것으로 보인다. revfactory의 논문화는 "바이럴이 아닌 증거로 승부"하는 다른 경로의 시작일 수 있다.

&nbsp;

### MO5. 스태킹 담론이 커뮤니티에서 먼저 출현했다

하네스들 자신이 서로를 대체제로 포지션하지 않는다. 오히려 커뮤니티 관찰자(tentenco/Ewan Mak, dev.to imaginex, heyuan110)가 "gstack thinks, GSD stabilizes, Superpowers executes" 같은 스태킹 레시피를 먼저 개발했다. 하네스 저자들은 자신의 사용 사례에 집중하고, 커뮤니티가 조합법을 발견한다.

**노트 추론**: harness landscape이 winner-takes-all 경쟁이 아니라 레이어별로 특화된 도구들의 생태계로 수렴하고 있음을 시사한다.

&nbsp;

### MO6. "완전한 워크플로"의 정의 불일치

- gstack: Build 단계 skill 없음. 가장 중요한 실행 단계에서 Claude Code가 기본 모드로 돌아간다.
- Ralph: 그린필드 90% ceiling. 마지막 10%는 사람이 마감.
- GSD 단독: "does not directly produce code, run tests, or open a PR." Context anchor 역할이지 shipping 도구가 아님.
- OpenSpec: 단일 에이전트 모델 + 코드 리뷰 내장 없음.

**노트 추론**: 9개 하네스 중 어느 것도 "다 커버"하지 못한다. 이것은 설계 실패가 아니라 포지셔닝의 정직함일 수도 있다. 그러나 이 공백을 메우기 위해 사용자가 조합하거나 수동으로 처리해야 한다는 것은 채택 비용의 숨겨진 요소다.
