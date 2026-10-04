# FLAMORIS AI architecture

Decision: [AI #18](https://github.com/flamoris-jp/flamoris-ai/issues/18), 2026-10-04. This is the corrected target architecture, not a claim that existing code or deployments have been migrated. The current phase is documentation and Issue organization in Chat only.

## External adapters and internal owners

```text
External:
ChatGPT -> MCP Hub -> Generation MCP -> Generation Controller -> providers
                  -> Intelligence MCP -> internal intelligence capabilities
                  -> GPU Manager MCP surface -> GPU Node Manager

Internal:
Studio -> Generation Controller -> generation providers
Studio -> internal intelligence interface -> AI Runtime / API / vendor runtime
Studio -> Agent Support -> AI Agent -> internal execution interface
```

The MCP-to-owner boundary uses the appropriate internal contract; internal FLAMORIS calls do not route back through MCP or MCP Hub. The exact transport is a later design choice, not a mandatory additional network service.

An external Intelligence MCP capability may expose personality-enabled behavior by calling Agent through an internal contract. Raw inference need not use Agent. Existing external Agent MCP compatibility and any consolidation are separately reviewed; removing internal MCP dependencies is not an instruction to delete all external MCP surfaces.

## Responsibility matrix

| Owner | Responsibility | Excluded responsibility |
| --- | --- | --- |
| Generation Controller | Generation providers/adapters, execution-definition construction, generation jobs/results, managed inputs/references, assets and generation validation | MCP routing, inference kernel, Agent memory, host lifecycle |
| Generation MCP | External tool contracts, protocol validation, discovery and result translation | A second generation-domain state owner |
| Intelligence MCP | External intelligence tool facade and protocol translation | Internal universal provider gateway or durable Agent state |
| AI Agent | Optional personality, conversation/memory, principal/session and context policy | Mandatory mediation of all AI work |
| AI Runtime | Model-adjacent inference, ExecutionPlans, active Jobs/Continuations/state/resources | ComfyUI graph construction or durable Agent identity |
| MCP Hub | External catalog/connection/routing and transport boundary | Internal service bus, provider selection, generation or inference scheduling |
| GPU Node Manager | Host-wide configured runtime/GPU transitions and lifecycle coordination | Generation jobs, provider graph semantics or per-ComfyWorkFlow approval |
| Studio | Authenticated UI, product drafts, user-scoped access and presentation | Provider-specific graph implementation, duplicate domain stores or host control logic |

## ExecutionPlan and ComfyWorkFlow are different concepts

### ComfyWorkFlow

For the current ComfyUI path, this means building ComfyUI API-format ComfyWorkFlow JSON from a trusted definition and allowed parameter/reference bindings. The provider executes that graph.

```text
trusted graph + declared bindings + validated values
  -> ComfyWorkFlow Builder
  -> ComfyWorkFlow JSON
  -> separate submission
  -> ComfyUI execution
```

Its future owner is Generation Controller, but the current Generation MCP implementation is not transferred there. Controller remains unimplemented in this phase; the misplaced MCP-side ComfyWorkFlow subsystem is to be retired when Generation cleanup is explicitly resumed. It is not Runtime IR and does not need to be converted into an ExecutionPlan. Registry versions, references and provider-specific validation remain generation-domain concerns.

Building JSON can be tested offline. Provider submission, real node/model compatibility and production qualification are different operations. Do not add a generic scheduler, inference bridge or personality service as prerequisites for the builder. Do not remove existing input/qualification protections as a shortcut either.

### ExecutionPlan

This controls inference execution, associated steps and their active state, including supported dependencies, waits, branching, interruption and resumption. Runtime retains its model-adjacent control points; it is not only a loop that repeatedly calls external APIs.

A future explicitly requested inference workload may use generation through a bounded internal Controller capability. That optional interaction is not a prerequisite for ordinary generation, reference images or Controller extraction.

## Gateway means replaceability, not MCP

Keep provider/runtime details behind narrow internal interfaces so implementations can be replaced. Do not equate this with routing through Intelligence MCP, introduce another central bus, or copy provider adapters into every application.

Reusable intelligence adapter ownership must be resolved by the Agent/Intelligence/Runtime design tasks against real code and callers. This decision does not create a new Intelligence Controller repository. Library versus service packaging, concrete endpoints and credential configuration remain explicit design questions, not assumptions in this map.

## One state owner through migration

Internal and MCP callers must use the same generation domain authority. Do not create two JobStores or independent resource reservations simply by giving each frontend its own Controller instance. Preserve job/input/asset IDs, immutable references, historical provenance, storage compatibility and uncertain accepted work.

Agent conversations and Runtime active Jobs are distinct from generation Jobs. A reference to another owner's work does not transfer its lifecycle or authorize access. Host-wide runtime transitions stay with GPU Node Manager. Runtime facts and generation qualification are different claims; lifecycle READY is not proof that a particular graph is qualified.

## Safeguards that remain requirements

Maintain authentication and scoped authorization, complete-context remote consent, bounded input/output/time/resources, safe staging and paths, declared output handling, exact identity and provenance, and no automatic replay after ambiguous acceptance. Retain existing automated verification and readiness protections until a separately reviewed change replaces them.

Static JSON validation, domain/provider fixture tests, external MCP mapping tests and live migration/qualification must have distinct evidence. Passing one does not establish all four.

## Current migration boundary

Generation logic is currently co-located in Generation MCP, and existing internal callers may still use MCP adapters. These are as-built facts to inventory, not the corrected target. Generation Controller stays documentation-only. The current Generation MCP ComfyWorkFlow implementation is not an extraction source: later Generation cleanup should delete it from the MCP side rather than move it. No deployed service, data or API changed in this documentation pass.

The [roadmap](ROADMAP.md) and #18 children track source/test inventory and the minimal internal contract. ComfyWorkFlows and reference-image feature development remain paused at the merged baseline. Documentation review/merge does not automatically authorize extraction, deployment or new features.

## 日本語

MCPは外部入口、Generation Controllerは生成domain、AI Agentは任意の人格、AI Runtimeは推論とExecutionPlan、GPU Node Managerはhostの起動停止を担当します。Studioは内部interfaceを利用します。

ComfyWorkFlowはComfyUI等へ渡すJSONの組み立てで、実行はproviderが行います。ExecutionPlanは推論を制御する別物です。前者を後者へ移管したり、単純なJSON生成に推論基盤を必須化したりしません。現在のコードと新設計を区別し、移設は別途承認後に行います。


## Current implementation priority

After this documentation correction, the first implementation work is the Intelligence boundary cleanup: remove internal Agent/Studio dependency on Intelligence MCP and keep MCP as the external ChatGPT-facing facade. Generation Controller implementation and Generation ComfyWorkFlow cleanup remain paused until a later explicit instruction.
