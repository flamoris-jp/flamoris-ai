# FLAMORIS AI Ecosystem

This document is the cross-repository map for the AI- and MCP-facing parts of FLAMORIS.

It describes **stable responsibility, authority, dependency direction, and interoperability**. Fast-changing implementation status, open bugs, release gates, and near-term sequencing remain authoritative in each repository and in [the integrated roadmap](ROADMAP.md).

## At a glance

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
Generation MCP  │   AI Agent   GPU Node Manager
                │        │
                └───┬────┘
                    ▼
             Intelligence MCP
                    │
                    ▼
           local / remote providers
```

These are responsibility boundaries, not mandatory network layers. Applications may call Generation MCP or Intelligence MCP directly when a persistent Agent is not needed.

The central dependency direction is:

```text
Agent → Intelligence MCP → model/provider
```

## 🤖 Core AI repositories

### 🌱 FLAMORIS AI Agent

Repository: [flamoris-jp/flamoris-ai-agent](https://github.com/flamoris-jp/flamoris-ai-agent)

The Agent is the persistent FLAMORIS-aware AI runtime.

It owns:

- identity and personality;
- conversations and messages;
- memory;
- knowledge context;
- prompts and Agent policy;
- bounded Agent MCP surface;
- later tools and durable Agent orchestration state.

It may call Intelligence MCP and Generation MCP, but those services must not become second owners of Agent state.

The Agent MCP surface belongs inside `flamoris-ai-agent` unless a real deployment or ownership boundary later justifies a separate repository.

### 🧠 FLAMORIS Intelligence MCP

Repository: [flamoris-jp/flamoris-intelligence-mcp](https://github.com/flamoris-jp/flamoris-intelligence-mcp)

Intelligence MCP is the provider-neutral boundary for raw language, reasoning, coding, and related bounded intelligence execution.

It may route to local or remote providers.

It does **not** own:

- persistent Agent conversations;
- Agent memory or knowledge;
- personality or Agent policy;
- generated-media workflows/assets;
- product documents;
- local GPU runtime switching.

Provider details such as llama.cpp remain behind adapters rather than becoming the public FLAMORIS contract.

### 🧪 FLAMORIS Generation MCP

Repository: [flamoris-jp/flamoris-generation-mcp](https://github.com/flamoris-jp/flamoris-generation-mcp)

Generation MCP owns provider-neutral generative-media and closely related media-analysis workflows.

Its domain includes:

- capabilities;
- workflows;
- jobs;
- providers;
- generated/materialized assets.

The conceptual execution flow is:

```text
Capability
    ↓
Workflow
    ↓
Job
    ↓
Provider
    ↓
Assets
```

ComfyUI is a provider, not the identity of the project. Other image, video, music, voice, or analysis providers can live behind the same explicit provider boundary when their contracts fit.

## 🔌 Routing and runtime boundaries

### 🔀 FLAMORIS MCP Hub

Repository: [flamoris-jp/flamoris-mcp-hub](https://github.com/flamoris-jp/flamoris-mcp-hub)

MCP Hub is the aggregation and routing boundary.

Its namespace family is organized around owning services:

```text
generation.*
intelligence.*
agent.*
lime.*
```

The Hub should:

- expose one MCP-facing entry point when useful;
- preserve upstream ownership;
- route namespaced calls;
- keep upstream composition explicit.

The Hub must not become:

- a second Agent;
- a Generation workflow engine;
- an Intelligence state machine;
- a duplicate GPU/runtime authority.

Using the Hub is optional architecture. Direct application-to-service calls remain valid when a single stable MCP boundary is sufficient.

### 🖥️ FLAMORIS GPU Node Manager

Repository: [flamoris-jp/flamoris-gpu-node-manager](https://github.com/flamoris-jp/flamoris-gpu-node-manager)

GPU Node Manager is the provider-neutral local GPU/runtime lifecycle authority.

It owns bounded runtime transitions such as activation, stop, health/state observation, and shared local GPU coordination through its public interfaces.

Other services may request or inspect runtime state, but they must not duplicate the host's systemd/GPU transition state machine.

**LIME is one deployment identity, not the public service identity.**

Machine names, private topology, local usernames, tunnel IDs, credentials, and host-specific paths must not leak into the public architecture.

## 🎛️ Product boundary

### FLAMORIS Studio

Repository: [flamoris-jp/flamoris-studio](https://github.com/flamoris-jp/flamoris-studio)

Studio is the human-facing multi-user creative control plane.

It connects users to Intelligence and Generation capabilities while keeping Studio-specific UI, orchestration, account/session, and access-control concerns inside Studio.

Studio does not own:

- local GPU runtime transitions;
- Generation provider internals;
- raw Intelligence provider internals;
- desktop product document state.

Generated job/asset authority stays with the owning MCP service; Studio presents and authorizes access to those capabilities for its users.

Desktop products such as FLAMORIS 2D, Cutwork, and Kachinco remain authoritative for their own project/document state and editing behavior.

## Authority rules

Across the AI family:

1. **One authority per domain.**  
   Agent state belongs to the Agent. Intelligence execution belongs to Intelligence MCP. Generation workflows/jobs/assets belong to Generation MCP. Runtime transitions belong to GPU Node Manager. Product state belongs to the product.

2. **Provider-neutral public boundaries.**  
   Local and remote providers are implementation choices behind explicit adapters. A provider name should not become the architecture unless the public behavior is intentionally provider-specific.

3. **Local-first, not local-only.**  
   Local providers are first-class, but the architecture must not assume one machine or one runtime.

4. **Hub is optional routing, not ownership.**  
   Hub may aggregate services, but upstream services remain authoritative.

5. **AI services do not silently become product state stores.**  
   Product/application repositories retain authority for their own documents, projects, and editing semantics.

6. **Bounded behavior over hidden automation.**  
   Inference, generation, tool execution, filesystem access, network access, retries, and resource use should be explicit and bounded.

## Interoperability

The preferred integration style is explicit capability boundaries rather than shared internal state.

Examples:

```text
Studio
  ├─→ Intelligence MCP
  └─→ Generation MCP
```

```text
AI Agent
  ├─→ Intelligence MCP
  └─→ Generation MCP
```

```text
ChatGPT / client
  └─→ MCP Hub
         ├─→ Intelligence MCP
         ├─→ Generation MCP
         ├─→ AI Agent
         └─→ GPU Node Manager
```

A component should call another component because it needs that component's owned capability, not because both are "AI-related".

## Current-status discipline

This file is a **map**, not a second roadmap.

Use it for:

- repository responsibility;
- authority boundaries;
- dependency direction;
- cross-service interoperability;
- stable architectural invariants.

Do not use it as the authority for:

- exact feature completion;
- open bugs;
- live deployment readiness;
- release status;
- test counts;
- current Issue ordering;
- implementation-specific commands.

For those, follow each repository's README, Issues, PRs, and the integrated [ROADMAP](ROADMAP.md).

## Related ecosystem

Organization-wide map:

- [flamoris-jp/.github](https://github.com/flamoris-jp/.github)

Windows/Desktop family map:

- [flamoris-jp/flamoris-commons/docs/desktop-ecosystem.md](https://github.com/flamoris-jp/flamoris-commons/blob/main/docs/desktop-ecosystem.md)

---

# 日本語

この文書は、FLAMORISのAI・MCP系Repositoryを横断して見るための**AI ecosystem全体地図**です。🤖🗺️

ここでは、変化しにくい**責任範囲・authority・依存方向・相互接続**を整理します。細かな実装状況、未解決Issue、release gate、直近の作業順は各Repositoryと [統合ROADMAP](ROADMAP.md) を正とします。

## ひと目で見る全体像

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
Generation MCP  │   AI Agent   GPU Node Manager
                │        │
                └───┬────┘
                    ▼
             Intelligence MCP
                    │
                    ▼
           local / remote providers
```

これは責任分界を示す図であって、必ずこの順番でnetwork接続しなければならないという意味ではありません。

永続Agentが不要なら、ApplicationからGeneration MCPやIntelligence MCPを直接呼んでも構いません。

中心となる依存方向は:

```text
Agent → Intelligence MCP → model/provider
```

です。

## 🤖 AIの中心Repository

### 🌱 FLAMORIS AI Agent

Repository: [flamoris-jp/flamoris-ai-agent](https://github.com/flamoris-jp/flamoris-ai-agent)

AI Agentは、**FLAMORISを知る永続Agent**です。

担当するもの:

- Identity / Personality
- Conversation / Message
- Memory
- Knowledge context
- Prompt / Agent policy
- bounded Agent MCP surface
- 将来のToolやdurable Agent orchestration state

AgentはIntelligence MCPやGeneration MCPを利用できますが、それらにConversationやMemoryの第二authorityを作りません。

Agent MCP surfaceも、実際に独立したdeployment / lifecycle / ownership境界が必要になるまでは `flamoris-ai-agent` 内に置きます。

### 🧠 FLAMORIS Intelligence MCP

Repository: [flamoris-jp/flamoris-intelligence-mcp](https://github.com/flamoris-jp/flamoris-intelligence-mcp)

Intelligence MCPは、言語・推論・Codingなどの**素の知能実行**を扱うprovider-neutralな境界です。

local provider / remote providerのどちらもadapterの内側に置けます。

担当しないもの:

- 永続Conversation
- Agent Memory / Knowledge
- Personality / Agent policy
- 生成mediaのworkflow / asset
- Product document
- ローカルGPU runtimeの切替

llama.cppなどのprovider実装はadapterの内側に閉じ込め、FLAMORISの公開contractそのものにはしません。

### 🧪 FLAMORIS Generation MCP

Repository: [flamoris-jp/flamoris-generation-mcp](https://github.com/flamoris-jp/flamoris-generation-mcp)

Generation MCPは、画像・動画・音楽・音声などのgenerationと、それらに密接なmedia analysis workflowを担当します。

主なauthority:

- Capability
- Workflow
- Job
- Provider
- 生成・materializeされたAsset

基本的な考え方:

```text
Capability
    ↓
Workflow
    ↓
Job
    ↓
Provider
    ↓
Assets
```

ComfyUIはproviderのひとつであって、Generation MCPそのもののidentityではありません。

将来のimage / video / music / voice / analysis providerも、明示的なprovider contractの内側へ追加できます。

## 🔌 Routing・Runtime境界

### 🔀 FLAMORIS MCP Hub

Repository: [flamoris-jp/flamoris-mcp-hub](https://github.com/flamoris-jp/flamoris-mcp-hub)

MCP Hubは、複数MCPをまとめる**aggregation / routing境界**です。

namespaceは、各authorityを持つserviceに沿って分けます。

```text
generation.*
intelligence.*
agent.*
lime.*
```

Hubが担当するもの:

- 必要なときに単一のMCP entry pointを提供する
- upstream ownershipを維持する
- namespace付きtoolをroutingする
- upstream compositionを明示的に保つ

Hubが担当しないもの:

- 第二のAgent
- Generation workflow engine
- Intelligence state machine
- GPU/runtimeの第二authority

Hub経由は便利な構成ですが必須ではありません。1つの安定したMCP境界だけ必要なら、Applicationから各serviceへ直接接続して構いません。

### 🖥️ FLAMORIS GPU Node Manager

Repository: [flamoris-jp/flamoris-gpu-node-manager](https://github.com/flamoris-jp/flamoris-gpu-node-manager)

GPU Node Managerは、ローカルGPU / AI runtime lifecycleのprovider-neutralなauthorityです。

runtimeのactivate / stop、状態観測、ローカルGPU resource coordinationなどをboundedなinterfaceとして提供します。

他serviceはruntime状態を問い合わせたり切替を依頼したりできますが、systemd / GPU transition state machineを各serviceへ複製しません。

**LIMEはdeployment identityのひとつであり、公開serviceのidentityではありません。**

machine name、private topology、username、tunnel ID、credential、host固有pathはpublic architectureへ持ち込みません。

## 🎛️ Product境界

### FLAMORIS Studio

Repository: [flamoris-jp/flamoris-studio](https://github.com/flamoris-jp/flamoris-studio)

Studioは、人間がAI capabilityを使うための**マルチユーザーcreative control plane**です。

Intelligence / Generation capabilityを利用者へつなぎつつ、Studio固有のUI、orchestration、account/session、access controlを担当します。

Studioがauthorityにならないもの:

- ローカルGPU runtime transition
- Generation provider内部
- Intelligence provider内部
- Desktop productのDocument / Project state

Generation job / assetなどの実行authorityは、所有するMCP service側に残します。Studioは利用者に対する表示・操作・authorizationを担当します。

FLAMORIS 2D、Cutwork、KachincoなどのDesktop productは、それぞれ自分のProject / Document / editing behaviorのauthorityを維持します。

## Authority rules / 責任分界ルール

AI familyでは次のルールを守ります。

1. **1 domain = 1 authority。**  
   Agent stateはAgent、Intelligence executionはIntelligence MCP、Generation workflow/job/assetはGeneration MCP、runtime transitionはGPU Node Manager、Product stateは各Productが持ちます。

2. **公開境界はprovider-neutral。**  
   local / remote providerはadapterの内側の実装選択です。意図的なprovider-specific behaviorでない限り、provider名をarchitectureそのものにしません。

3. **Local-firstだがlocal-onlyではない。**  
   local modelを第一級に扱いますが、1台のmachineや1種類のruntimeにarchitectureを固定しません。

4. **Hubはroutingであってownershipではない。**  
   Hubが複数serviceをまとめても、upstream authorityは各serviceに残ります。

5. **AI serviceはProduct stateを勝手に所有しない。**  
   Product / ApplicationのDocument・Project・editing semanticsは各Product側です。

6. **隠れた自動化よりbounded behavior。**  
   inference、generation、tool execution、filesystem、network、retry、resource useは明示的に制限します。

## 🔄 Interoperability / つなぎ方

内部stateを共有するより、明示的なcapability境界でつなぎます。

例:

```text
Studio
  ├─→ Intelligence MCP
  └─→ Generation MCP
```

```text
AI Agent
  ├─→ Intelligence MCP
  └─→ Generation MCP
```

```text
ChatGPT / client
  └─→ MCP Hub
         ├─→ Intelligence MCP
         ├─→ Generation MCP
         ├─→ AI Agent
         └─→ GPU Node Manager
```

「どちらもAIっぽいから」ではなく、**そのRepositoryが所有しているcapabilityが必要だから呼ぶ**という関係にします。🐈

## 🧭 Current-status discipline / ステータス情報の置き場所

この文書は**地図**であって、第二のroadmapではありません。

ここに置くもの:

- Repository responsibility
- authority boundary
- dependency direction
- cross-service interoperability
- 安定したarchitecture invariant

ここを正にしないもの:

- exact feature completion
- open bug
- live deployment readiness
- release status
- test count
- 現在のIssue順
- implementation-specific command

それらは各RepositoryのREADME / Issue / PRと [統合ROADMAP](ROADMAP.md) を正とします。

## Related ecosystem / 関連地図

FLAMORIS全体:

- [flamoris-jp/.github](https://github.com/flamoris-jp/.github)

Windows / Desktop family:

- [flamoris-jp/flamoris-commons/docs/desktop-ecosystem.md](https://github.com/flamoris-jp/flamoris-commons/blob/main/docs/desktop-ecosystem.md)
