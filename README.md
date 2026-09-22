# FLAMORIS AI

Home for FLAMORIS AI projects, agents, orchestration, model runtimes, memory, knowledge, and generative AI integrations.

FLAMORIS AI is the integration and architecture home for the AI-facing parts of the FLAMORIS ecosystem.
This repository intentionally starts as a small meta repository: it documents boundaries, shared direction, and the relationship between dedicated AI repositories without becoming a second runtime or source of truth.

## Repository map

```text
flamoris-ai
├── flamoris-ai-agent
│   └── persistent agent runtime:
│       conversations / memory / knowledge / prompts / tools
├── flamoris-intelligence-mcp
│   └── provider-neutral MCP intelligence gateway:
│       local & remote LLMs / coding agents / routing / execution
└── flamoris-generation-mcp
    └── provider-neutral generative-media gateway:
        image / video / music / voice + related media analysis
```

### [FLAMORIS AI Agent](https://github.com/flamoris-jp/flamoris-ai-agent)

**Status: repository created; runtime implementation is not initialized yet.**

Planned persistent AI agent runtime for FLAMORIS, with conversations, memory, knowledge, prompts, tools, and pluggable local or remote intelligence access.

Its intended responsibility is agent-facing state and behavior. It may call intelligence or generation services, but those services should not silently become a second owner of conversations, memory, or agent state.

### [FLAMORIS Intelligence MCP](https://github.com/flamoris-jp/flamoris-intelligence-mcp)

**Status: repository created; runtime implementation is not initialized yet.**

Planned MCP-native, provider-neutral gateway for language, reasoning, coding, and multi-agent intelligence.

Its intended responsibilities include exposing intelligence capabilities through MCP, model/provider routing, task coordination, scheduling, multi-agent workflows, and explicit access to shared memory or knowledge services without becoming the owner of Agent memory itself. Model/runtime-specific behavior belongs behind provider adapters rather than leaking into applications or agents.

The Intelligence MCP is the MCP-facing boundary for language, reasoning, coding, and related intelligence. Any multi-agent or task coordination here is bounded intelligence execution, not ownership of persistent Agent identity or memory. It is deliberately separate from generative-media and media-domain execution.

### [FLAMORIS Generation MCP](https://github.com/flamoris-jp/flamoris-generation-mcp)

**Status: active implementation. ComfyUI is the first provider.**

MCP-native gateway for generative media such as images, video, music, and voice, plus closely related media-domain analysis that shares the same workflow/job/asset lifecycle.

ComfyUI is the first provider, not the identity of the project. Provider-specific execution remains behind adapters while jobs, workflows, assets, and MCP-facing behavior stay provider-neutral where practical.

The Generation Hub foundation currently lives inside `flamoris-generation-mcp`. It is not a separate repository unless a future boundary becomes clear enough to justify one.

## How the pieces fit together

A typical future path is:

```text
FLAMORIS application / Studio
          │
          ▼
   flamoris-ai-agent
      │         │
      │         └──────────────► flamoris-generation-mcp
      │                           image / video / music / voice
      ▼
flamoris-intelligence-mcp
 local LLM / remote LLM / coding intelligence
```

These are boundaries, not mandatory layers.

- Applications may call the Intelligence MCP directly when no persistent Agent is needed.
- Applications may call Generation MCP directly for media generation.
- The Agent may coordinate both, but should not duplicate their provider/runtime responsibilities.
- Shared non-AI infrastructure such as logging and MCP foundations belongs in [FLAMORIS Commons](https://github.com/flamoris-jp/flamoris-commons), not here.
- Product-specific state and editing authority stay in the product repository that owns them.

## Repository policy

This repository follows the shared [FLAMORIS Repository Policy](https://github.com/flamoris-jp/flamoris-commons/blob/main/docs/repository-policy.md).

AI-specific boundaries and safety rules in this repository supplement that shared policy rather than replacing it.

## Design principles

1. **One authority per domain**
   - Conversations, memory, and agent state belong to the Agent.
   - Language/reasoning/coding intelligence execution and provider routing belong to the Intelligence MCP.
   - Generative-media and related media-analysis workflows, jobs, and assets belong to Generation MCP.
   - Product document state remains owned by the FLAMORIS application.

2. **Provider-neutral FLAMORIS boundaries**
   - Local and remote providers are implementation choices behind adapters.
   - A provider name should not become the public architecture of FLAMORIS.

3. **Local-first without local-only assumptions**
   - FLAMORIS may use local models on machines such as LIME as well as remote services.
   - Repositories must not contain machine-specific credentials, private network details, or model weights.

4. **No speculative mega-framework**
   - Split repositories when a boundary is real and useful.
   - Do not centralize code merely because it is AI-related.

5. **Explicit model and asset licensing**
   - Repository code licenses do not automatically apply to models, weights, datasets, generated media, prompts sourced from third parties, or provider-hosted assets.

## Philosophy

FLAMORIS is open-source software for creative work and AI-native production.

Use it however you like.

Commercial use is welcome and does not require permission.  
If you'd like, we'd be happy to hear what you used FLAMORIS for.  
This is completely optional.

FLAMORIS software is provided as-is.
We do not provide individual support or guaranteed assistance.

If you run into trouble, we encourage you to let your AI assistant read the repository, documentation, issues, and source code and help you solve it.

If FLAMORIS helps you or you find it interesting,
your support helps fund development and keeps the project growing. 🌱  
<sub>Mostly GPU bills.</sub>

## License

Code and documentation in this repository are licensed under the [Apache License 2.0](LICENSE), unless otherwise noted.

AI models, model weights, datasets, generated media, third-party prompts, provider-hosted assets, and other non-code material are not automatically covered by this repository's license. Their applicable licenses and usage terms must be checked and documented separately.

---

## 日本語

FLAMORIS AIは、FLAMORISのAI関連プロジェクトをまとめる入口です。

ここ自体を巨大なAIランタイムにするのではなく、AI Agent、推論・Coding系のIntelligence MCP、画像・動画・音楽・音声生成のGeneration MCPについて、役割と境界、共通方針を整理します。

### リポジトリ構成

- **[flamoris-ai-agent](https://github.com/flamoris-jp/flamoris-ai-agent)**  
  **現在: repository作成済み、runtime実装は未初期化。**  
  会話、Memory、Knowledge、Prompt、Toolを持つ永続的なAI Agent runtimeとして設計します。

- **[flamoris-intelligence-mcp](https://github.com/flamoris-jp/flamoris-intelligence-mcp)**  
  **現在: repository作成済み、runtime実装は未初期化。**  
  LLM、推論、Coding AgentなどをMCPから扱うprovider-neutralなintelligence gateway。model/provider routing、task coordination、scheduling、multi-agent workflowを扱い、必要なMemory/Knowledgeへ明示的なinterfaceでアクセスします。ただしAgent Memoryそのもののauthorityにはなりません。

- **[flamoris-generation-mcp](https://github.com/flamoris-jp/flamoris-generation-mcp)**  
  **現在: 実装進行中。最初のproviderはComfyUI。**  
  画像・動画・音楽・音声などの生成AIと、同じworkflow/job/asset lifecycleに乗る密接なmedia-domain analysisをMCPから扱う実行ゲートウェイ。ComfyUIは最初のproviderであり、プロジェクトそのものではありません。

Generation Hubの基盤は現在 `flamoris-generation-mcp` の内部に置きます。境界が明確になるまでは、Generation Hubという別リポジトリを増やしません。

### 境界の考え方

- Agentの会話やMemoryはAgentが持つ。
- 言語・推論・Coding系のprovider選択とintelligence実行はIntelligence MCPが持つ。multi-agent/task coordinationを行う場合も、永続Agent Memoryのauthorityにはならない。
- 生成mediaと密接なmedia-domain analysisのjob、workflow、assetはGeneration MCPが持つ。
- 2D、Cutwork、Kachincoなどの制作データは各アプリ自身がauthorityを持つ。
- LoggingやMCP共通基盤など、AI専用ではないものはFLAMORIS Commons側に置く。
- 「AIっぽいから全部ここへ」は禁止。AIにも物置部屋は作らない。🐈

### 方針

FLAMORISは、クリエイティブ制作とAIネイティブな制作環境のためのオープンソースソフトウェアです。

勝手に使ってください。  
改造しても、組み込んでも、面白いものや変なものを作ってもOKです。

商用作品や製品で使う場合も、許可は不要です。  
もしよければ「こんなのに使ったよ」と教えてもらえるとうれしいです。  
もちろん強制ではありません。

FLAMORISのソフトウェアは現状のまま提供されます。  
個別サポートや動作保証はありません。

困ったときは、README、ドキュメント、Issue、ソースコードをあなたのAIに読ませて、自己サポートしてもらってください。

このリポジトリのコードとドキュメントは、明記がない限りApache License 2.0です。

AIモデル、model weights、データセット、生成物、第三者由来のprompt、provider側のassetなどには、別のライセンスや利用条件が適用される場合があります。それぞれの条件を確認し、必要な場所に明記してください。

もし、あなたのお役に立てたり、面白いと思っていただけたなら、  
開発費用をご支援いただけるとうれしいです。  
FLAMORISは元気になって育ちます。🌱  
<sub>主にGPU代とか。</sub>
