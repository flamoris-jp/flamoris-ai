# FLAMORIS AI Ecosystem

This maps current responsibilities and explicitly marked future scope. [Architecture](ARCHITECTURE.md) defines boundaries and terminology under [AI #18](https://github.com/flamoris-jp/flamoris-ai/issues/18); [roadmap](ROADMAP.md) lists remaining work. Source and live acceptance are recorded separately in [PROGRESS.md](../PROGRESS.md). Earlier maps remain in [pinned history](LEGACY_DESIGN.md).

## External access and generation

| Repository | Responsibility / status |
| --- | --- |
| [Generation Controller](https://github.com/flamoris-jp/flamoris-generation-controller) | MCP-free shared generation authority and HTTP adapter; implementation PRs, live cutover pending |
| [Generation MCP](https://github.com/flamoris-jp/flamoris-generation-mcp) | External tools/ingress/content facade hosting one Controller and its internal HTTP adapter in the matched open PRs |
| [Intelligence MCP](https://github.com/flamoris-jp/flamoris-intelligence-mcp) | External facade and home of a separate importable non-MCP provider-adapter package; not Agent/Studio's internal network gateway |
| [MCP Hub](https://github.com/flamoris-jp/flamoris-mcp-hub) | External aggregation, namespaces, connections and routing |

External generation clients use MCP Hub → Generation MCP facade → Controller. Studio uses authenticated Controller HTTP directly; both adapters share one runtime/reservation. The matched [Controller #5](https://github.com/flamoris-jp/flamoris-generation-controller/pull/5), [Generation #71](https://github.com/flamoris-jp/flamoris-generation-mcp/pull/71) and [Studio #65](https://github.com/flamoris-jp/flamoris-studio/pull/65) source PRs are open/unmerged. Their [implementation policy](https://github.com/flamoris-jp/flamoris-generation-controller/blob/640a5bd48c76e4bf736e9a3589c216123ccd18b3/docs/IMPLEMENTATION.md) and [API](https://github.com/flamoris-jp/flamoris-generation-controller/blob/640a5bd48c76e4bf736e9a3589c216123ccd18b3/docs/API.md) define hosting, service permission and state continuity. Live cutover remains pending.

ComfyWorkFlow specifically means ComfyUI graph/API-format JSON. The generation provider adapter constructs the bounded builtin graphs; ComfyUI owns graph execution. Other providers consume their declared generation requests/recipes.

## Personality and inference

[AI Agent](https://github.com/flamoris-jp/flamoris-ai-agent) owns optional personality, identity, conversation/memory, principal/session and context policy. Model selection is independent of personality identity. Ordinary inference and generation do not require Agent.

[AI Runtime](https://github.com/flamoris-jp/flamoris-ai-runtime) owns model-adjacent inference and supported control points, ExecuteFlow, its compiled ExecutionPlan representation, Jobs/Continuations, interrupts, resource accounting and events. It is not the ComfyUI JSON builder or durable Agent-memory owner.

Studio raw inference and Agent execution use the shared neutral provider adapters directly. Studio Agent Support uses Agent JSON HTTP `/api/v1`. These are implemented non-MCP contracts; approved runtime/API/vendor targets remain behind the provider boundary. An ordinary API request need not use AI Runtime. Future inference-to-generation interaction remains an optional capability. External Agent MCP is retained independently.

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

[GPU Node Manager](https://github.com/flamoris-jp/flamoris-gpu-node-manager) owns host-wide configured runtime/GPU lifecycle. CLI/HTTP/MCP may adapt one manager. Internal callers use non-MCP interfaces; external callers may use its MCP surface through Hub. No Controller, Runtime, Agent or Studio duplicate of the host state machine is introduced. Host readiness and acceptance of a selected generation provider are separate checks.

[Studio](https://github.com/flamoris-jp/flamoris-studio) owns authenticated UI, drafts, account/session mappings and per-user access. Its Intelligence/Agent gateways implement non-MCP contracts; the matched open Generation PR replaces upstream MCP with authenticated Controller HTTP. Asset/conversation IDs alone grant no authorization. The [source matrix](../README.md#implemented-source-boundaries) links exact evidence without claiming merged or deployed generation routes.

Desktop products retain their own project/document/editing state. Logging, MCP foundations, diagnostics and generic security primitives remain with [FLAMORIS Commons](https://github.com/flamoris-jp/flamoris-commons) or dedicated packages, not an AI domain merely because several callers need them.

## 日本語

ExecuteFlowはRuntimeの推論フロー、ExecutionPlanはコンパイル済み表現、ComfyWorkFlowはComfyUI用グラフ・JSONです。MCPは外部入口、Agentは人格が必要な場合のみ。内部Intelligence／Agentは共通provider adapterとAgent HTTPに接続します。保持された生成機能をControllerへ切り出し、Studioの認証付きHTTPと外部MCPが同じ生成状態を使うソースを実装しました。対応PRは未マージで、実機cutoverは別工程です。

See the [organization map](https://github.com/flamoris-jp/.github) and [desktop ecosystem](https://github.com/flamoris-jp/flamoris-commons/blob/main/docs/desktop-ecosystem.md). Exact status, test results and live readiness remain with each owner.
