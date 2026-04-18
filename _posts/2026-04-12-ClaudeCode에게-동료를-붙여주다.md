---
title: "ClaudeCode에게 동료를 붙여주다"
date: 2026-04-12
categories: [개발일지, GameMakerEngine]
tags: [GameMakerEngine, Codex, 멀티에이전트, 비용최적화, AI에이전트]
---

## 두 개의 벽

엔지니어링 플래너를 만들면서 Context 문제가 해결될 거라 생각했다. 에이전트가 매 세션마다 시스템 상태를 깊게 분석하고, 명확한 다음 과제를 받아 출발한다면 더 이상 길을 잃지 않을 것이라는 기대였다.

실제로는 그렇지 않았다. 플래너가 올바른 방향을 가리켜도, 그 방향을 따라 조금만 걸어가면 Context가 터졌다. 지도를 쥐어줬지만, 지도를 읽으면서 동시에 짐을 지고 달리기엔 한계가 있었다.

그래서 모든 구현 작업을 메인 에이전트 대신 서브에이전트로 위임하는 방식을 시도했다. 메인은 지시만, 서브는 실행만. 절반은 통했다. 기존보다 몇 번의 대화를 더 버틸 수 있었다. 하지만 Context가 터지는 시점이 조금 늦춰졌을 뿐, 결국 같은 벽에 부딪혔다. 그리고 이 방식은 또 다른 문제를 더욱 가속화시켰다.

사실 사용량 문제는 이전부터 있었다. GameMakerEngine은 경쟁 사이클을, GameMaker는 게임을 만들어내는 과정에서 막대한 토큰을 소모하는 탓에, MAX 20x 플랜 위에 매주 약 100달러의 추가 사용량을 구매하고 있었다. 거의 모든 업무를 서브에이전트에게 위임하는 방식은 그 소모를 배로 끌어올리며 문제를 더욱 악화시켰다. 이 개발이 언제 끝날지 모르는 상황에서, 비용을 더 늘려가는 건 지속 가능한 구조가 아니었다.

Context 폭발과 비용 과소모. 두 개의 벽이 동시에 서 있었다.

&nbsp;

## 두 가지 해결책

두 문제는 각각 다른 원인에서 비롯됐고, 해결책도 달랐다.

비용 문제의 핵심은 모든 작업이 Claude 한 곳에 몰린다는 데 있었다. 이건 Provider Router로 풀었다. 태스크의 성격에 따라 Claude와 Codex 중 어느 쪽이 처리할지 선택하는 라우팅 레이어를 Engine과 GameMaker 내부에 구성해, 구현 중심의 반복 작업은 Codex로 분산했다. Claude 사용량이 줄고, 비용 구조에 숨통이 트였다.

Context 폭발 문제의 핵심은 Engine 혼자 너무 많은 걸 짊어진다는 데 있었다. 무엇을 개발해야 하는지 계획하고 진행 상황을 관리하는 기능을 외부 Codex 프로젝트에 위임하고, Engine은 짧은 세션 사이클을 반복하며 구현에만 집중하는 구조로 바꿨다. Context가 터지기 전에 세션을 끊고 다음 사이클에서 이어받는다면, 거대한 코드베이스도 감당할 수 있었다.

&nbsp;

## 세 가지 장치

이 두 해결책을 실제로 구현한 장치가 셋이다.

첫 번째는 **Provider Router**다. Engine과 GameMaker 내부에 LLM 라우팅 레이어를 만들어, 태스크의 성격에 따라 Claude와 Codex 중 어느 쪽이 처리할지 선택하도록 했다. 무거운 판단이 필요한 작업은 Claude가, 구현 중심의 반복적인 작업은 Codex가 담당한다.

두 번째는 **Helper**다. GameMakerEngineHelper는 독립된 Codex 프로젝트로, GameMakerEngine을 바깥에서 바라보는 에이전트다. 매 세션마다 새로운 눈으로 코드베이스를 분석하고, 무엇을 개발해야 하는지 계획과 진행 상황을 관리한다. Context를 공유하지 않기 때문에, 메인 에이전트가 쌓아온 맥락의 무게에 눌리지 않는다.

세 번째는 **Monitor**다. Engine(Claude)과 Helper(Codex)가 실제로 협업하려면 두 에이전트 사이를 연결하는 장치가 필요하다. Monitor는 그 연결고리다. 짧은 세션 사이클을 주고받으며, 한쪽이 분석한 결과를 다른 쪽이 이어받아 작업을 진행할 수 있도록 흐름을 만든다.

&nbsp;

## 혼자에서 협업으로

이 세 가지를 묶어서 보면, 결국 "Claude 혼자 감당하는 구조"에서 "에이전트들이 역할을 나눠 협업하는 구조"로의 전환이다.

비용이 한 모델에 몰리는 문제는 Provider Router가 작업을 분산함으로써 해소한다. Context 폭발 문제는 Helper가 계획과 관리를 맡고 Engine이 짧은 사이클로 구현에 집중함으로써 우회한다. 두 에이전트가 따로 놀지 않도록 Monitor가 연결한다.

세션을 홀로 감당하던 Claude Code에게, Codex라는 동료를 붙여줬다.

&nbsp;

---

<details>
<summary><strong>부록: Provider Router / Helper / Monitor 구축 타임라인</strong> (Git Storyteller 커밋 데이터)</summary>
<div markdown="1">

> 아래는 Git Storyteller가 커밋 히스토리에서 추출한 데이터를 바탕으로 작성된 내용이다.
{: .prompt-info }

**Provider Router (GameMakerEngine — 04-05 ~ 04-12)**

| 날짜 | 작업 |
|------|------|
| **04-05** | Provider Router System — Claude/Codex 간 태스크별 LLM 라우팅 + 사용량 기반 자동 전환 |
| **04-05** | 라우팅 프리셋 3종 추가 + max-codex 활성화 — Claude 사용량 절약 |
| **04-06** | Provider Router 확장 — 교차 검증 협업 모드 + Provider Intelligence |
| **04-07** | Codex 모델 split — gpt-5.4 / gpt-5.4-mini 태스크별 분기 |
| **04-07** | Claude Opus/Sonnet split — 태스크별 모델 분기 |
| **04-08** | feat(codex): CodexAdvisor — Codex CLI 전문가 모듈 추가 |

**Helper 구축 (GameMakerEngineHelper_Codex — 04-11 ~ 04-12)**

| 날짜 | 작업 |
|------|------|
| **04-11** | Initial commit — 협업 자동화 허브 초기 구조 (40파일, +3,794줄) |
| **04-11** | feat(helper): 협업 자동화 + 모니터 툴링 추가 — `helper_cycle.py`(964줄), `collab_runtime.py`, `monitor_collab.py` |
| **04-12** | chore(helper): GameMaker CLI evidence 프로파일 확장 |

**Monitor 구축 (GameMakerCollabMonitor_Codex — 04-12)**

| 날짜 | 작업 |
|------|------|
| **04-12** | feat(monitor): 오퍼레이터 콘솔 워크플로우 추가 — 사람이 개입할 수 있는 결정 지점 설계 |
| **04-12** | feat(monitor): 엔진 핸드오프 컨트롤 플레인 추가 — Engine↔Monitor 메시지 스키마 계약화 |

Provider Router로 비용 분산 구조를 먼저 잡고, Helper로 분석/계획 레이어를 세운 뒤, Monitor로 두 에이전트의 협업 흐름을 정식화하는 순서로 구조가 쌓였다.

</div>
</details>
