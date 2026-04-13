---
title: "Ouroboros — 모호성을 숫자로 수렴시키는 루프"
date: 2026-04-13
categories: [AI 시스템 리서치, 하네스 엔지니어링]
tags: [하네스엔지니어링, Ouroboros, MCP, 스펙주도개발, 정량게이팅]
---

## 한 줄로

> "Stop prompting. Start specifying."

Ouroboros 는 이 한 문장으로 자기를 소개한다. 2026년 1월 14일 Q00 (Seoul 거주, ZEP Tech Lead) 이 공개했고, 사용자와 Claude Code / Codex CLI / OpenCode 같은 AI 런타임 사이에 **MCP 서버**로 끼어들어 `interview → seed → execute → evaluate → evolve` 5단계 사이클을 강제한다. 이름은 "자기 꼬리를 먹는 뱀" — 평가 결과가 다음 generation 의 스펙으로 다시 들어간다는 은유다.

&nbsp;

## 왜 태어났는가

Q00 의 진단은 Ralph나 GSD와 또 다른 각도를 찍는다. Huntley 는 "완벽한 프롬프트 집착" 을, TÂCHES 는 "context rot" 을 적으로 지목했다. Q00 가 지목한 적은 **인풋의 모호성**이다.

> "Most AI coding fails at the **input**, not the output. The bottleneck is not AI capability -- it is human clarity." — README

즉 코드가 실패하는 건 모델이 멍청해서가 아니라 사람이 무엇을 만들고 싶은지 스스로 모르기 때문이라는 주장. 해법도 자연스럽다. 모델에게 "뭘 만들어 줘" 라고 말하는 대신, **모델이 먼저 Socratic 질문으로 사용자를 심문**해서 숨은 가정을 전부 꺼내게 하고, 그 결과를 불변 스펙으로 굳힌 뒤에야 코드 생성으로 넘어간다.

&nbsp;

## 실제로 어떻게 돌아가는가

`ooo evolve "build a task CLI"` 한 줄로 트리거하면:

1. **Interview** — Socratic 면접이 시작된다. 매 질문은 4-path 라우팅 중 하나로 답한다 — 코드 확인 / 사용자 판단 / 둘 다 / 리서치. "3 연속 코드·리서치 답변이면 다음 질문은 **반드시** 사용자에게" 라는 dialectic rhythm guard 가 박혀 있다. 코드 탐험에 매몰되어 사람을 잊지 말라는 뜻
2. 면접은 **ambiguity score ≤ 0.2** 가 될 때까지 반복된다
3. **Seed** — 면접 결과가 YAML 스펙으로 crystallize 된다. 7개 필드: GOAL / CONSTRAINTS / ACCEPTANCE_CRITERIA / ONTOLOGY_SCHEMA / EVALUATION_PRINCIPLES / EXIT_CONDITIONS / METADATA. 한 번 굳어진 seed 는 **불변**
4. **Execute** — Double Diamond 분해로 실행. 첫 diamond 는 Socratic (Wonder → Ontology), 둘째 diamond 는 Pragmatic (Design → Evaluation)
5. **Evaluate** — 3-stage 검증이 순차로: Mechanical (lint/build/test, $0) → Semantic (AC 준수, 드리프트 측정) → Multi-Model Consensus (Frontier tier, 불확실할 때만)
6. **Evolve** — Wonder 와 Reflect 사이클이 다음 generation 의 seed 를 만든다. 여기서 우로보로스가 자기 꼬리를 먹는다

루프의 종료 조건은 셋 중 하나 — ontology similarity 가 **≥ 0.95** 가 되면 `converged`, 30 세대에 도달하면 `exhausted`, 3 세대 연속 불변이면 `stagnated`.

`ooo unstuck` 은 작지만 예쁜 primitive 다. 5개의 lateral thinking 페르소나 중 하나를 주입한다: **Hacker**("작동하게 먼저"), **Researcher**("뭘 모르고 있지?"), **Simplifier**("범위 자르고 MVP 로"), **Architect**("접근 자체를 다시 짜자"), **Contrarian**("잘못된 문제를 풀고 있는 건 아닐까?").

&nbsp;

## 왜 작동한다고 (Q00 는) 주장하는가

증거는 얇다. 이게 정직한 평가다.

README 에는 "12 hidden assumptions exposed, ambiguity scored to 0.19" 같은 한 줄 예시가 있고, "rework rate Low vs vanilla AI coding" 이라는 정성 비교 테이블이 있다. 숫자 수사(0.2 / 0.95 / 0.3) 가 풍성해서 정량 지향으로 보이지만, 정작 그 숫자들의 **측정 함수는 블랙박스**고 벤치마크는 없다.

3자 재현·리뷰도 Ralph / Superpowers / GSD 대비 **가장 얇다**. 그 빈자리를 대신 메우는 건 활발한 릴리즈 페이스 — 약 2주 동안 마이너 10개 릴리즈. 지금 성장 중인 하네스라는 얘기고, 역으로 지금 막 써보기에는 이르다는 얘기이기도 하다.

&nbsp;

## 진짜 핵심 아이디어 하나

Ouroboros 가 자기를 판매하는 방식("스펙 우선") 과, 실제로 갖고 있는 가장 독창적인 primitive 는 다르다.

진짜 핵심은 **"모호성을 숫자로 재단해 게이트 구문으로 쓴다"** 는 점이다. Superpowers 의 `<HARD-GATE>` XML 태그나 Ralph 의 CAPS 호통과 달리, Ouroboros 의 게이트는 ambiguity ≤ 0.2, similarity ≥ 0.95, drift ≤ 0.3 같은 **정량 임계값**이다. 모델 출력을 "통과 / 차단" 두 상태로 가르는 기준이 숫자라는 것 — 이 아이디어 하나가 계보 상 새롭다. 다른 하네스는 정성 게이트를 쓴다.

이 정량 게이트 발상은 이식 가능하지만, 측정 함수의 품질은 이식 불가능하다. "ambiguity 를 뭐라고 정의하고 어떻게 재는가" 라는 핵심 질문에 Ouroboros 는 코드로만 답하고 스펙 레이어에서는 답하지 않는다.

&nbsp;

## 가져갈 만한 것들

딥다이브 노트에서 뽑은 가장 주목할 5개:

1. **Ambiguity-as-gate** — 인풋 모호성을 숫자 임계값으로 차단. 측정 함수는 프로젝트에 맞게 커스텀 — LLM-as-judge, 구조 검증, 체크리스트 매칭 중 고르면 된다
2. **Seed = immutable 7-field YAML 계약** — GOAL / CONSTRAINTS / ACCEPTANCE_CRITERIA / ONTOLOGY_SCHEMA / EVALUATION_PRINCIPLES / EXIT_CONDITIONS / METADATA. 스키마가 명시적이고 불변
3. **Socratic interview + dialectic rhythm guard** — 3 연속 코드/리서치 확인이면 다음 질문은 강제로 사람에게. 순수 프롬프트 레이어로 이식 가능
4. **5 lateral thinking personas as escape hatch** — stagnation 감지 시 Hacker/Researcher/Simplifier/Architect/Contrarian 중 하나를 주입. 구조화된 "막혔을 때 이렇게 생각해봐" 메뉴
5. **Path A / Path B skill duality** — MCP 툴 있으면 Path A, 없으면 agent 채택해서 Path B. 플러그인 의존과 self-contained 폴백을 같은 스킬 안에 넣는 컨벤션

&nbsp;

## 조심할 것

GitHub 이슈 트래커가 가장 솔직한 소스다:

- **Issue #371 — parallel session 의 429 cascade**: 병렬 세션을 스폰하면 rate-limit 토큰 버킷이 공유되지 않아 연쇄 폭탄
- **Issue #369 — AC 트리 무한 분해**: Acceptance Criteria 가 재귀적으로 쪼개지다가 3분 만에 `ac_3000002` 같은 300만 노드에 도달
- **Issue #341 — 서브프로세스 누수**: 취소된 job 이 Claude CLI 서브프로세스를 살려둔다
- **Issue #364 — interview 피로도**: 3+ 라운드 면접이 과한 경우 있음

그리고 구조적 한계:

- **Ambiguity score 가 블랙박스**: 0.2 임계값이 게이트지만 "왜 0.19 인가" 를 사용자가 검증할 수 없다
- **3자 검증 부재**: Ralph/Superpowers/GSD 대비 외부 재현 보고가 사실상 없음. 지금 프로덕션 의존 대상으로 쓰기엔 이르다
- **30-generation 상한은 cost cap 이지 safety control 이 아니다**

그리고 팔지 않을 게 하나 있다. **seed crystallize 직후 GitHub star 를 요구하는 gate**. Full Mode 잠금해제를 star 선조건처럼 제시하는 건 하네스 primitive 가 아니라 distribution tactic 이다. 이식할 때 버릴 것.

&nbsp;

## 어디에 쓰고 어디에 쓰지 말까

**쓸 만한 영역**
- 요구사항이 애매한 상태로 Claude 에게 던져지는 패턴을 개선하고 싶을 때
- 스펙을 구조화된 불변 YAML 로 관리하고 싶은 프로젝트
- Socratic 면접 단계나 unstuck 의 5 페르소나 같은 것만 독립 primitive 로 추출해 쓸 때
- MCP 스택을 이미 운영 중이고 서버 관리 복잡도를 감당할 수 있는 팀

**쓰면 안 되는 영역**
- 그린필드 부트스트랩 속도가 최우선일 때 (Ralph 가 훨씬 가볍다)
- 3자 검증 부족이 신경 쓰이는 프로덕션 파이프라인
- 파일 기반 감사 추적이 필요한 환경 (권위적 상태가 EventStore라 `git log` 로 못 본다)

&nbsp;

## 더 읽을거리

- **Q00 본인** — [레포](https://github.com/Q00/ouroboros), [PyPI 패키지](https://pypi.org/project/ouroboros-ai/), [릴리즈 노트](https://github.com/Q00/ouroboros/releases)
- **스킬 내부** — [seed/SKILL.md](https://github.com/Q00/ouroboros/blob/main/skills/seed/SKILL.md), [evolve/SKILL.md](https://github.com/Q00/ouroboros/blob/main/skills/evolve/SKILL.md), [unstuck/SKILL.md](https://github.com/Q00/ouroboros/blob/main/skills/unstuck/SKILL.md)
- **실패 모드 추적** — [open issues](https://github.com/Q00/ouroboros/issues)

&nbsp;

## 인사이트 정리

Ouroboros를 공부하면서 건진 것들을 세 줄로 압축하면:

**정량 게이트라는 발상이 새롭다.** 다른 하네스가 HARD-GATE 태그나 CAPS 호통 같은 정성 게이트를 쓰는 반면, Ouroboros는 ambiguity ≤ 0.2, similarity ≥ 0.95 같은 숫자로 통과 여부를 결정한다. 이 발상 자체는 런타임과 무관하게 이식할 수 있다.

**측정 함수가 전제다.** 정량 게이트의 가치는 측정 함수의 신뢰도에 달려 있다. "모호성을 어떻게 재는가"라는 질문에 Ouroboros는 코드로만 답한다. 이식할 때는 게이트 아이디어를 가져오되 측정 함수는 직접 정의해야 한다.

**Socratic 면접과 unstuck 페르소나는 독립 이식이 가능하다.** 전체 시스템을 도입하지 않아도 이 두 primitive만 떼어내면 쓸 수 있다. 특히 "막혔을 때 어떤 페르소나로 접근할지"를 구조화해두는 아이디어는 프레임워크 없이도 바로 적용된다.
