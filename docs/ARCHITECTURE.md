# FLAMORIS AI architecture

Decision: [AI #18](https://github.com/flamoris-jp/flamoris-ai/issues/18), 2026-10-04. This document defines the corrected target, not a migrated deployment. The current pass is documentation review, correction and explicitly authorized documentation merge only.

## External adapters and internal owners

```text
External:
ChatGPT -> MCP Hub -> Generation MCP -> future Generation Controller -> providers
                  -> Intelligence MCP -> internal intelligence capabilities
                  -> GPU Manager MCP surface -> GPU Node Manager

Internal:
Studio generation -> future Generation Controller -> generation providers
Studio raw intelligence -> internal runtime / API / vendor interface
Studio Agent Support -> AI Agent -> internal execution interface
```

Calls behind an external MCP adapter use the owner's non-MCP internal contract. Internal FLAMORIS components do not route through MCP or MCP Hub. A replaceable interface does not prescribe an additional network service.

Intelligence MCP may expose an explicitly requested personality capability through an internal Agent call. Raw inference does not require Agent. Existing external Agent MCP compatibility is reviewed separately; removing internal MCP dependencies is not permission to remove every external MCP surface.

## Responsibility matrix

| Owner | Target responsibility | Not its responsibility |
| --- | --- | --- |
| Generation Controller | Future internal generation providers/adapters, generation requests/jobs/results, managed inputs/references and assets | Implementation now; copying the old MCP-side ComfyWorkFlow subsystem; inference kernel or host lifecycle |
| Generation MCP | External tools, protocol validation, discovery and result translation | Permanent internal generation-domain authority |
| Intelligence MCP | External intelligence facade and MCP translation | Internal universal execution gateway or durable Agent state |
| AI Agent | Optional personality, conversation/memory, principal/session and context policy | Mandatory mediation of all AI work |
| AI Runtime | Model-adjacent inference, ExecuteFlow, compiled ExecutionPlan, Jobs/Continuations and resources | ComfyUI graph construction or durable Agent identity |
| MCP Hub | External catalog/connections/routing | Internal service bus, provider selection or execution engine |
| GPU Node Manager | Host-wide configured runtime/GPU transitions | Generation jobs or per-ComfyWorkFlow approval |
| Studio | Authenticated UI, drafts, user-scoped access and presentation | Provider graph internals, duplicate domain stores or host state machine |

## Three distinct names

### ExecuteFlow

The inference dependency/data/control flow owned by AI Runtime. It describes associated steps and supported waits, branches, interrupts and resumption. Runtime retains model-adjacent inference control rather than becoming only an API orchestrator.

### ExecutionPlan

The Runtime's existing compiled representation. The current C++ type is `ExecutionPlan` in `include/flamoris/runtime/compiler.hpp`; preserve it. Conceptually, an ExecuteFlow definition is validated/compiled into an ExecutionPlan, and the scheduler runs Jobs. Do not collapse the source description, compiled representation and active Job state.

Current source/wire identifiers such as `WorkflowIR` and `workflow.node.*` are not renamed by this documentation change. A later explicit naming migration must map each symbol and preserve or version external contracts. The abandoned idea of naming both concepts ExecutionPlan is not the target.

### ComfyWorkFlow

A ComfyUI execution graph / API-format JSON with declared parameter/reference bindings. ComfyUI executes the graph. Other providers may use ordinary generation requests/recipes without a ComfyWorkFlow.

```text
trusted ComfyUI graph + allowed bindings + validated values
  -> ComfyWorkFlow JSON construction
  -> separate provider submission
  -> ComfyUI execution
```

The existing Generation MCP ComfyWorkFlow implementation is to be removed in a later authorized Generation cleanup, not transferred to Controller or Runtime. The precise source/tool/dependent-test scope must be inventoried first. Controller is still unimplemented; this decision does not order a replacement implementation or a rebuild of the same subsystem.

JSON construction can be tested offline. Submission, installed node/model compatibility, generation qualification and user authorization are separate checks. Do not make Agent, a generic scheduler or a Runtime bridge prerequisites for JSON construction. Removing obsolete functionality must not create a safety bypass for retained functionality.

Use the specific names in new design prose. Retain exact spelling for current API/config symbols, external names and historical quotations rather than inventing a completed migration.

## Gateway means replaceability

Keep internal execution details behind narrow interfaces. Removing internal MCP does not mean hard-coding providers into Studio, replicating credentials everywhere or inventing a new central gateway service. Reusable intelligence adapter ownership and the minimal replacement path must be decided against actual source and callers before code removal. No new Intelligence Controller repository is implied.

## State and safety

Agent conversations, Runtime active Jobs and generation jobs are different authorities. Referencing work does not transfer its lifecycle or authorize access. A future shared generation boundary must not instantiate one independent JobStore/reservation per frontend.

Preserve identities, retained assets/inputs/conversations, provenance and uncertain accepted work during later code changes. Source deletion is not persistent-data deletion. Retained paths keep scoped authentication/authorization, complete-context remote consent, bounded I/O/time/resources, safe staging, declared outputs, redacted errors and no hidden replay/fallback. Lifecycle READY is not proof that a particular ComfyWorkFlow is qualified.

## Sequencing

1. Review/fix/merge these documentation PRs. No source, configuration or deployment changes.
2. Prepare the Intelligence removal inventory and smallest working non-MCP execution contract for a separately scoped Work implementation.
3. Start that code cleanup only under explicit implementation authorization. Do not mix it with new persona features or Runtime kernel redesign.
4. Keep Controller implementation and Generation/ComfyWorkFlow/reference-image work paused. Revisit later under separate authorization.
5. Live cutover, retained-data recovery and rollback require their own operational approval.

The [roadmap](ROADMAP.md) links the owning tasks. A document merge does not resume code execution or production work.

## 日本語

MCPは外部入口、Agentは任意の人格、Runtimeは推論とExecuteFlow、既存ExecutionPlanはコンパイル済み表現です。ComfyWorkFlowはComfyUI用グラフ・JSONであり、Runtimeへ移しません。

Intelligence整備を先行します。Generation Controllerは未実装のまま。Generation MCPの既存ComfyWorkFlow実装は後の削除対象で、Controllerへ移植したり同じものを作り直したりする指示ではありません。今回は文書のレビュー・修正・マージまでです。
