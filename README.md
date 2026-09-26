# FLAMORIS AI

Home and coordination point for the AI-facing parts of FLAMORIS.

This repository is intentionally small. It documents architecture boundaries, cross-repository dependency direction, and the integrated roadmap without becoming a second runtime or implementation authority.

See [the integrated roadmap](docs/ROADMAP.md) and [the top-level roadmap tracker](https://github.com/flamoris-jp/flamoris-ai/issues/4).

## Repository map

```text
flamoris-ai
├── flamoris-ai-agent
│   └── persistent Agent runtime + Agent MCP surface:
│       identity / conversations / memory / knowledge / personality / tools
├── flamoris-intelligence-mcp
│   └── provider-neutral raw intelligence gateway:
│       language / reasoning / coding / local & remote providers
└── flamoris-generation-mcp
    └── provider-neutral generative-media gateway:
        image / video / music / voice + related media analysis
```

Supporting ecosystem boundaries coordinated by the roadmap include:

- [flamoris-mcp-hub](https://github.com/flamoris-jp/flamoris-mcp-hub) — MCP aggregation/routing boundary
- [flamoris-gpu-node-manager](https://github.com/flamoris-jp/flamoris-gpu-node-manager) — provider-neutral local GPU runtime authority; LIME is one deployment
- [flamoris-studio](https://github.com/flamoris-jp/flamoris-studio) — human-facing AI workspace and application boundary

Current implementation status belongs in each repository's own README, Issues, and PRs rather than being duplicated here.

## Core boundary

The central dependency direction is:

```text
Agent → Intelligence MCP → model/provider
```

The Agent owns persistent FLAMORIS-aware state. Intelligence MCP owns raw language/reasoning/coding execution. Generation MCP owns media workflows/jobs/assets. GPU Node Manager owns local GPU runtime transitions without making a particular node name part of the architecture.

A caller should eventually be able to choose intentionally between:

```text
intelligence.*  raw LLM / reasoning / coding
agent.*         persistent FLAMORIS-aware Agent
generation.*    image / video / music / voice / media analysis
lime.*          explicit runtime/GPU operations
```

### FLAMORIS AI Agent

[flamoris-ai-agent](https://github.com/flamoris-jp/flamoris-ai-agent) owns persistent Agent behavior and Agent-facing state:

- identity and personality;
- conversations/messages;
- memory;
- knowledge context;
- prompts and Agent policy;
- later tools and durable orchestration state.

It may call Intelligence MCP and Generation MCP, but those services must not become second owners of Agent state.

The Agent's MCP surface should initially live inside `flamoris-ai-agent`. Do not create a separate Agent MCP repository unless a real deployment or ownership boundary appears.

### FLAMORIS Intelligence MCP

[flamoris-intelligence-mcp](https://github.com/flamoris-jp/flamoris-intelligence-mcp) is the provider-neutral boundary for raw language, reasoning, coding, and related bounded intelligence execution.

It may route to local or remote providers, but it does not own persistent conversations, Agent memory, personality, generated-media workflows, product documents, or LIME runtime switching.

### FLAMORIS Generation MCP

[flamoris-generation-mcp](https://github.com/flamoris-jp/flamoris-generation-mcp) owns generative-media and closely related media-analysis workflows, jobs, providers, and assets.

ComfyUI is a provider, not the identity of the project. The Generation Hub foundation remains inside `flamoris-generation-mcp` unless a future Issue establishes a genuinely independent boundary.

## How the pieces fit together

```text
ChatGPT / Studio / FLAMORIS clients
                │
                ▼
             MCP Hub
       ┌────────┼────────┬────────┐
       │        │        │        │
       ▼        ▼        ▼        ▼
 generation.* intelligence.* agent.* lime.*
       │        │        │        │
       ▼        │        ▼        ▼
Generation MCP  │   Agent runtime  GPU Node Manager
                │        │
                └───┬────┘
                    ▼
             Intelligence MCP
                    │
                    ▼
           local / remote providers
```

These are boundaries, not mandatory layers. Applications may call Generation or Intelligence directly when no persistent Agent is needed.

## Repository policy

This repository follows the shared [FLAMORIS Repository Policy](https://github.com/flamoris-jp/flamoris-commons/blob/main/docs/repository-policy.md).

AI-specific boundaries and safety rules in this repository supplement that shared policy rather than replacing it.

## Design principles

1. **One authority per domain**
   - Conversations, memory, knowledge context, personality, and Agent policy belong to the Agent.
   - Language/reasoning/coding provider execution belongs to Intelligence MCP.
   - Generative-media workflows/jobs/assets belong to Generation MCP.
   - Local GPU runtime transitions belong to GPU Node Manager.
   - Product document state remains owned by the FLAMORIS application.

2. **Provider-neutral FLAMORIS boundaries**
   - Local and remote providers are implementation choices behind adapters.
   - Provider names should not become the public architecture unless the behavior is intentionally provider-specific.

3. **Local-first without local-only assumptions**
   - FLAMORIS may use local models on machines such as LIME as well as remote services.
   - Repositories must not contain machine-specific credentials, private deployment topology, or model weights.

4. **No speculative mega-framework**
   - Split repositories only when a real boundary exists.
   - Do not centralize code merely because it is AI-related.

5. **Explicit model and asset licensing**
   - Repository code licenses do not automatically apply to model weights, datasets, generated media, third-party prompts, or provider-hosted assets.

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

FLAMORIS AIは、FLAMORISのAI関連プロジェクトをまとめる**管制塔**です。

ここ自体を巨大なAIランタイムにはせず、各リポジトリの責任範囲、依存方向、3並行の開発ロードマップを管理します。

現在の実装状況そのものは各リポジトリのREADME / Issue / PRを正とし、このREADMEには変化の速いステータスを重複して書きません。

詳しくは [統合ロードマップ](docs/ROADMAP.md) を参照してください。

### 主要な役割

- **[flamoris-ai-agent](https://github.com/flamoris-jp/flamoris-ai-agent)**  
  永続的なAgent。Identity、Conversation、Memory、Knowledge、Personality、Prompt/Policy、将来のToolを持ちます。Agent用MCP surfaceも当面このリポジトリ内に置きます。

- **[flamoris-intelligence-mcp](https://github.com/flamoris-jp/flamoris-intelligence-mcp)**  
  素のLLM、推論、Codingなどを扱うprovider-neutralなIntelligence gateway。AgentのMemoryやConversationのauthorityにはなりません。

- **[flamoris-generation-mcp](https://github.com/flamoris-jp/flamoris-generation-mcp)**  
  画像・動画・音楽・音声などのgenerationと、同じworkflow/job/asset lifecycleに属するmedia analysisを扱います。

- **[flamoris-mcp-hub](https://github.com/flamoris-jp/flamoris-mcp-hub)**  
  `generation.*` / `intelligence.*` / `agent.*` / `lime.*` をまとめる入口。workflow authorityにはしません。

- **[flamoris-gpu-node-manager](https://github.com/flamoris-jp/flamoris-gpu-node-manager)**  
  LIMEのruntime/GPU authority。systemdやGPU切替ロジックを他サービスへ複製しません。

中心となる依存方向は:

```text
Agent → Intelligence MCP → model/provider
```

です。

つまり、

```text
intelligence.* = 外部AIとして呼ぶ素の知能
agent.*        = FLAMORISを知る永続Agent
generation.*   = 生成工房
lime.*         = GPU/runtime機関室
```

という役割分担を守ります。

「AIっぽいから全部ここへ」は禁止です。AIにも物置部屋は作りません。🐈

### 方針

FLAMORISは、クリエイティブ制作とAIネイティブな制作環境のためのオープンソースソフトウェアです。

勝手に使ってください。  
改造しても、組み込んでも、面白いものや変なものを作ってもOKです。

商用作品や製品で使う場合も、許可は不要です。  
もしよければ「こんなのに使ったよ」と教えてもらえるとうれしいです。もちろん強制ではありません。

このリポジトリのコードとドキュメントは、明記がない限りApache License 2.0です。

AIモデル、model weights、データセット、生成物、第三者由来のprompt、provider側assetなどには別の条件が適用される場合があります。それぞれ確認してください。
