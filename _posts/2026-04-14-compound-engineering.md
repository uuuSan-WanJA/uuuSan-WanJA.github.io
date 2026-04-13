---
title: "Compound Engineering — 이터레이션이 이터레이션을 가르친다"
date: 2026-04-14
categories: [AI 시스템 리서치, 하네스 엔지니어링]
tags: [하네스엔지니어링, CompoundEngineering, ClaudeCode, 기관지식, 누적학습]
---

## 한 줄로

> "Each unit of engineering work should make subsequent units easier—not harder."

Plan → Work → Review → **Compound**. 4단계 루프인데, 처음 세 단계는 어디서든 본 것들이다. 마지막 한 단계가 이름을 결정한다.

&nbsp;

## 왜 태어났는가

Every.to의 Kieran Klaassen이 제안하고 CEO Dan Shipper와 함께 사내에서 굳힌 방법론이다. 2025년 12월 블로그 포스트로 처음 공개됐고 이후 GitHub 오픈소스 플러그인으로 배포됐다. 2026년 4월 기준 ⭐14.2k.

Klaassen의 문제의식은 이렇다: AI 도움 받아 코드를 쓰면 빨리 쓸 수 있다. 하지만 피처가 쌓일수록 코드베이스는 여전히 무거워진다. 기관 지식은 여전히 특정 엔지니어의 머릿속에 있다. 새로운 버그 유형이 생기면 같은 실수가 반복된다. AI가 코드를 써준다는 사실만으로는 이 문제가 해결되지 않는다.

Klaassen이 뒤집고 싶은 질문은 하나였다: **"매 이터레이션이 다음 이터레이션을 더 어렵게 만드는 대신, 더 쉽게 만들 수 있는가?"**

&nbsp;

## 실제로 어떻게 돌아가는가

하나의 피처를 짠다고 가정하자.

1. **Brainstorm** — `/ce:brainstorm`으로 요구사항이 불명확한 부분을 Q&A로 좁힌다
2. **Plan** — `/ce:plan`을 실행하면 3개 병렬 리서치 에이전트가 코드베이스·외부 문서를 조사하고, 목표·아키텍처·수정 파일 목록·성공 기준이 담긴 계획 문서를 생성한다. 계획이 코드보다 먼저 나온다
3. **Work** — `/ce:work`로 에이전트가 계획을 따라 구현한다. git worktree로 격리, 변경마다 테스트·린터·타입 체크
4. **Review** — `/ce:review`로 14개 전문 에이전트가 병렬 심사한다. security-sentinel(OWASP), performance-oracle(N+1 쿼리), dhh-rails-reviewer(Rails 관습)... 결과는 P1(필수)/P2(권장)/P3(선택)로 우선순위화된다
5. **Compound** — `/ce:compound`가 이번 이터레이션에서 무엇을 배웠는지 마크다운으로 정리한다. YAML frontmatter로 태깅해 `docs/solutions/`에 저장하고, 핵심 패턴은 `CLAUDE.md`에 추가한다
6. 루프 반복. 다음 플래닝 단계에서 베스트프랙티스 에이전트가 `docs/solutions/`에서 관련 패턴을 골라 컨텍스트에 주입한다

그리고 `/lfg` ("Let's F'ing Go")라는 커맨드가 있다. 아이디어 하나를 입력하면 50개 이상의 에이전트가 plan→build→review→fix→merge를 처리하고 스크린샷, 영상, 브라우저 테스트가 붙은 PR을 내놓는다.

&nbsp;

## 왜 작동한다고 (저자는) 주장하는가

Every.to는 5개 프로덕션 제품을 이 방법으로 운영 중이며, 각 제품당 주로 엔지니어 1인이 담당하고 수천 명이 매일 사용한다. "A single developer can do the work of five developers a few years ago."

구체적인 비율: 80%의 엔지니어링 시간을 Plan과 Review에, 20%를 Work와 Compound에 쓴다. 전통적인 90% 구현 + 10% 리뷰를 뒤집은 것이다.

증거 유형은 내부 일화와 채택 지표다. 통제된 벤치마크는 없다.

&nbsp;

## 진짜 핵심 아이디어 하나

Compound Engineering을 "AI 코딩 워크플로"로 이해하면 절반만 본 것이다.

핵심은 **하네스 자체가 이터레이션마다 재구성된다**는 점이다. 매 Compound 단계가 `CLAUDE.md`와 `docs/solutions/`를 갱신하고, 다음 플래닝 에이전트는 그 갱신된 컨텍스트를 출발점으로 삼는다. 이전 이터레이션이 발견한 버그 패턴을 다음 이터레이션이 자동 회피한다.

Klaassen이 직접 표현한 방식:

> "each bug, failed test, or a-ha problem-solving insight gets documented and used by future agents"

Angelo Lima의 분석이 정확하게 지적했다: "Without it, you're just doing traditional AI-assisted development." Compound 단계는 선택이 아니라 이 방법론의 정체성이다.

&nbsp;

## 가져갈 만한 것들

딥다이브 노트에서 가장 독립적으로 이식 가능한 것들:

1. **학습의 아티팩트화** — 매 이터레이션 후 "무엇이 됐고 무엇이 안 됐나"를 YAML 태깅 마크다운으로 저장. `docs/solutions/`가 쌓이면 다음 에이전트가 같은 실수를 안 한다. 특정 런타임이나 플러그인 없이도 적용 가능한 원칙
2. **선택적 컨텍스트 주입(Genetic search)** — 누적 문서를 전부 컨텍스트에 밀어 넣지 말고, 전용 에이전트가 관련 패턴만 골라 주입. 컨텍스트 폭발 없이 기관 기억 활용하는 설계 원칙
3. **80/20 위상 역전** — 구현보다 계획+리뷰에 더 많이 투자. 에이전트가 구현을 담당하면 인간의 비교 우위는 설계와 판단으로 이동한다
4. **역할 페르소나 병렬 리뷰** — 범용 리뷰어 하나 대신, 도메인 전문 페르소나(security/performance/style)가 병렬 심사하고 P1/P2/P3로 우선순위화. 리뷰 결과가 actionable해진다
5. **CLAUDE.md를 살아있는 시스템 스펙으로** — 단순 지시 파일이 아니라 매 Compound 단계에서 갱신되는 누적 지성

&nbsp;

## 조심할 것

**저자가 직접 인정한 것들**:
- Compound 단계 규율 의존 — 바쁘면 스킵되고, 스킵되면 일반 AI 보조 개발과 차이 없다
- `--dangerously-skip-permissions` 조건부 사용 권고 — 프로덕션 코드이거나 롤백이 안 되는 환경에서는 쓰지 말 것

**3자 관찰**:
- 업스트림 스펙 부재 — 복잡한 요구사항을 다루기에는 계약적 엄밀성이 부족. SDD 계열(OpenSpec 등)과 결합으로 보완 가능
- **CLAUDE.md 비대화** — 이터레이션마다 학습이 추가되면 크기가 컨텍스트 창을 침식할 수 있다. Genetic search가 완화책이지만 장기 운영 시 크기 관리 정책 필요
- `/lfg` 비용 — 50+ 에이전트 실행의 실제 API 비용 사례가 없다

&nbsp;

## 어디에 쓰고 어디에 쓰지 말까

**쓸만한 영역**
- 팀 기관 지식이 특정 사람에게 집중되어 있는 조직
- 같은 종류의 버그가 반복되는 코드베이스
- 1~3인 소규모 팀이 여러 제품을 병렬 유지해야 하는 상황
- 신입 온보딩 비용이 큰 프로젝트

**쓰면 안 되는/조심해야 하는 영역**
- Compound 단계를 꾸준히 실행할 여유가 없는 팀 (half-applied는 보통 악화)
- 감사 추적이 필수인 규제 환경
- 실험적 탐색 단계 — 가이드 자체가 "vibe code to discover; spec to build properly; delete prototypes and replan"이라고 명시

&nbsp;

## 더 읽을거리

- **저자** — [원글 (Chain of Thought, 2025-12-11)](https://every.to/chain-of-thought/compound-engineering-how-every-codes-with-agents), [공식 가이드](https://every.to/guides/compound-engineering)
- **플러그인** — [EveryInc/compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin)
- **비교 분석** — [Angelo Lima: SDD vs Compound Engineering vs BMAD](https://angelo-lima.fr/en/sdd-compound-engineering-bmad-philosophies-en/)

&nbsp;

## 인사이트 정리

Compound Engineering을 공부하면서 건진 것들을 세 줄로 압축하면:

**Compound 단계가 빠지면 그냥 일반 AI 보조 개발이다.** Plan/Work/Review는 어디서든 본 것들이다. "이터레이션이 다음 이터레이션을 가르친다"는 약속은 Compound 단계에서만 성립한다. 이것이 선택이 아니라 방법론의 정체성이라는 점을 빠뜨리면 절반을 오해한 것이다.

**학습의 아티팩트화는 런타임과 무관하다.** `docs/solutions/`에 YAML 태깅 마크다운으로 패턴을 쌓고, 다음 플래닝 에이전트가 관련 패턴만 선택적으로 가져오는 구조. 이 원칙은 어떤 하네스에도 이식 가능하다.

**80/20 위상 역전이 핵심 주장이다.** 에이전트가 구현을 담당하면 사람의 비교 우위는 설계와 판단으로 이동한다. "Plans are the new code."라는 말은 단순 슬로건이 아니라 시간 배분을 바꾸라는 실질적 제안이다.
