# FLAMORIS AI architecture

Architecture authority: [AI #18](https://github.com/flamoris-jp/flamoris-ai/issues/18). This document defines current responsibilities and explicitly marked future boundaries. [PROGRESS.md](../PROGRESS.md) records accepted source, additional review PRs and live acceptance separately.

## External adapters and internal owners

| Caller / capability | Current source contract |
| --- | --- |
| External generation client | MCP Hub → Generation MCP → co-located generation domain/providers |
| External Intelligence client | MCP Hub → Intelligence MCP facade → shared provider adapters |
| External Agent client | Agent MCP facade → Agent state/execution contracts |
| External GPU lifecycle client | GPU Manager MCP facade, optionally through Hub → GPU Node Manager |
| Studio generation | Generation MCP compatibility route → generation domain/providers |
| Studio raw Intelligence | Shared `flamoris_intelligence` provider adapters → approved providers |
| Studio Agent Support | Agent JSON HTTP `/api/v1` → shared provider adapters → approved providers |

The target is for external MCP adapters and internal callers to share the owner's non-MCP contract. Intelligence/Agent source implements this separation. Generation retains its MCP compatibility boundary while Controller is unimplemented. Controller is a future shared generation-domain contract; design and implementation are held. A replaceable interface does not prescribe an additional network service.

Raw inference uses the provider contract directly. Agent supplies optional personality through its internal HTTP and separate external MCP surfaces.

## Responsibility matrix

| Owner | Responsibility / status | Boundary |
| --- | --- | --- |
| Generation Controller | Future common non-MCP contract for generation providers, requests/jobs/results, inputs and assets; documentation only | Concrete design/implementation held; inference kernel and host lifecycle remain independent |
| Generation MCP | Current bounded generation domain/providers plus external tools, validation, discovery and result translation | Future domain separation requires a reviewed Controller contract |
| Intelligence MCP | Shared non-MCP provider adapters and optional external MCP translation | Agent owns durable personality/conversation state |
| AI Agent | Optional personality, conversation/memory, principal/session and context policy | Raw inference and generation have their own independently authorized paths |
| AI Runtime | Model-adjacent inference, ExecuteFlow, compiled ExecutionPlan, Jobs/Continuations and resources | ComfyUI owns its graph execution; Agent owns durable personality state |
| MCP Hub | External catalog/connections/routing | Internal provider execution remains with domain owners |
| GPU Node Manager | Host-wide configured runtime/GPU transitions | Generation owns provider requests and jobs; host lifecycle stays separate |
| Studio | Authenticated UI, drafts, user-scoped access and presentation | Provider internals, scheduling, domain state and host lifecycle stay with their respective owners |

## Three distinct names

### ExecuteFlow

The inference dependency/data/control flow owned by AI Runtime. It describes associated steps and supported waits, branches, interrupts and resumption. Runtime retains model-adjacent inference control rather than becoming only an API orchestrator.

### ExecutionPlan

The Runtime's existing compiled representation. The current C++ type is `ExecutionPlan` in `include/flamoris/runtime/compiler.hpp`; preserve it. Conceptually, an ExecuteFlow definition is validated/compiled into an ExecutionPlan, and the scheduler runs Jobs. Do not collapse the source description, compiled representation and active Job state.

Current source identifiers such as `WorkflowMachine` and literal schema/path names keep their implemented spelling. Any later naming migration must map real identifiers and preserve or version external contracts.

### ComfyWorkFlow

A ComfyUI execution graph / API-format JSON. The current generation package builds bounded builtin image graphs from trusted templates and validated parameters; ComfyUI executes them. Other providers use their declared generation request/recipe contracts. Generic provider recipes remain with the current generation owner.

Template JSON construction can be tested offline. Submission depends separately on installed node/model compatibility, provider acceptance and user authorization. The retained generation paths preserve those checks and their data protections.

Use the specific names in new design prose. Retain exact spelling for current API/config symbols, external names and historical quotations rather than inventing a completed migration.

## Gateway means replaceability

Keep internal execution details behind narrow interfaces. The neutral `flamoris_intelligence` namespace in the existing Intelligence distribution owns reusable provider adapters. Agent and Studio raw inference import that namespace without using MCP; Studio Agent Support uses Agent's versioned JSON HTTP API. External MCP facade translation remains separate. No new central gateway service or Intelligence Controller repository is introduced.

## State and safety

Agent conversations, Runtime active Jobs and generation jobs are different authorities. Referencing work does not transfer its lifecycle or authorize access. A future shared generation boundary must not instantiate one independent JobStore/reservation per frontend.

Preserve identities, retained assets/inputs/conversations, provenance and uncertain accepted work during later code changes. Source deletion is not persistent-data deletion. Retained paths keep scoped authentication/authorization, complete-context remote consent, bounded I/O/time/resources, safe staging, declared outputs, redacted errors and no hidden replay/fallback. Host readiness, provider availability, actual generation acceptance and caller authorization are separate checks.

## Acceptance and remaining work

The [README source matrix](../README.md#implemented-source-boundaries) links accepted contracts. [PROGRESS.md](../PROGRESS.md) and the [review report](REVIEW_2026-10-05.md) retain source/test history and additional open PRs; [roadmap](ROADMAP.md) lists remaining work. Live cutover, retained-data reconciliation, Controller design/implementation and new reference-image features are separate pending or held phases.

## 日本語

MCPは外部入口、Agentは任意の人格、Runtimeは推論とExecuteFlow、ExecutionPlanはコンパイル済み表現です。ComfyWorkFlowはComfyUIが実行するグラフ・JSONです。

Studioのraw IntelligenceとAgent内部の実行は共通provider adapterへ直接接続し、StudioのAgent SupportはAgent JSON HTTPを使います。GenerationにはMCP互換経路が残り、基本/native生成と保存データの保護を現行packageが担当します。Generation Controllerは文書のみで未実装、具体設計・実装は保留です。実機受け入れは別工程です。
