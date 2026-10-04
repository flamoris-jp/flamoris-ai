# FLAMORIS AI Ecosystem

This is a target responsibility map, not deployment evidence. [Architecture](ARCHITECTURE.md) defines the boundaries and terminology under [AI #18](https://github.com/flamoris-jp/flamoris-ai/issues/18); [roadmap](ROADMAP.md) defines Intelligence-first sequencing. Earlier maps remain in [pinned history](LEGACY_DESIGN.md).

## External access and generation

| Repository | Target role |
| --- | --- |
| [Generation Controller](https://github.com/flamoris-jp/flamoris-generation-controller) | Future internal generation boundary; documentation only, no implementation now |
| [Generation MCP](https://github.com/flamoris-jp/flamoris-generation-mcp) | External MCP adapter; its existing ComfyWorkFlow subsystem is a later removal candidate, not code to migrate into Controller |
| [Intelligence MCP](https://github.com/flamoris-jp/flamoris-intelligence-mcp) | External intelligence facade, not Agent/Studio's internal provider gateway |
| [MCP Hub](https://github.com/flamoris-jp/flamoris-mcp-hub) | External aggregation, namespaces, connections and routing |

```text
ChatGPT -> MCP Hub -> Generation MCP -> future Generation Controller -> provider
Studio ------------------------------> future Generation Controller -> provider
```

ComfyWorkFlow specifically means ComfyUI graph/API-format JSON. Other providers may consume generation requests/recipes without any ComfyWorkFlow. ComfyUI owns graph execution. The existing builder is not moved into AI Runtime or Controller in this correction. Neither Controller implementation nor recreation of the old subsystem is authorized.

## Personality and inference

[AI Agent](https://github.com/flamoris-jp/flamoris-ai-agent) owns optional personality, identity, conversation/memory, principal/session and context policy. Model selection is independent of personality identity. Ordinary inference and generation do not require Agent.

[AI Runtime](https://github.com/flamoris-jp/flamoris-ai-runtime) owns model-adjacent inference and supported control points, ExecuteFlow, its compiled ExecutionPlan representation, Jobs/Continuations, interrupts, resource accounting and events. It is not the ComfyUI JSON builder or durable Agent-memory owner.

```text
Studio raw inference -> internal runtime / API / vendor interface
Studio Agent Support -> AI Agent -> internal execution interface
```

Internal FLAMORIS calls do not use MCP/Hub. An ordinary API request need not use AI Runtime. Future inference-to-generation interaction is an optional internal capability, not a prerequisite for generation or Intelligence cleanup. Existing external Agent MCP compatibility is reviewed independently.

## Independent model boundaries

### Maidionis

[Maidionis](https://github.com/flamoris-jp/Maidionis) owns specialization-neutral model/training/evaluation/artifact machinery and bounded inference contracts, including datasets/manifests, education, checkpointing, reproducibility, serialization, calibration and specialization identity. It does not own ExecuteFlow scheduling, tool authorization, retry/fallback orchestration, product state or host lifecycle.

### Arbitrium

[Arbitrium](https://github.com/flamoris-jp/Arbitrium) is the first Maidionis Decision specialization. It owns Decision-specific TaskSpecs, curricula, experiments, evidence, failure analysis and specialization metadata. Shared machinery belongs to Maidionis; active execution belongs to Runtime. Actual quality and integration evidence remain with each owner.

### Oblivionis

[Oblivionis](https://github.com/flamoris-jp/Oblivionis) retains its independent name and experimental non-LLM identity. Its intended path is experience -> evolving oscillatory state -> firing -> Runtime modulation -> changed behavior. This is a model concept, not a claim of deployed integration or biological fidelity.

Its model authority includes Active Field dynamics, oscillation/resonance/coupling/fatigue/fluctuation, firing, forgetting, snapshots and association/recall through Profundumis. Runtime owns the bounded mapping, timing and application of modulation. The model is not just a memory lookup, independent noise generator or ExecuteFlow-start detector.

Modulating existing execution and triggering new work are distinct. Keep [sensor/trigger #3](https://github.com/flamoris-jp/Oblivionis/issues/3) and [latent-recall #4](https://github.com/flamoris-jp/Oblivionis/issues/4) independent. Max-state search and percentage reactivation apply after latent storage during association/recall, not normal firing thresholds or continuous restoration. Detailed semantics remain in the [model concept](https://github.com/flamoris-jp/Oblivionis/blob/main/docs/MODEL.md). This correction does not redesign these models or native inference.

## Host, product and common infrastructure

[GPU Node Manager](https://github.com/flamoris-jp/flamoris-gpu-node-manager) owns host-wide configured runtime/GPU lifecycle. CLI/HTTP/MCP may adapt one manager. Internal callers use non-MCP interfaces; external callers may use its MCP surface through Hub. No Controller, Runtime, Agent or Studio duplicate of the host state machine is introduced. Lifecycle readiness differs from per-ComfyWorkFlow qualification.

[Studio](https://github.com/flamoris-jp/flamoris-studio) owns authenticated UI, drafts, account/session mappings and per-user access. The diagram describes future internal contracts, not an already migrated gateway. Asset/conversation IDs alone grant no authorization.

Desktop products retain their own project/document/editing state. Logging, MCP foundations, diagnostics and generic security primitives remain with [FLAMORIS Commons](https://github.com/flamoris-jp/flamoris-commons) or dedicated packages, not an AI domain merely because several callers need them.

## 日本語

ExecuteFlowはRuntimeの推論フロー、ExecutionPlanは既存のコンパイル済み表現、ComfyWorkFlowはComfyUI用グラフ・JSONです。MCPは外部入口、Agentは人格が必要な場合のみ。Intelligence整理を先行し、Controllerは未実装のままにします。Generation MCPの既存ComfyWorkFlowコードは後の削除対象で、移植や再実装を今進めません。

See the [organization map](https://github.com/flamoris-jp/.github) and [desktop ecosystem](https://github.com/flamoris-jp/flamoris-commons/blob/main/docs/desktop-ecosystem.md). Exact status, test results and live readiness remain with each owner.
