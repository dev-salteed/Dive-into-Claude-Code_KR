[메인 README로 돌아가기](../README_KR.md)

# 관련 리소스: 커뮤니티 분석

> Claude Code의 아키텍처를 둘러싼 저장소, 재구현, 블로그 글, 학술 논문을 큐레이션한 지도.

---

## 아키텍처 분석

Claude Code 내부 설계에 대한 심층 분석.

| 저장소 | 설명 |
|:-----------|:------------|
| [**ComeOnOliver/claude-code-analysis**](https://github.com/ComeOnOliver/claude-code-analysis) | 종합적인 리버스 엔지니어링: 소스 트리 구조, 모듈 경계, 도구 인벤토리, 아키텍처 패턴. |
| [**alejandrobalderas/claude-code-from-source**](https://github.com/alejandrobalderas/claude-code-from-source) | 18장 분량의 기술 서적(약 400페이지). 모두 자체 의사 코드로 작성되어 독점 소스를 포함하지 않음. |
| [**liuup/claude-code-analysis**](https://github.com/liuup/claude-code-analysis) | 중국어 심층 분석 — 시작 흐름, 쿼리 메인 루프, MCP 통합, 멀티 에이전트 아키텍처. |
| [**sanbuphy/claude-code-source-code**](https://github.com/sanbuphy/claude-code-source-code) | 4개 언어(EN/JA/KO/ZH) 분석 — 10개 도메인, 75개 보고서. 텔레메트리, 코드네임, KAIROS, 미공개 도구 등을 다룸. |
| [**cablate/claude-code-research**](https://github.com/cablate/claude-code-research) | 내부 구조, Agent SDK, 관련 도구에 대한 독립 연구. |
| [**Yuyz0112/claude-code-reverse**](https://github.com/Yuyz0112/claude-code-reverse) | Claude Code의 LLM 상호작용 시각화 — 프롬프트, 도구 호출, 컴팩션을 추적하는 로그 파서 및 시각화 도구. |
| [**AgiFlow/claude-code-prompt-analysis**](https://github.com/AgiFlow/claude-code-prompt-analysis) | 5개 대화 세션에 걸친 API 요청/응답 로그. 재현 가능한 데이터를 갖춘 독자적 경험적 방법론. |

---

## 오픈소스 재구현

클린룸 재작성 및 빌드 가능한 연구용 포크.

| 저장소 | 설명 |
|:-----------|:------------|
| [**chauncygu/collection-claude-code-source-code**](https://github.com/chauncygu/collection-claude-code-source-code) | 메타 컬렉션 — claw-code(Rust, 30K+ stars), nano-claude-code(Python 약 5K 줄), 그리고 원본 소스 아카이브. |
| [**ultraworkers/claw-code**](https://github.com/ultraworkers/claw-code) | 클린룸 Rust 재구현. 9일 만에 179K stars 달성 — GitHub 역사상 100K stars에 가장 빠르게 도달한 저장소. 512K LoC TypeScript를 약 20K 줄로 축소. |
| [**777genius/claude-code-working**](https://github.com/777genius/claude-code-working) | 동작하는 리버스 엔지니어링 CLI. Bun으로 실행 가능, 450개 이상의 청크 파일, 30개의 기능 플래그가 폴리필됨. |
| [**T-Lab-CUHKSZ/claude-code**](https://github.com/T-Lab-CUHKSZ/claude-code) | 홍콩중문대학교 선전 캠퍼스의 빌드 가능한 연구용 포크 — 원본 TypeScript 스냅샷에서 빌드 시스템을 재구성. |
| [**ruvnet/open-claude-code**](https://github.com/ruvnet/open-claude-code) | 매일 자동 디컴파일 재빌드 — 903개 이상의 테스트, 25개 도구, 4개의 MCP 트랜스포트, 6가지 권한 모드. |
| [**Enderfga/openclaw-claude-code**](https://github.com/Enderfga/openclaw-claude-code) | OpenClaw 플러그인 — Claude/Codex/Gemini/Cursor를 통합하는 ISession 인터페이스. 멀티 에이전트 카운슬. |
| [**memaxo/claude_code_re**](https://github.com/memaxo/claude_code_re) | 미니파이된 번들로부터의 리버스 엔지니어링 — 공개 배포된 cli.js 파일 디오브퓨스케이션. |
| [**agentforce314/clawcodex**](https://github.com/agentforce314/clawcodex) | 멀티 프로바이더 LLM을 지원하는 Python 재구축. |

---

## 가이드 & 학습

튜토리얼과 실습 학습 경로.

| 저장소 | 설명 |
|:-----------|:------------|
| [**shareAI-lab/learn-claude-code**](https://github.com/shareAI-lab/learn-claude-code) | "Bash is all you need" (Bash만 있으면 충분하다) — 실행 가능한 Python 에이전트와 웹 플랫폼이 포함된 19장 분량의 0-to-1 코스. ZH/EN/JA. |
| [**FlorianBruniaux/claude-code-ultimate-guide**](https://github.com/FlorianBruniaux/claude-code-ultimate-guide) | 초보자부터 파워 유저까지를 위한 가이드. 프로덕션 수준 템플릿, 에이전트 워크플로우 가이드, 치트시트 제공. |
| [**affaan-m/everything-claude-code**](https://github.com/affaan-m/everything-claude-code) | 에이전트 하네스 최적화 — 스킬, 본능(instincts), 메모리, 보안, 리서치 우선 개발. 50K+ stars. |
| [**nblintao/awesome-claude-code-postleak-insights**](https://github.com/nblintao/awesome-claude-code-postleak-insights) | 가장 잘 큐레이션된 유출 이후 자료 모음. BUDDY, KAIROS, ULTRAPLAN, Undercover Mode, AutoDream 메모리 통합 등을 다룸. |
| [**hesreallyhim/awesome-claude-code**](https://github.com/hesreallyhim/awesome-claude-code) | 스킬, 훅, 슬래시 커맨드, 에이전트 오케스트레이터, 플러그인의 큐레이션 목록. |
| [**rohitg00/awesome-claude-code-toolkit**](https://github.com/rohitg00/awesome-claude-code-toolkit) | 135개 에이전트, 35개 스킬, 42개 커맨드, 176개 이상의 플러그인, 20개 훅, 15개 룰, 7개 템플릿, 14개 MCP 설정. |

---

## 블로그 글 & 기술 아티클

### 유출 이전 리버스 엔지니어링 (2025 — 2026년 초)

> **[번역 주석]** "pre-leak / post-leak"는 2026년 3월 Claude Code 소스 코드가 외부로 유출된 사건을 기준으로 한 시기 구분이다. 유출 이전 자료는 미니파이된 번들·트래픽 관찰에 의존했고, 유출 이후 자료는 원본 소스를 직접 분석할 수 있게 되었다.

| 글 | 가치 있는 점 |
|:--------|:----------------------|
| [Marco Kotrotsos — "Claude Code Internals" (15부작 시리즈)](https://kotrotsos.medium.com/claude-code-internals-part-1-high-level-architecture-9881c68c799f) | 유출 이전 분석 중 가장 체계적. 아키텍처, 에이전트 루프, 권한, 서브 에이전트, MCP, 텔레메트리. v2.0.76 기준. |
| [George Sung — "Tracing Claude Code's LLM Traffic"](https://medium.com/@georgesung/tracing-claude-codes-llm-traffic-agentic-loop-sub-agents-tool-use-prompts-7796941806f5) | 전체 시스템 프롬프트와 완전한 API 로그. 듀얼 모델 사용을 발견(메인 루프는 Opus, 메타데이터는 Haiku). |
| [Reid Barber — "Reverse Engineering Claude Code"](https://www.reidbarber.com/blog/reverse-engineering-claude-code) | 가장 이른 시기(2025년 중반)의 분석 중 하나. REPL 아키텍처와 도구 모음을 다룸. |
| [Kir Shatrov — "Reverse Engineering Claude Code"](https://kirshatrov.com/posts/claude-code-internals) | mitmproxy로 API 호출을 가로챔. 단일 실험 결과: LLM 시간 40초, 비용 $0.11. |
| [Sabrina Ramonov — "Reverse-Engineering Using Sub Agents"](https://www.sabrina.dev/p/reverse-engineering-claude-code-using) | 창의적인 방법론: 커스텀 서브 에이전트(File Splitter, Structure Analyzer)를 만들어 미니파이된 JS를 리버스 엔지니어링. |

### 유출 이후 분석 (2026년 3~4월)

| 글 | 가치 있는 점 |
|:--------|:----------------------|
| [Alex Kim — "The Claude Code Source Leak"](https://alex000kim.com/posts/2026-03-31-claude-code-source-leak/) | 유출 직후 가장 결정적인 분석. 안티 디스틸레이션, 사용자 좌절(frustration) 감지 정규식, Undercover Mode, 일일 약 25만 건의 낭비된 API 호출을 다룸. HN에서 화제. |
| [Haseeb Qureshi — "Inside the Claude Code source"](https://gist.github.com/Haseeb-Qureshi/d0dc36844c19d26303ce09b42e7188c1) | 핵심 모듈 분석. React/Ink UI, 4가지 컴팩션 전략, 동적 프롬프트 경계 시스템. |
| [Haseeb Qureshi — Cross-agent architecture comparison](https://gist.github.com/Haseeb-Qureshi/2213cc0487ea71d62572a645d7582518) | Claude Code vs Codex vs Cline vs OpenCode — 아키텍처 수준에서의 비교. |
| [Engineer's Codex — "Diving into the Source Code Leak"](https://read.engineerscodex.com/p/diving-into-claude-codes-source-code) | 모듈식 시스템 프롬프트, 약 40개 도구, 46K 줄 분량의 쿼리 엔진, 안티 디스틸레이션. 폭넓은 독자가 이해하기 쉬움. |

> **[번역 주석]** "anti-distillation"은 모델 증류(distillation)를 통한 무단 복제를 막기 위한 메커니즘이다. 출력 패턴을 분산시키거나 의도적으로 노이즈를 섞어 외부에서 학습 데이터로 활용하기 어렵게 만든다.

| [Han HELOIR YAN — "Nobody Analyzed Its Architecture"](https://medium.com/data-science-collective/everyone-analyzed-claude-codes-features-nobody-analyzed-its-architecture-1173470ab622) | "Nobody Analyzed Its Architecture" (아무도 그 아키텍처를 분석하지 않았다). "해자(moat)는 모델이 아니라 하네스다." 본 보고서의 논지와 결을 같이 함. |
| [Agiflow — "Reverse Engineering Prompt Augmentation"](https://agiflow.io/blog/claude-code-internals-reverse-engineering-prompt-augmentation/) | 실제 네트워크 트레이스로 뒷받침된 5가지 프롬프트 증강 메커니즘. Skills의 2단계 시맨틱 매칭을 보여줌. |

> **[번역 주석]** "moat"는 원래 성을 둘러싼 해자를 뜻하지만, 비즈니스에서는 경쟁자가 따라잡기 어려운 지속 가능한 경쟁 우위를 가리키는 비유로 쓰인다. 여기서는 Claude Code의 진짜 경쟁력이 모델 자체가 아니라 모델을 감싸는 하네스(harness, 실행 환경) 설계에 있다는 뜻이다.

### 특정 주제 심층 분석

| 주제 | 자료 |
|:------|:---------|
| **메모리 아키텍처** | [MindStudio — "Three-Layer Memory Architecture"](https://www.mindstudio.ai/blog/claude-code-source-leak-memory-architecture) — 컨텍스트 내 메모리, MEMORY.md 포인터 인덱스, CLAUDE.md 정적 설정. 메모리에 관한 단일 자료로는 최고. |
| **권한 시스템** | [Marco Kotrotsos — "Part 8: The Permission System"](https://kotrotsos.medium.com/claude-code-internals-part-8-the-permission-system-624bd7bb66b7) |
| **프롬프트 캐싱** | [ClaudeCodeCamp — "How Prompt Caching Actually Works"](https://www.claudecodecamp.com/p/how-prompt-caching-actually-works-in-claude-code) |
| **MCP 통합** | [Gigi Sayfan — "MCP Unleashed"](https://medium.com/@the.gigi/claude-code-deep-dive-mcp-unleashed-0c7692f9c2c2) |
| **압축 & 텔레메트리** | [WaveSpeedAI — "Architecture Deep Dive"](https://wavespeed.ai/blog/posts/claude-code-architecture-leaked-source-deep-dive/) — 3계층 압축, 사용자 좌절(frustration) 지표. |
| **Rust 재작성 분석** | [DEV Community — "Architecture via Rust Rewrite"](https://dev.to/brooks_wilson_36fbefbbae4/claude-code-architecture-explained-agent-loop-tool-system-and-permission-model-rust-rewrite-41b2) — 3계층 구조 안의 18개 도구. |
| **권한 설정** | [Vincent Qiao — "Permissions System Deep Dive"](https://blog.vincentqiao.com/en/posts/claude-code-settings-permissions/) |

---

## 관련 학술 논문

| 논문 | 발표처 | 관련성 |
|:------|:------|:----------|
| [Decoding the Configuration of AI Coding Agents](https://arxiv.org/abs/2511.09268) | arXiv | 328개 Claude Code 설정 파일에 대한 경험적 연구 — 소프트웨어 공학적 관심사와 동시 출현 패턴. |
| [On the Use of Agentic Coding Manifests](https://arxiv.org/abs/2509.14744) | arXiv | 242개 저장소의 CLAUDE.md 파일 253개 분석 — 운영 명령의 구조적 패턴. |
| [Context Engineering for Multi-Agent Code Assistants](https://arxiv.org/abs/2508.08322) | arXiv | 코드 생성을 위해 여러 LLM을 결합하는 멀티 에이전트 워크플로우. |
| [OpenHands: An Open Platform for AI Software Developers](https://arxiv.org/abs/2407.16741) | ICLR 2025 | 오픈소스 AI 코딩 에이전트의 주요 학술적 레퍼런스. |
| [SWE-Agent: Agent-Computer Interfaces](https://arxiv.org/abs/2405.15793) | NeurIPS 2024 | 커스텀 에이전트-컴퓨터 인터페이스를 갖춘 Docker 기반 코딩 에이전트. |
| [The OpenHands Software Agent SDK](https://arxiv.org/abs/2511.03690) | arXiv | 프로덕션용 에이전트를 위한 조합형(composable) SDK 토대. |
| [A Survey on Code Generation with LLM-based Agents](https://arxiv.org/abs/2508.00083) | arXiv | AI 코딩 에이전트 분야의 가장 우수한 서베이. |
| [AI Agent Systems: Architectures, Applications, and Evaluation](https://arxiv.org/html/2601.01743v1) | arXiv 2026 | 폭넓은 에이전트 시스템 분류 체계. |

---

## 이 논문이 다른 점

위 프로젝트들이 **엔지니어링 리버스 엔지니어링**이나 **실용적 재구현**에 집중하는 반면, 이 논문은 **가치 → 원칙 → 구현**이라는 체계적인 분석 프레임워크를 제시한다. 다섯 가지 인간적 가치를 13가지 설계 원칙을 거쳐 구체적인 소스 수준의 선택까지 추적하고, OpenClaw와의 비교를 통해 엔지니어링 복잡성의 진정한 핵심이 모듈화된 기능이 아니라 횡단적이고 통합적인 메커니즘에 있음을 드러낸다.

---

*여기에 추가될 만한 자료를 알고 있다면 이슈나 PR을 열어 알려 주세요.*
