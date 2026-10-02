# FLAMORIS AI

Home and coordination point for the AI-facing parts of FLAMORIS.

This repository is intentionally small. It documents architecture boundaries, cross-repository dependency direction, and the integrated roadmap without becoming a second runtime or implementation authority.

See the [AI ecosystem map](docs/ai-ecosystem.md), [integrated roadmap](docs/ROADMAP.md), and [top-level roadmap tracker](https://github.com/flamoris-jp/flamoris-ai/issues/4).

## Proposed integration design

[Multimodal Studio architecture](docs/MULTIMODAL_STUDIO_ARCHITECTURE.md) records the proposed media/Workflow/Agent integration coordinated by [FLAMORIS AI #15](https://github.com/flamoris-jp/flamoris-ai/issues/15). It is a design proposal, not a claim that new providers, composed execution or shared-user Agent assistance are implemented. Existing public contracts and readiness gates remain authoritative.

## Repository map

```text
flamoris-ai
├── flamoris-ai-agent
│   └── persistent Agent runtime + Agent MCP surface:
│       identity / conversations / memory / knowledge / personality / tools
├── flamoris-ai-runtime
│   └── model-adjacent execution runtime:
│       inference / workflow / jobs / interrupts / capabilities / traces
├── Maidionis
│   ├── specialization-neutral trainable foundation:
│   │   training / evaluation / artifacts / bounded inference contracts
│   └── Arbitrium
│       └── first Decision specialization:
│           bounded advisory judgments / abstention / research evidence
├── Oblivionis
│   └── experimental non-LLM model for experience-dependent behavior:
│       Active Field / firing / runtime modulation / recall / Profundumis
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

Two important execution paths now coexist:

```text
Agent → Intelligence MCP → model/provider
```

for provider-neutral intelligence service access, and:

```text
Agent / Application → AI Runtime → controlled model execution / capabilities
                           ├─→ Maidionis specialization
                           │     └─→ Arbitrium (Decision)
                           └─→ Oblivionis

Planned response path:
Oblivionis → history-shaped firing signals → bounded AI Runtime modulation
```

for model-adjacent execution that needs direct control over inference, workflow state, jobs, interrupts, and runtime events. The response path is a planned data/control-signal boundary, not a transfer of execution ownership to Oblivionis.

The Agent owns persistent FLAMORIS-aware identity and durable state. AI Runtime owns bounded active execution for the models/backends it directly controls, including whether and how modulation is applied. Maidionis owns specialization-neutral model, training, evaluation, artifact, and bounded inference contracts; Arbitrium owns Decision-specific task semantics, curricula, experiment evidence, and specialization metadata. AI Runtime, rather than either model repository, owns workflow execution and orchestration. Oblivionis owns its evolving model state, firing responses, and latent recall semantics, not Agent identity or workflow orchestration. Intelligence MCP owns provider-neutral language/reasoning/coding service execution. Generation MCP owns media workflows/jobs/assets. GPU Node Manager owns local GPU runtime transitions without making a particular node name part of the architecture.

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

### FLAMORIS AI Runtime

[flamoris-ai-runtime](https://github.com/flamoris-jp/flamoris-ai-runtime) is the model-adjacent execution layer for inference and workflow control.

Its design scope includes active inference lifecycle, Workflow IR execution, scheduler-visible Jobs, Continuations, interrupts, registered capabilities, resource accounting, and structured runtime events/traces. It may call external AI, APIs, MCP services, algorithms, Vision/Speech/Generation capabilities, or future models while preserving the authority of those external systems.

AI Runtime does not own durable Agent identity, conversations, personality, or long-term Agent memory.

### Maidionis

[Maidionis](https://github.com/flamoris-jp/Maidionis) is the specialization-neutral foundation for training, evaluating, and packaging small bounded task-specific AI models.

It owns reusable model/training/inference infrastructure, dataset and manifest contracts, education orchestration, checkpointing, reproducibility, artifact serialization, evaluation/calibration, specialization identity/versioning, and a stable Runtime-facing inference contract. It does **not** own Workflow execution, tool execution, authorization, host lifecycle, retry/fallback orchestration, product state, or GPU/runtime lifecycle policy.

### Arbitrium

[Arbitrium](https://github.com/flamoris-jp/Arbitrium) is the first Maidionis specialization: a Decision model for bounded advisory judgments over supplied evidence.

It owns Decision-specific TaskSpecs, curricula, experiment reports, measured results, failure analysis, and specialization metadata. Specialization-neutral machinery belongs in Maidionis, while execution/orchestration remains in AI Runtime. The current public repository is a research-migration/bootstrap boundary and does not claim production-quality generalization.

### Oblivionis

[Oblivionis](https://github.com/flamoris-jp/Oblivionis) is an experimental non-LLM dynamic state and memory model exploring **AI behavior that changes with experience**.

Its intended role is **experience → evolving oscillatory state → firing → runtime fluctuation → changed behavior**. It is not only a memory lookup or a detector that starts Workflows. The model supplies history-shaped responses; AI Runtime decides how to apply them through explicit, bounded modulation mappings, including at supported points of existing execution.

Planned model ownership includes the Active Field, oscillation/resonance/fluctuation dynamics, firing responses, forgetting, associative/resonance recall, snapshots, and **Profundumis**. It does not own Agent identity, workflow scheduling, media-generation domains, or external asset lifecycle. Firing-derived modulation is not a claim of biological brain simulation or an already-integrated runtime capability.

Keep modulation separate from [sensor/Workflow-start triggers](https://github.com/flamoris-jp/Oblivionis/issues/3). Keep [Profundumis association/recall](https://github.com/flamoris-jp/Oblivionis/issues/4) separate as well: stored Max is used for search and percentage reactivation **after latent storage**, not to drive normal firing or continuously restore active state. Refer to the owning [model concept](https://github.com/flamoris-jp/Oblivionis/blob/main/docs/MODEL.md) rather than duplicating its implementation specification here.

The repository intentionally keeps the name **Oblivionis** without the `flamoris-` prefix: it is developed within FLAMORIS while retaining an independent model identity.

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
   - Model-adjacent active inference/workflow execution belongs to AI Runtime for the backends it directly controls.
   - Maidionis owns specialization-neutral model/training/evaluation/artifact contracts for bounded specializations.
   - Arbitrium owns Decision-specific semantics, curricula, experiment evidence, and specialization metadata.
   - Oblivionis owns its dynamic model state, firing responses, forgetting, and recall semantics; Runtime owns application of modulation.
   - Language/reasoning/coding provider service execution belongs to Intelligence MCP.
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

- **[flamoris-ai-runtime](https://github.com/flamoris-jp/flamoris-ai-runtime)**  
  モデルに隣接して、推論・Workflow・Job・Continuation・割り込み・Capability・Resource・Event/Traceを同じ実行層で制御するAI Runtime。永続Agent identityやConversationのauthorityにはなりません。

- **[Maidionis](https://github.com/flamoris-jp/Maidionis)**  
  小さな専門AIを教育・評価・packageするためのspecialization-neutralな共通基盤。model / training / evaluation / artifact / bounded inference contractを持ち、Workflow実行やRuntime orchestrationは持ちません。

- **[Arbitrium](https://github.com/flamoris-jp/Arbitrium)**  
  Maidionis最初のDecision specialization。Decision固有のTaskSpec、curriculum、実験結果、failure analysis、specialization metadataを持ちます。現在はresearch migration / bootstrap段階です。

- **[Oblivionis](https://github.com/flamoris-jp/Oblivionis)**  
  **経験で変わる振動状態から発火し、その発火をもとにRuntimeへ揺らぎを与えて、AIの振る舞いを変える**ことを目指す実験的な非LLMモデル。現在の場、忘却、連想・想起、深淵Profundumisを扱います。AgentやWorkflow engineにはせず、`flamoris-`を付けない独立したモデルidentityを残します。

- **[flamoris-intelligence-mcp](https://github.com/flamoris-jp/flamoris-intelligence-mcp)**  
  素のLLM、推論、Codingなどを扱うprovider-neutralなIntelligence gateway。AgentのMemoryやConversationのauthorityにはなりません。

- **[flamoris-generation-mcp](https://github.com/flamoris-jp/flamoris-generation-mcp)**  
  画像・動画・音楽・音声などのgenerationと、同じworkflow/job/asset lifecycleに属するmedia analysisを扱います。

- **[flamoris-mcp-hub](https://github.com/flamoris-jp/flamoris-mcp-hub)**  
  `generation.*` / `intelligence.*` / `agent.*` / `lime.*` をまとめる入口。workflow authorityにはしません。

- **[flamoris-gpu-node-manager](https://github.com/flamoris-jp/flamoris-gpu-node-manager)**  
  ローカルGPU runtimeのauthority。LIMEはdeploymentのひとつで、systemdやGPU切替ロジックを他サービスへ複製しません。

Oblivionisの発火を実行中の振る舞いへ作用させることと、新しい処理を始めるトリガにすることは別です。発火の出力はOblivionis、適用場所・強さ・実行判断はRuntime側が担当します。この連携は構想段階で、実装済みの機能を示すものではありません。

Maxの記録・保持と利用も区別します。**Maxでの検索と割合復活は、Profundumisへしまった後の連想・想起に使い、通常の発火や活動をMaxへ戻し続けません。** 詳細は [Oblivionisのモデル概念](https://github.com/flamoris-jp/Oblivionis/blob/main/docs/MODEL.md)、[トリガ #3](https://github.com/flamoris-jp/Oblivionis/issues/3)、[深淵の連想・想起 #4](https://github.com/flamoris-jp/Oblivionis/issues/4) を参照してください。

中心となる依存方向は:

```text
Agent → Intelligence MCP → model/provider
```

です。これとは別に、model-adjacentな実行では次の呼び出しと応答の境界を設計します。

```text
Agent / Application → AI Runtime ─┬→ Maidionis specialization
                          │           └→ Arbitrium (Decision)
                          └→ Oblivionis
                               ↑     │
                               └─ 発火由来の信号
                             → Runtime側で揺らぎとして適用
```

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
