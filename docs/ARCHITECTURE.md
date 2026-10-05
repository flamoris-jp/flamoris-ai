# FLAMORIS AI architecture

Architecture authority: [AI #18](https://github.com/flamoris-jp/flamoris-ai/issues/18). This document defines current responsibilities and explicitly marked future boundaries. [PROGRESS.md](../PROGRESS.md) records accepted source, additional review PRs and live acceptance separately.

## External adapters and internal owners

| Caller / capability | Current source contract |
| --- | --- |
| External generation client | MCP Hub → Generation MCP facade → shared Controller → providers |
| External Intelligence client | MCP Hub → Intelligence MCP facade → shared provider adapters |
| External Agent client | Agent MCP facade → Agent state/execution contracts |
| External GPU lifecycle client | GPU Manager MCP facade, optionally through Hub → GPU Node Manager |
| Studio generation | Authenticated Controller HTTP v1 → same Controller → providers |
| Studio raw Intelligence | Shared `flamoris_intelligence` provider adapters → approved providers |
| Studio Agent Support | Agent JSON HTTP `/api/v1` → shared provider adapters → approved providers |

External MCP adapters and internal callers share their owner's non-MCP contract. Intelligence/Agent source already implements this separation; the matched Controller #5 / Generation #71 / Studio #65 PRs now implement generation. The generation rows describe those open/unmerged source PRs, not an already deployed installation. The facade hosts both adapters around one Controller on its existing listener, adding no compulsory daemon or port.

Raw inference uses the provider contract directly. Agent supplies optional personality through its internal HTTP and separate external MCP surfaces.

## Responsibility matrix

| Owner | Responsibility / status | Boundary |
| --- | --- | --- |
| Generation Controller | Implemented MCP-free retained generation domain; source PRs, live cutover pending | Reuse retained domain with one authority; inference kernel and host lifecycle remain independent |
| Generation MCP | External MCP tools/translation and co-hosted internal HTTP adapter | Retained domain is extracted into Controller; external MCP compatibility remains |
| Intelligence MCP | Shared non-MCP provider adapters and optional external MCP translation | Agent owns durable personality/conversation state |
| AI Agent | Optional personality, conversation/memory, principal/session and context policy | Raw inference and generation have their own independently authorized paths |
| AI Runtime | Model-adjacent inference, ExecuteFlow, compiled ExecutionPlan, Jobs/Continuations and resources | ComfyUI owns its graph execution; Agent owns durable personality state |
| MCP Hub | External catalog/connections/routing | Internal provider execution remains with domain owners |
| GPU Node Manager | Host-wide configured runtime/GPU transitions | Generation owns provider requests and jobs; host lifecycle stays separate |
| Studio | Authenticated UI, drafts, user-scoped access and presentation | Provider internals, scheduling, domain state and host lifecycle stay with their respective owners |

## Generation Controller implementation direction

The implementation is coordinated in [Controller #1](https://github.com/flamoris-jp/flamoris-generation-controller/issues/1), [the retained-source inventory](https://github.com/flamoris-jp/flamoris-generation-controller/blob/640a5bd48c76e4bf736e9a3589c216123ccd18b3/docs/IMPLEMENTATION.md) and [HTTP contract](https://github.com/flamoris-jp/flamoris-generation-controller/blob/640a5bd48c76e4bf736e9a3589c216123ccd18b3/docs/API.md). The scoped 2026-10-05 code instruction supersedes preparation-only holds.

| Concern | Implemented source decision |
| --- | --- |
| Core | `flamoris_generation_controller` owns retained recipes, model/capability metadata, provider adapters, jobs/results, managed inputs/uploads/staging, assets/transfer and retention; no MCP/Hub/Studio imports |
| Shared state | One runtime/lifecycle in Generation MCP's host, used by both adapters. Output-root lifetime ownership lock is acquired before provider construction/recovery; duplicate processes fail before effects |
| External facade | 23 retained tool names/annotations, signed MCP ingress, wire validation and SDK errors/binary mapping remain in Generation MCP |
| Studio | Direct authenticated JSON/binary HTTP; retains accounts/CSRF, owner-scoped opaque mappings, history/presets, request fences and publication rechecks |
| Specification | Controller validates retained generation constraints/profiles; Studio retains consumer DTO checks, browser limits and untrusted-result validation |
| Compatibility | Schemas 1/4/5/6, job/recipe/input/asset IDs, provider/storage config, journals/leases/copy records, verified provenance and unknown/no-replay are unchanged |

Internal POST `/api/v1/generation/{operation}` has strict bounded JSON requests, direct object results, binary `assets.get` and constant safe error codes. A separate private operator `FLAMORIS_CONTROLLER_TOKEN` grants the trusted backend the bounded service surface; Studio uses the same service credential and continues all user authorization. Public JSON/identity headers cannot inject trusted context; external signed provenance is separate and is not an ownership grant. An unset internal credential disables internal effects while external MCP remains usable.

Both ingress paths contend for the same durable reservation. Disconnect/restart/timeout never authorizes resubmission; ambiguous provider acceptance or journal commit stays unknown/reserved. Lifetime lock exclusion is local to one output root, not a distributed scheduler/GPU lock, and cannot constrain old binaries that do not acquire it. Operational cutover must drain/reconcile and stop the previous matched authority, preserve/back up records and explicitly update Studio endpoint/token/empty namespace. No DB migration is introduced.

The removed custom registry/versioning/v3/qualification/Runtime bridge is not copied. Reference-image/new-provider features remain separate. Source PR review/CI/merge and live cutover are recorded independently in PROGRESS §4.6.

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

Agent conversations, Runtime active Jobs and generation jobs are different authorities. Referencing work does not transfer its lifecycle or authorize access. The shared generation boundary must not instantiate one independent JobStore/reservation per frontend.

Preserve identities, retained assets/inputs/conversations, provenance and uncertain accepted work during later code changes. Source deletion is not persistent-data deletion. Retained paths keep scoped authentication/authorization, complete-context remote consent, bounded I/O/time/resources, safe staging, declared outputs, redacted errors and no hidden replay/fallback. Host readiness, provider availability, actual generation acceptance and caller authorization are separate checks.

## Acceptance and remaining work

The [README source matrix](../README.md#implemented-source-boundaries) links accepted contracts. [PROGRESS.md](../PROGRESS.md) and the [review report](REVIEW_2026-10-05.md) retain source/test history and accepted follow-up fixes; [roadmap](ROADMAP.md) lists remaining work. Controller implementation and matched callers are in open source PRs; main acceptance remains pending, and live deployment, retained-data reconciliation and new reference-image features retain separate scopes.

## 日本語

MCPは外部入口、Agentは任意の人格、Runtimeは推論とExecuteFlow、ExecutionPlanはコンパイル済み表現です。ComfyWorkFlowはComfyUIが実行するグラフ・JSONです。

Studioのraw IntelligenceとAgent内部の実行は共通provider adapterへ直接接続し、StudioのAgent SupportはAgent JSON HTTPを使います。GenerationもControllerを実装し、Studioの認証付きHTTPと外部MCPが同じ生成状態・予約を使うソースへ切り替えました。保存形式と基本/native生成の保護は保持します。実装PRは未マージで、実機受け入れは未着手です。
