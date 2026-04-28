# Dive into Claude Code

<p align="center">
  <img src="./assets/main_structure.png" width="85%" alt="High-level system structure of Claude Code">
</p>

<p align="center">
  <a href="./paper/Dive_into_Claude_Code.pdf"><img src="https://img.shields.io/badge/Paper-PDF-blue.svg?logo=adobeacrobatreader&logoColor=white" alt="Paper"></a>
  <a href="https://arxiv.org/abs/2604.14228"><img src="https://img.shields.io/badge/arXiv-2604.14228-b31b1b.svg" alt="arXiv"></a>
  <a href="./LICENSE"><img src="https://img.shields.io/badge/License-CC--BY--NC--SA--4.0-lightgrey.svg" alt="License"></a>
  <a href="https://github.com/VILA-Lab/Dive-into-Claude-Code/stargazers"><img src="https://img.shields.io/github/stars/VILA-Lab/Dive-into-Claude-Code?style=social" alt="Stars"></a>
</p>

<p align="center">
  <a href="./README.md">English</a> | <b>한국어</b>
</p>

> **Claude Code(v2.1.88, 약 1,900개의 TypeScript 파일, 약 512K 라인 규모)에 대한 종합적인 소스 레벨 아키텍처 분석과 함께, 커뮤니티 분석 자료의 큐레이션, 에이전트 빌더를 위한 디자인 스페이스 가이드, 그리고 다른 시스템과의 비교를 한데 모은 자료입니다.**

> [!TIP]
> **TL;DR** -- Claude Code 코드베이스에서 AI 의사결정 로직이 차지하는 비율은 단 1.6%에 불과합니다. 나머지 98.4%는 결정론적 인프라 -- 권한 게이트, 컨텍스트 관리, 도구 라우팅, 복구 로직 -- 입니다. 에이전트 루프 자체는 단순한 while 루프이며, 진짜 엔지니어링 복잡도는 그 주변을 둘러싼 시스템에 자리합니다. 이 저장소는 그 아키텍처를 해부하고, AI 에이전트 시스템을 만드는 모든 이를 위한 실용적인 디자인 가이드로 정제합니다.

---

## 목차

**논문에서 발췌**

- [🌟 핵심 하이라이트](#key-highlights)
- [📖 읽기 가이드](#reading-guide)
- [🏗️ 아키텍처 개요](#architecture-at-a-glance)
- [🧭 가치와 설계 원칙](#values-and-design-principles)
- [🔄 에이전트 쿼리 루프](#the-agentic-query-loop)
- [🛡️ 안전성과 권한](#safety-and-permissions)
- [🧩 확장성](#extensibility)
- [🧠 컨텍스트와 메모리](#context-and-memory)
- [👥 서브에이전트 위임](#subagent-delegation)
- [💾 세션 영속성](#session-persistence)

**논문을 넘어서**

- [🛠️ AI 에이전트 직접 만들기: 디자인 가이드](#build-your-own-ai-agent-a-design-guide)
- [🌐 커뮤니티 프로젝트 & 연구](#community-projects--research)
- [🚀 그 외 주목할 만한 AI 에이전트 프로젝트](#other-notable-ai-agent-projects)
- [🔖 인용](#citation)

---

## 핵심 하이라이트

- **인프라 98.4%, AI 1.6%** -- 에이전트 루프는 단순한 while 루프이며, 진짜 복잡도는 권한 게이트, 컨텍스트 관리, 복구 로직에 있다.
- **5가지 가치 → 13가지 원칙 → 구현** -- 모든 설계 결정은 인간의 권한, 안전성, 신뢰성, 역량, 적응성으로 거슬러 올라간다.
- **공유 실패 모드를 가진 심층 방어(defense in depth)** -- 7개의 안전 레이어가 있지만 모두 동일한 성능 제약을 공유한다. 50개를 초과하는 서브커맨드는 보안 분석을 우회한다.
- **4개의 CVE가 드러낸 사전 신뢰 윈도우** -- 익스텐션은 신뢰 다이얼로그가 표시되기 *전*에 실행된다.

> **[번역 주석]** CVE(Common Vulnerabilities and Exposures)는 공개적으로 알려진 보안 취약점에 부여되는 표준 식별자 체계입니다.

- **교차 관심사 하네스(harness)는 재구현에 저항한다** -- 루프는 복사하기 쉽지만, 훅(hook), 분류기, 컴팩션(compaction), 격리는 그렇지 않다.

---

## 읽기 가이드

| 당신이... | 여기서 시작 | 그다음 읽을 것 |
|:----------------|:-----------|:----------|
| **에이전트 빌더** 라면 | [나만의 에이전트 만들기](./docs/build-your-own-agent_KR.md) | [아키텍처 심층 분석](./docs/architecture_KR.md) |
| **보안 연구자** 라면 | [안전성과 권한](#safety-and-permissions) | [아키텍처: 안전 레이어](./docs/architecture_KR.md#seven-independent-safety-layers) |
| **프로덕트 매니저** 라면 | [핵심 하이라이트](#key-highlights) | [가치와 원칙](#values-and-design-principles) |
| **연구자** 라면 | [전체 논문 (arXiv)](https://arxiv.org/abs/2604.14228) | [커뮤니티 리소스](#community-projects--research) |

`1,884 files` ·  `~512K lines` ·  `v2.1.88` ·  `7 safety layers` ·  `5 compaction stages` ·  `54 tools` ·  `27 hook events` ·  `4 extension mechanisms` ·  `7 permission modes`

---

<details open>
<summary><h2>아키텍처 개요</h2></summary>

Claude Code는 모든 프로덕션 코딩 에이전트가 마주해야 하는 **네 가지 설계 질문**에 답한다:

| 질문 | Claude Code의 답 |
|:---------|:---------------------|
| 추론은 어디에 두는가? | 모델은 추론하고, 하네스(harness)는 강제한다. AI 약 1.6%, 인프라 98.4%. |
| 실행 엔진은 몇 개인가? | 모든 인터페이스(CLI, SDK, IDE)에 대해 단일 `queryLoop`. |
| 기본 안전 자세는? | 거부 우선 (deny-first): deny > ask > allow. 가장 엄격한 규칙이 우선한다. |
| 자원의 결정적 제약은? | 약 200K(구형 모델) / 1M(Claude 4.6 시리즈) 컨텍스트 윈도우. 모델 호출 전마다 5단계 컴팩션. |

시스템은 **5개의 아키텍처 레이어**에 걸친 **7개의 컴포넌트**(사용자 → 인터페이스 → 에이전트 루프 → 권한 시스템 → 도구 → 상태 & 영속성 → 실행 환경)로 분해된다.

<p align="center">
  <img src="./assets/layered_architecture.png" width="100%" alt="5-layer subsystem decomposition">
</p>

> [!NOTE]
> 7개의 안전 레이어, 9단계 턴 파이프라인, 5단계 컴팩션 등 아키텍처 전반에 대한 심층 분석은 **[docs/architecture_KR.md](./docs/architecture_KR.md)** 를 참조하세요.

<p align="right"><a href="#dive-into-claude-code-the-design-space-of-todays-ai-agent-system">↑ Back to top</a></p>

</details>

---

<details open>
<summary><h2>가치와 설계 원칙</h2></summary>

이 아키텍처는 **5가지 인간 가치**에서 시작하여 **13가지 설계 원칙**을 거쳐 구현으로 이어진다:

| 가치 | 핵심 아이디어 |
|:------|:----------|
| **인간의 결정 권한** | 인간은 주체 계층 구조(principal hierarchy)를 통해 통제권을 유지한다. 93%에 달하는 프롬프트 승인율이 승인 피로(approval fatigue)를 드러냈을 때, 대응책은 더 많은 경고가 아닌 경계의 재구성이었다. |
| **안전, 보안, 프라이버시** | 인간의 경계심이 느슨해질 때조차 시스템이 보호한다. 7개의 독립적인 안전 레이어. |
| **신뢰성 있는 실행** | 의도된 일을 수행한다. 수집-실행-검증 루프. 우아한 복구. |
| **역량 증폭** | "제품이 아니라 Unix 유틸리티." 98.4%는 모델을 가능케 하는 결정론적 인프라. |
| **컨텍스트 적응성** | CLAUDE.md 계층, 점진적 확장성, 시간이 흐르며 진화하는 신뢰 경로(trust trajectory). |

<details>
<summary><b>13가지 설계 원칙</b></summary>

| 원칙 | 설계 질문 |
|:----------|:----------------|
| 거부 우선 + 인간 에스컬레이션 | 인식되지 않은 액션은 허용/차단/에스컬레이션 중 어느 것이어야 하는가? |
| 점진적 신뢰(graduated trust) 스펙트럼 | 권한을 고정 레벨로 둘 것인가, 시간에 따라 사용자가 통과하는 스펙트럼으로 둘 것인가? |
| 심층 방어(defense in depth) | 단일 안전 경계인가, 여러 겹의 중첩된 경계인가? |
| 외부화된 프로그래머블 정책 | 정책을 하드코딩할 것인가, 라이프사이클 훅과 함께 외부 설정으로 둘 것인가? |
| 희소 자원으로서의 컨텍스트 | 단일 패스 잘라내기(truncation)인가, 단계적 파이프라인인가? |
| 추가 전용(append-only) 영속 상태 | 가변 상태인가, 스냅샷인가, 추가 전용 로그인가? |
| 최소 스캐폴딩(scaffolding), 최대 하네스 | 스캐폴딩에 투자할 것인가, 운영 인프라에 투자할 것인가? |
| 규칙보다 가치 | 경직된 절차인가, 결정론적 가드레일과 함께 작동하는 맥락적 판단인가? |
| 조합 가능한 다중 메커니즘 확장성 | 단일 API인가, 비용이 다른 계층화된 메커니즘인가? |
| 가역성(reversibility) 가중 위험 평가 | 모든 작업에 동일한 감독을 적용할 것인가, 가역적 작업에는 더 가벼운 감독을 적용할 것인가? |
| 투명한 파일 기반 설정과 메모리 | 불투명 DB, 임베딩인가, 사용자가 볼 수 있는 파일인가? |
| 격리된 서브에이전트 경계 | 컨텍스트/권한 공유인가, 격리인가? |
| 우아한 복구와 회복력 | 즉시 실패할 것인가, 조용히 복구할 것인가? |

</details>

논문은 또한 **여섯 번째 평가 렌즈** -- 장기적 역량 보존 -- 를 적용하며, AI 보조 환경의 개발자가 이해도 시험에서 17% 더 낮은 점수를 받았다는 증거를 인용한다.

<p align="right"><a href="#dive-into-claude-code-the-design-space-of-todays-ai-agent-system">↑ Back to top</a></p>

</details>

---

<details>
<summary><h2>에이전트 쿼리 루프</h2></summary>

<p align="center">
  <img src="./assets/iteration.png" width="60%" alt="Runtime turn flow">
</p>

핵심은 **ReAct 패턴 기반의 while 루프**이다: 컨텍스트 조립 → 모델 호출 → 도구 디스패치 → 권한 확인 → 실행 → 반복. 스트리밍 이벤트를 yield하는 `AsyncGenerator`로 구현된다.

> **[번역 주석]** ReAct 패턴은 LLM이 추론(Reasoning)과 행동(Acting)을 번갈아 수행하며 문제를 풀어가는 방식을 가리킵니다.

**모델 호출 전마다**, 다섯 개의 컴팩션 셰이퍼가 비용이 적은 순서대로 차례로 실행된다: 예산 축소 → 스닙(Snip) → 마이크로컴팩트(Microcompact) → 컨텍스트 콜랩스(Context Collapse) → 오토컴팩트(Auto-Compact).

**턴당 9단계 파이프라인:** 설정 해석 → 상태 초기화 → 컨텍스트 조립 → 5개의 모델 전 셰이퍼 → 모델 호출 → 도구 디스패치 → 권한 게이트 → 도구 실행 → 중지 조건

**두 가지 실행 경로:**
- `StreamingToolExecutor` -- 도구가 스트리밍되는 즉시 실행을 시작 (지연 시간 최적화)
- 폴백 `runTools` -- 도구를 동시 실행 가능(concurrent-safe) / 배타적(exclusive)으로 분류

**복구:** 최대 출력 토큰 에스컬레이션 (3회 재시도), 반응적 컴팩션 (턴당 1회), 프롬프트 길이 초과 처리, 스트리밍 폴백, 폴백 모델

**5가지 중지 조건:** 도구 미사용, 최대 턴 수 도달, 컨텍스트 오버플로, 훅 개입, 명시적 중단

<p align="right"><a href="#dive-into-claude-code-the-design-space-of-todays-ai-agent-system">↑ Back to top</a></p>

</details>

---

<details open>
<summary><h2>안전성과 권한</h2></summary>

<p align="center">
  <img src="./assets/permission.png" width="75%" alt="Permission gate">
</p>

**7가지 권한 모드** 가 점진적 신뢰 스펙트럼을 형성한다: `plan` → `default` → `acceptEdits` → `auto` (ML 분류기) → `dontAsk` → `bypassPermissions` (+ 내부용 `bubble`).

**거부 우선(deny-first)**: 광범위한 deny가 *항상* 좁은 allow를 덮어쓴다. 도구 사전 필터링부터 셸 샌드박싱, 훅 인터셉션까지 이어지는 **7개의 독립적인 안전 레이어**. 권한은 **재개(resume) 시 결코 복원되지 않는다** -- 신뢰는 세션마다 다시 수립된다.

> [!WARNING]
> **공유 실패 모드:** 심층 방어는 레이어들이 제약을 공유할 때 약화된다. 서브커맨드별 파싱은 이벤트 루프 기아(event-loop starvation)를 유발한다 -- 50개를 초과하는 서브커맨드를 가진 명령은 REPL이 멈추는 것을 막기 위해 보안 분석 자체를 통째로 우회한다.

<details>
<summary><b>더 자세히: 인가 파이프라인, 자동 모드 분류기, CVE</b></summary>

**인가 파이프라인:** 사전 필터링(거부된 도구 제거) → PreToolUse 훅 → 거부 우선 규칙 평가 → 권한 핸들러 (4갈래: 코디네이터, 스웜 워커, 추측 분류기, 인터랙티브)

**자동 모드 분류기** (`yoloClassifier.ts`): 내부/외부 권한 템플릿을 사용하는 별도의 LLM 호출. 두 단계 구성: 빠른 필터 + chain-of-thought.

**사전 신뢰 실행 윈도우:** 패치된 2건의 CVE가 동일한 근본 원인을 공유한다 -- 훅과 MCP 서버가 신뢰 다이얼로그가 표시되기 *전* 초기화 단계에서 실행되어, 거부 우선 파이프라인 바깥에서 구조적으로 특권을 가진 공격 윈도우가 생긴다.

</details>

<p align="right"><a href="#dive-into-claude-code-the-design-space-of-todays-ai-agent-system">↑ Back to top</a></p>

</details>

---

<details>
<summary><h2>확장성</h2></summary>

<p align="center">
  <img src="./assets/extensibility.png" width="85%" alt="Three injection points: assemble, model, execute">
</p>

**컨텍스트 비용에 따라 점진적으로 분포한 4가지 메커니즘:** 훅 (영) → Skills (낮음) → Plugins (중간) → MCP (높음). 에이전트 루프 내 세 곳의 주입 지점: **assemble()** (모델이 보는 것), **model()** (모델이 도달할 수 있는 것), **execute()** (액션이 실행될지/어떻게 실행될지).

**도구 풀(tool pool) 조립** (5단계): 기본 열거(최대 54개 도구) → 모드 필터링 → 거부 사전 필터링 → MCP 통합 → 중복 제거

**27개의 훅 이벤트** 가 5개의 카테고리에 걸쳐 4가지 실행 유형(셸, LLM 평가, 웹훅, 서브에이전트 검증자)으로 제공된다

**플러그인 매니페스트**는 10가지 컴포넌트 유형을 받는다: commands, agents, skills, hooks, MCP 서버, LSP 서버, 출력 스타일, 채널, 설정, 사용자 설정

**Skills:** 15개 이상의 YAML frontmatter 필드를 갖는 SKILL.md. 핵심 차이 -- SkillTool은 현재 컨텍스트에 주입되는 반면, AgentTool은 격리된 컨텍스트를 새로 띄운다.

<p align="right"><a href="#dive-into-claude-code-the-design-space-of-todays-ai-agent-system">↑ Back to top</a></p>

</details>

---

<details open>
<summary><h2>컨텍스트와 메모리</h2></summary>

<p align="center">
  <img src="./assets/context.png" width="95%" alt="Context construction">
</p>

**9개의 정렬된 소스**가 컨텍스트 윈도우를 구성한다. CLAUDE.md 지시문은 시스템 프롬프트(결정론적)가 아니라 **사용자 컨텍스트**(확률적 준수)로 전달된다. 메모리는 **파일 기반**(벡터 DB 없음)이며 -- 완전히 검사 가능하고, 편집 가능하며, 버전 관리가 가능하다.

**4단계 CLAUDE.md 계층:** 관리형(`/etc/`) → 사용자(`~/.claude/`) → 프로젝트(`CLAUDE.md`, `.claude/rules/`) → 로컬(`CLAUDE.local.md`, gitignore 처리)

**5단계 컴팩션** (점진적 지연 감쇠): 예산 축소 → 스닙(Snip) → 마이크로컴팩트(Microcompact) → 컨텍스트 콜랩스(Context Collapse) (읽기 시점 프로젝션, 비파괴적) → 오토컴팩트(Auto-Compact) (모델을 통한 전체 요약, 최후의 수단)

**메모리 검색:** 메모리 파일 헤더에 대한 LLM 기반 스캔으로 관련 파일을 최대 5개까지 선택. 임베딩 없음, 벡터 유사도 없음.

<p align="right"><a href="#dive-into-claude-code-the-design-space-of-todays-ai-agent-system">↑ Back to top</a></p>

</details>

---

<details>
<summary><h2>서브에이전트 위임</h2></summary>

<p align="center">
  <img src="./assets/subagent.png" width="90%" alt="Subagent architecture">
</p>

**6가지 내장 유형** (Explore, Plan, General-purpose, Guide, Verification, Statusline)에 더해 `.claude/agents/*.md`로 정의하는 커스텀 에이전트. **사이드체인(sidechain) 트랜스크립트**: 부모로 돌아오는 것은 요약뿐이다 (부모의 컨텍스트는 서브에이전트의 장황함으로부터 *보호된다*). 세 가지 격리 모드: 워크트리(worktree), 원격(remote), 인프로세스. 조정은 POSIX `flock()` 으로 이루어진다.

**SkillTool vs AgentTool:** SkillTool은 현재 컨텍스트에 주입된다 (저렴함). AgentTool은 격리된 컨텍스트를 새로 띄운다 (비싸지만 컨텍스트 폭발을 막는다).

**권한 오버라이드:** 서브에이전트의 `permissionMode`가 적용된다. 단, 부모가 `bypassPermissions`/`acceptEdits`/`auto` 모드일 때는 예외다 (사용자의 명시적 결정이 항상 우선한다).

**커스텀 에이전트:** YAML frontmatter는 tools, disallowedTools, model, effort, permissionMode, mcpServers, hooks, maxTurns, skills, 메모리 스코프, 백그라운드 플래그, 격리 모드를 지원한다.

<p align="right"><a href="#dive-into-claude-code-the-design-space-of-todays-ai-agent-system">↑ Back to top</a></p>

</details>

---

<details>
<summary><h2>세션 영속성</h2></summary>

<p align="center">
  <img src="./assets/session_compact.png" width="75%" alt="Session persistence and context compaction">
</p>

세 가지 채널: 추가 전용(append-only) JSONL 트랜스크립트, 전역 프롬프트 히스토리, 서브에이전트 사이드체인. **권한은 재개 시 결코 복원되지 않는다** -- 신뢰는 세션마다 다시 수립된다. 설계는 **쿼리 능력보다 감사 가능성(auditability)** 을 우선한다.

**체인 패칭(chain patching):** 컴팩트 경계는 `headUuid`/`anchorUuid`/`tailUuid`를 기록한다. 세션 로더는 읽기 시점에 메시지 체인을 패치한다. 디스크 상의 어떤 것도 파괴적으로 편집되지 않는다.

**체크포인트:** `--rewind-files`를 위한 파일 히스토리 체크포인트는 `~/.claude/file-history/<sessionId>/`에 저장된다.

<p align="right"><a href="#dive-into-claude-code-the-design-space-of-todays-ai-agent-system">↑ Back to top</a></p>

</details>

---

## AI 에이전트 직접 만들기: 디자인 가이드

> 코딩 튜토리얼이 아니다. 아키텍처 분석에서 도출된, 반드시 내려야 할 **설계 결정**들에 대한 가이드이다.

모든 프로덕션 에이전트가 통과해야 할 결정들:

| 결정 | 질문 | 핵심 통찰 |
|:---------|:-------------|:------------|
| [**추론의 위치**](./docs/build-your-own-agent_KR.md#decision-1-where-does-reasoning-live) | 모델과 하네스에 각각 얼마만큼의 로직을 둘 것인가? | 모델 역량이 수렴함에 따라 하네스가 차별화 요소가 된다. |
| [**안전 자세**](./docs/build-your-own-agent_KR.md#decision-2-what-is-your-safety-posture) | 유해한 행동을 어떻게 막을 것인가? | 레이어들이 실패 모드를 공유하면 심층 방어는 무너진다. |
| [**컨텍스트 관리**](./docs/build-your-own-agent_KR.md#decision-3-how-do-you-manage-context) | 모델이 보는 것은 무엇인가? | 첫날부터 컨텍스트 희소성을 전제로 설계하라. 단계적 > 단일 패스. |
| [**확장성**](./docs/build-your-own-agent_KR.md#decision-4-how-do-you-handle-extensibility) | 확장 기능은 어떻게 끼워 넣는가? | 모든 확장이 컨텍스트 토큰을 소비할 필요는 없다. |
| [**서브에이전트 아키텍처**](./docs/build-your-own-agent_KR.md#decision-5-how-do-subagents-work) | 컨텍스트를 공유할 것인가, 격리할 것인가? | plan 모드의 에이전트 팀은 약 7배의 토큰을 소비한다. 서브에이전트가 요약만 반환하는 구조가 컨텍스트 폭발을 막는다. |
| [**세션 영속성**](./docs/build-your-own-agent_KR.md#decision-6-how-do-sessions-persist) | 무엇이 이어져야 하는가? | 재개 시 권한은 절대로 복원하지 말라. 쿼리 능력보다 감사 가능성을 우선하라. |

**전체 가이드 읽기: [docs/build-your-own-agent_KR.md](./docs/build-your-own-agent_KR.md)**

<p align="right"><a href="#dive-into-claude-code-the-design-space-of-todays-ai-agent-system">↑ Back to top</a></p>

---

## 커뮤니티 프로젝트 & 연구

Claude Code 아키텍처를 둘러싼 저장소, 재구현, 학술 논문을 큐레이션한 지도.

### Anthropic 공식 자료

논문 전반에서 인용된 1차 출처 — Anthropic 자체의 엔지니어링/연구 발행물 및 제품 문서.

#### 연구 & 엔지니어링 블로그

| 글 | 주제 |
|:--------|:------|
| [Building Effective Agents](https://www.anthropic.com/research/building-effective-agents) | 기초: 무거운 프레임워크보다 단순하고 조합 가능한 패턴. |
| [Effective Context Engineering for AI Agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) | 컨텍스트 큐레이션과 토큰 예산 관리. |
| [Harness Design for Long-Running Application Development](https://anthropic.com/engineering/harness-design-long-running-apps) | 자율 풀스택 개발을 위한 하네스 아키텍처와 멀티 에이전트 패턴. |
| [Claude Code Auto Mode: A Safer Way to Skip Permissions](https://www.anthropic.com/engineering/claude-code-auto-mode) | ML 분류기 기반 승인 자동화. 93% 승인율 데이터의 출처. |
| [Beyond Permission Prompts: Making Claude Code More Secure and Autonomous](https://www.anthropic.com/engineering/claude-code-sandboxing) | 샌드박스 기반 보안. 권한 프롬프트 84% 감소. |
| [Measuring AI Agent Autonomy in Practice](https://anthropic.com/research/measuring-agent-autonomy) | 종단 사용성 분석: 자동 승인 비율은 경험에 따라 약 20%에서 40% 이상으로 증가. |
| [Our Framework for Developing Safe and Trustworthy Agents](https://www.anthropic.com/news/our-framework-for-developing-safe-and-trustworthy-agents) | 책임 있는 에이전트 배포를 위한 거버넌스 프레임워크. |
| [Scaling Managed Agents: Decoupling the Brain from the Hands](https://www.anthropic.com/engineering/managed-agents) | 추론, 실행, 세션을 분리하는 호스팅 서비스 아키텍처. |

#### 제품 문서

| 문서 | 주제 |
|:---------|:------|
| [How Claude Code Works](https://code.claude.com/docs/en/how-claude-code-works) | 에이전트 루프, 도구, 터미널 자동화에 대한 공식 개요. |
| [Permissions](https://code.claude.com/docs/en/permissions) | 계층화된 권한 시스템, 모드, 세분화된 규칙. |
| [Hooks](https://code.claude.com/docs/en/hooks) | 27개 이벤트 훅 레퍼런스, 실행 모델, 라이프사이클 이벤트. |
| [Memory](https://code.claude.com/docs/en/memory) | CLAUDE.md 계층, 자동 메모리, 학습된 선호도. |
| [Sub-agents](https://code.claude.com/docs/en/sub-agents) | 격리된 전문 어시스턴트, 커스텀 프롬프트, 도구 접근. |

### 아키텍처 분석

Claude Code 내부 설계에 대한 심층 분석.

| 저장소 | 설명 |
|:-----------|:------------|
| [**ComeOnOliver/claude-code-analysis**](https://github.com/ComeOnOliver/claude-code-analysis) | 종합적인 리버스 엔지니어링: 소스 트리 구조, 모듈 경계, 도구 인벤토리, 아키텍처 패턴. |
| [**alejandrobalderas/claude-code-from-source**](https://github.com/alejandrobalderas/claude-code-from-source) | 18장 분량의 기술서 (약 400페이지). 모든 의사 코드는 자체 작성, 독점 소스 미포함. |
| [**liuup/claude-code-analysis**](https://github.com/liuup/claude-code-analysis) | 중국어 심층 분석 — 시작 흐름, 메인 쿼리 루프, MCP 통합, 멀티 에이전트 아키텍처. |
| [**sanbuphy/claude-code-source-code**](https://github.com/sanbuphy/claude-code-source-code) | 4개 언어(EN/JA/KO/ZH) 분석 — 텔레메트리, 코드네임, KAIROS, 미공개 도구 등 다중 도메인 보고서. |
| [**cablate/claude-code-research**](https://github.com/cablate/claude-code-research) | 내부 구조, Agent SDK, 관련 도구에 대한 독립 연구. |
| [**Yuyz0112/claude-code-reverse**](https://github.com/Yuyz0112/claude-code-reverse) | Claude Code의 LLM 상호작용 시각화 — 프롬프트, 도구 호출, 컴팩션을 추적하는 로그 파서와 시각화 도구. |

### 오픈소스 재구현

클린룸 재작성 및 빌드 가능한 연구용 포크.

| 저장소 | 설명 |
|:-----------|:------------|
| [**chauncygu/collection-claude-code-source-code**](https://github.com/chauncygu/collection-claude-code-source-code) | 커뮤니티의 Claude Code 소스 결과물을 모은 메타 컬렉션 -- claw-code(Rust 포팅), nano-claude-code(Python), 추출된 원본 소스 아카이브를 포함. |
| [**777genius/claude-code-working**](https://github.com/777genius/claude-code-working) | 동작하는 리버스 엔지니어링 CLI. Bun으로 실행 가능, 450개 이상의 청크 파일, 31개의 피처 플래그가 폴리필되어 있다. |
| [**T-Lab-CUHKSZ/claude-code**](https://github.com/T-Lab-CUHKSZ/claude-code) | 홍콩중문대(선전) 빌드 가능 연구용 포크 — 원시 TypeScript 스냅샷에서 빌드 시스템을 재구성. |
| [**ruvnet/open-claude-code**](https://github.com/ruvnet/open-claude-code) | 매일 자동 디컴파일하여 재빌드 — 903개 이상의 테스트, 25개 도구, 4개의 MCP 트랜스포트, 6개의 권한 모드. |
| [**Enderfga/openclaw-claude-code**](https://github.com/Enderfga/openclaw-claude-code) | OpenClaw 플러그인 — Claude/Codex/Gemini/Cursor에 대한 통합 ISession 인터페이스. 멀티 에이전트 위원회. |
| [**memaxo/claude_code_re**](https://github.com/memaxo/claude_code_re) | 미니파이된 번들로부터 리버스 엔지니어링 — 공개 배포된 cli.js 파일의 난독화 해제. |
| [**agentforce314/clawcodex**](https://github.com/agentforce314/clawcodex) | 다중 LLM 프로바이더를 지원하는 Python 재구축. |

### 가이드 & 학습 자료

튜토리얼과 실습 학습 경로.

| 저장소 | 설명 |
|:-----------|:------------|
| [**shareAI-lab/learn-claude-code**](https://github.com/shareAI-lab/learn-claude-code) | "Bash is all you need" — 19장으로 구성된 0→1 강의. 실행 가능한 Python 에이전트, 웹 플랫폼 포함. ZH/EN/JA. |
| [**FlorianBruniaux/claude-code-ultimate-guide**](https://github.com/FlorianBruniaux/claude-code-ultimate-guide) | 입문자부터 파워 유저까지 아우르는 가이드. 프로덕션 레벨 템플릿, 에이전트 워크플로우 가이드, 치트시트 포함. |
| [**affaan-m/everything-claude-code**](https://github.com/affaan-m/everything-claude-code) | 에이전트 하네스 최적화 — skills, instincts, memory, security, research-first development. 50K+ stars. |

### 블로그 글 & 기술 아티클

| 글 | 가치 있는 이유 |
|:--------|:----------------------|
| [Marco Kotrotsos — "Claude Code Internals" (15-part series)](https://kotrotsos.medium.com/claude-code-internals-part-1-high-level-architecture-9881c68c799f) | 유출(leak) 이전 분석 중 가장 체계적인 시리즈. 아키텍처, 에이전트 루프, 권한, 서브에이전트, MCP, 텔레메트리. |
| [Alex Kim — "The Claude Code Source Leak"](https://alex000kim.com/posts/2026-03-31-claude-code-source-leak/) | 안티 디스틸레이션(anti-distillation) 메커니즘, 좌절 감지, Undercover Mode, 하루 약 25만 회의 낭비된 API 호출. |
| [Haseeb Qureshi — Cross-agent architecture comparison](https://gist.github.com/Haseeb-Qureshi/2213cc0487ea71d62572a645d7582518) | Claude Code vs Codex vs Cline vs OpenCode — 아키텍처 수준의 비교. |
| [George Sung — "Tracing Claude Code's LLM Traffic"](https://medium.com/@georgesung/tracing-claude-codes-llm-traffic-agentic-loop-sub-agents-tool-use-prompts-7796941806f5) | 완전한 시스템 프롬프트와 전체 API 로그. 듀얼 모델 사용(Opus + Haiku) 발견. |
| [Agiflow — "Reverse Engineering Prompt Augmentation"](https://agiflow.io/blog/claude-code-internals-reverse-engineering-prompt-augmentation/) | 실제 네트워크 트레이스로 뒷받침되는 5가지 프롬프트 증강 메커니즘. |
| [Engineer's Codex — "Diving into the Source Code Leak"](https://read.engineerscodex.com/p/diving-into-claude-codes-source-code) | 모듈식 시스템 프롬프트, 약 40개의 도구, 거대한 쿼리/도구 서브시스템, 안티 디스틸레이션. |
| [MindStudio — "Three-Layer Memory Architecture"](https://www.mindstudio.ai/blog/claude-code-source-leak-memory-architecture) | 인-컨텍스트 메모리, MEMORY.md 포인터 인덱스, CLAUDE.md 정적 설정. 메모리에 관한 단일 자료로는 최고. |
| [WaveSpeed — "Claude Code Architecture: Leaked Source Deep Dive"](https://wavespeed.ai/blog/posts/claude-code-architecture-leaked-source-deep-dive/) | 512K 라인 규모의 TS 소스 심층 분석. 컨텍스트 압축과 안티 디스틸레이션. |
| [Zain Hasan — "Inside Claude Code: An Architecture Deep Dive"](https://zainhas.github.io/blog/2026/inside-claude-code-architecture/) | 계층화된 아키텍처, 5가지 진입 모드, 멀티 에이전트 워크스루. |

### 관련 학술 논문

| 논문 | 게재처 | 관련성 |
|:------|:------|:----------|
| [Decoding the Configuration of AI Coding Agents](https://arxiv.org/abs/2511.09268) | arXiv | Claude Code 설정 파일 328개에 대한 실증 연구 — 소프트웨어 공학적 관심사와 동시 출현 패턴. |
| [On the Use of Agentic Coding Manifests](https://arxiv.org/abs/2509.14744) | arXiv | 242개 저장소의 CLAUDE.md 파일 253개 분석 — 운영 명령에서의 구조적 패턴. |
| [Context Engineering for Multi-Agent Code Assistants](https://arxiv.org/abs/2508.08322) | arXiv | 코드 생성을 위해 여러 LLM을 결합하는 멀티 에이전트 워크플로우. |
| [OpenHands: An Open Platform for AI Software Developers](https://arxiv.org/abs/2407.16741) | ICLR 2025 | 오픈소스 AI 코딩 에이전트의 주요 학술적 레퍼런스. |
| [SWE-Agent: Agent-Computer Interfaces](https://arxiv.org/abs/2405.15793) | NeurIPS 2024 | 커스텀 agent-computer 인터페이스를 갖춘 Docker 기반 코딩 에이전트. |

### 이 논문의 차별점

> 위의 프로젝트들이 **엔지니어링 차원의 리버스 엔지니어링** 또는 **실용적 재구현**에 초점을 두는 반면, 이 논문은 **가치 → 원칙 → 구현**으로 이어지는 체계적 분석 프레임워크를 제공한다 — 5가지 인간 가치를 13가지 설계 원칙을 거쳐 구체적인 소스 레벨 선택까지 추적하고, OpenClaw 비교를 통해 모듈화된 기능이 아니라 **교차 관심사(cross-cutting) 통합 메커니즘**이 엔지니어링 복잡도의 진정한 자리임을 드러낸다.

**더 많은 자료가 포함된 전체 큐레이션 목록 보기: [docs/related-resources_KR.md](./docs/related-resources_KR.md)**

<p align="right"><a href="#dive-into-claude-code-the-design-space-of-todays-ai-agent-system">↑ Back to top</a></p>

---

## 그 외 주목할 만한 AI 에이전트 프로젝트

Claude Code 생태계 바깥에서 최근(2025–2026) 출시된 오픈소스 AI 에이전트 프로젝트들.

| 저장소 | 출시 시점 | 초점 |
|:-----------|:-------|:------|
| [**openclaw/openclaw**](https://github.com/openclaw/openclaw) | 2026년 1월 | 메시징 플랫폼 전반에서 동작하는 로컬 우선 개인 AI 어시스턴트. |
| [**sst/opencode**](https://github.com/sst/opencode) | 2025년 6월 | 프로바이더 비종속 터미널 코딩 에이전트. |
| [**NousResearch/hermes-agent**](https://github.com/nousresearch/hermes-agent) | 2026년 2월 | 세션 간 메모리를 갖춘 자기 개선형 개인 에이전트. |
| [**666ghj/MiroFish**](https://github.com/666ghj/MiroFish) | 2026년 3월 | 멀티 에이전트 군집 지능 시뮬레이션 엔진. |
| [**MemPalace/mempalace**](https://github.com/MemPalace/mempalace) | 2026년 | AI 에이전트를 위한 로컬 우선 메모리 시스템. |
| [**multica-ai/multica**](https://github.com/multica-ai/multica) | 2026년 | 작업 할당과 스킬 누적을 위한 매니지드 에이전트 플랫폼. |
| [**badlogic/pi-mono**](https://github.com/badlogic/pi-mono) | 2025년 8월 | 모노레포 코딩 에이전트 툴킷 — 통합 LLM API(OpenAI/Anthropic/Google), TUI + 웹 UI, cwd별 세션 영속성. |
| [**coleam00/Archon**](https://github.com/coleam00/Archon) | 2025년 2월 | 결정론적 하네스 — 실행 감사 추적이 포함된 YAML 정의 워크플로우. 모델 주도형 에이전트 루프와 대비된다. |
| [**HKUDS/nanobot**](https://github.com/HKUDS/nanobot) | 2026년 2월 | 홍콩대 데이터사이언스(HKU-DS)의 초경량 개인 AI 에이전트. 무거운 하네스와 대비되는 미니멀리즘 설계. |
| [**HKUDS/OpenHarness**](https://github.com/HKUDS/OpenHarness) | 2026년 4월 | 내장 개인 에이전트(Ohmo)를 갖춘 오픈 에이전트 하네스. 하네스 아키텍처에 대한 학술적 레퍼런스 포인트. |
| [**openai/symphony**](https://github.com/openai/symphony) | 2026년 2월 | OpenAI의 격리된 자율 구현 실행을 위한 오케스트레이션 — 병렬 세션 설계 축. |
| [**karpathy/autoresearch**](https://github.com/karpathy/autoresearch) | 2026년 3월 | 단일 GPU에서 nanochat 학습 연구를 수행하는 Andrej Karpathy의 자율 AI 에이전트 루프. |
| [**HKUDS/CLI-Anything**](https://github.com/HKUDS/CLI-Anything) | 2026년 3월 | "모든 소프트웨어를 에이전트 네이티브로(Making ALL Software Agent-Native)" — 임의의 소프트웨어를 에이전트 호출 가능 도구로 감싸는 통합 CLI 표면. |
| [**Panniantong/Agent-Reach**](https://github.com/Panniantong/Agent-Reach) | 2026년 2월 | 유료 API 없이 에이전트가 Twitter, Reddit, YouTube, GitHub, Bilibili, 샤오홍슈에 읽기/검색 접근할 수 있게 하는 단일 CLI. MCP 및 Claude Code / Cursor 통합 동봉. |
| [**agentscope-ai/QwenPaw**](https://github.com/agentscope-ai/QwenPaw) | 2026년 2월 | AgentScope 팀의 개인 AI 어시스턴트 — 셀프 호스팅 또는 클라우드, 다중 채팅 앱 지원, 확장 가능한 능력. |
| [**cft0808/edict**](https://github.com/cft0808/edict) | 2026년 2월 | "三省六部制(삼성육부제)" — OpenClaw 기반 멀티 에이전트 오케스트레이션. 9개의 전문 에이전트, 실시간 대시보드, 모델 설정, 전체 감사 로그를 갖춤. |

> **[번역 주석]** "三省六部制(삼성육부제)"는 중국 수·당대에 정착된 중앙 행정 제도로, 기능별로 분화된 부처가 협업하는 구조를 가리킵니다. 여기서는 멀티 에이전트가 역할별로 분업하는 구조의 메타포로 사용되었습니다.

<p align="right"><a href="#dive-into-claude-code-the-design-space-of-todays-ai-agent-system">↑ Back to top</a></p>

---
[![Star History Chart](https://api.star-history.com/svg?repos=VILA-Lab/Dive-into-Claude-Code&type=Date)](https://www.star-history.com/#VILA-Lab/Dive-into-Claude-Code&Date)

## 인용

<!-- <details>
<summary>BibTeX</summary> -->

```bibtex
@article{diveclaudecode2026,
  title={Dive into Claude Code: The Design Space of Today's and Future AI Agent Systems},
  author={Jiacheng Liu, Xiaohan Zhao, Xinyi Shang, and Zhiqiang Shen},
  year={2026},
  eprint={2604.14228},
  archivePrefix={arXiv},
  primaryClass={cs.SE},
}
```

</details>


## 라이선스

이 저작물은 [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/) 라이선스에 따라 배포됩니다.
