# FLAMORIS AI architecture

Decision: [AI #18](https://github.com/flamoris-jp/flamoris-ai/issues/18), 2026-10-04. The documentation pass is complete; the next explicit instruction authorizes deletion-first implementation and connection correction. This document distinguishes the architecture and source acceptance from deployed routes.

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

The target is for external MCP adapters to call the owner's non-MCP contract and for internal components to avoid MCP/Hub routing. Intelligence/Agent source now implements that separation. Generation still uses its retained MCP compatibility boundary while Controller is unimplemented. A replaceable interface does not prescribe an additional network service.

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

Current source identifiers such as `WorkflowMachine` and literal schema/path names are not renamed by this documentation change. A later explicit naming migration must map each real identifier and preserve or version external contracts. A prose heading does not establish a C++ type. The abandoned idea of naming both concepts ExecutionPlan is not the target.

### ComfyWorkFlow

A ComfyUI execution graph / API-format JSON with declared parameter/reference bindings. ComfyUI executes the graph. Other providers may use ordinary generation requests/recipes without a ComfyWorkFlow.

```text
trusted ComfyUI graph + allowed bindings + validated values
  -> ComfyWorkFlow JSON construction
  -> separate provider submission
  -> ComfyUI execution
```

The added Generation MCP ComfyWorkFlow definition registry, versioning, composition and Runtime delegation are retired in the cleanup, not transferred to Controller or Runtime. The original bounded image-generation templates and generic provider recipe store have independent callers and remain. Controller is still unimplemented; no replacement subsystem is ordered. Retired definitions and uncertain provider jobs are retained as data rather than executed or silently replayed.

JSON construction can be tested offline. Submission, installed node/model compatibility, generation qualification and user authorization are separate checks. Do not make Agent, a generic scheduler or a Runtime bridge prerequisites for JSON construction. Removing obsolete functionality must not create a safety bypass for retained functionality.

Use the specific names in new design prose. Retain exact spelling for current API/config symbols, external names and historical quotations rather than inventing a completed migration.

## Gateway means replaceability

Keep internal execution details behind narrow interfaces. The neutral `flamoris_intelligence` namespace in the existing Intelligence distribution owns reusable provider adapters. Agent and Studio raw inference import that namespace without using MCP; Studio Agent Support uses Agent's versioned JSON HTTP API. External MCP facade translation remains separate. No new central gateway service or Intelligence Controller repository is introduced.

## State and safety

Agent conversations, Runtime active Jobs and generation jobs are different authorities. Referencing work does not transfer its lifecycle or authorize access. A future shared generation boundary must not instantiate one independent JobStore/reservation per frontend.

Preserve identities, retained assets/inputs/conversations, provenance and uncertain accepted work during later code changes. Source deletion is not persistent-data deletion. Retained paths keep scoped authentication/authorization, complete-context remote consent, bounded I/O/time/resources, safe staging, declared outputs, redacted errors and no hidden replay/fallback. Lifecycle READY is not proof that a particular ComfyWorkFlow is qualified.

## Sequencing

1. The documentation PRs are reviewed, corrected and merged.
2. Inventory internal MCP clients and obsolete generation additions; retain independent state and safety contracts.
3. Implement Studio-to-Agent internal HTTP and shared non-MCP provider adapters for Agent/raw Studio inference. Keep external MCP facades separate. Do not mix this with new persona features or Runtime kernel redesign.
4. Retire the generation registry/composition/delegation and matching caller/catalog assumptions. Preserve Runtime's real ExecuteFlow and compiled ExecutionPlan. Keep Controller unimplemented and new generation/reference-image features deferred.
5. Verify, independently review, fix and merge the source changes. Live cutover and retained-data recovery remain a separate operational task.

The [README source matrix](../README.md#implemented-source-boundaries) and
[roadmap](ROADMAP.md) link the implemented contracts and owning tasks. The later
explicit instruction authorized source implementation and merge; it does not
establish a production cutover. Generation's compatibility exception and held
Controller/reference-image work remain explicit.

## 日本語

MCPは外部入口、Agentは任意の人格、Runtimeは推論とExecuteFlow、既存ExecutionPlanはコンパイル済み表現です。ComfyWorkFlowはComfyUI用グラフ・JSONであり、Runtimeへ移しません。

Intelligence整備を先行し、Studio・Agentの内部MCP依存を直接接続へ置き換えます。Generationの後付け登録・合成・Runtime委譲を削除し、基本生成と保存済みデータの保護は維持します。Generation Controllerは未実装のままで、同じ仕組みの移植や再作成はしません。実機移行は別工程です。
