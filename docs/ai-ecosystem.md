# FLAMORIS AI Ecosystem

This is the repository responsibility map. [Architecture](ARCHITECTURE.md) defines the corrected target under [AI #18](https://github.com/flamoris-jp/flamoris-ai/issues/18); [roadmap](ROADMAP.md) records sequencing. The map is not evidence of deployed migration or feature readiness. Earlier maps are preserved in [pinned history](LEGACY_DESIGN.md).

## Generation and external access

| Repository | Intended role |
| --- | --- |
| [Generation Controller](https://github.com/flamoris-jp/flamoris-generation-controller) | Internal generation owner: providers/adapters, ComfyUI JSON builder, generation jobs, inputs/references and assets. Currently a design/extraction target. |
| [Generation MCP](https://github.com/flamoris-jp/flamoris-generation-mcp) | ChatGPT-facing MCP adapter over the generation domain. Current co-located domain code is to be extracted, not discarded. |
| [Intelligence MCP](https://github.com/flamoris-jp/flamoris-intelligence-mcp) | External MCP facade exposing internal intelligence capabilities; not Agent/Studio's internal provider gateway. |
| [MCP Hub](https://github.com/flamoris-jp/flamoris-mcp-hub) | External MCP aggregation, namespaced catalog, connection and routing. |

```text
ChatGPT -> MCP Hub -> Generation MCP -> Generation Controller -> provider
Studio ------------------------------> Generation Controller -> provider
```

The Controller owns **Generation Workflow construction**, including ComfyUI API-format JSON. ComfyUI owns graph execution. This is distinct from **AI Runtime inference Workflows**; a name shared by two concepts does not transfer ownership.

## Personality and inference

[AI Agent](https://github.com/flamoris-jp/flamoris-ai-agent) owns personality, identity, conversations, memory, principal/session and context policy. It participates when personality is requested, not as a mandatory step for every inference or generation operation. Its model choice is independent of personality identity.

[AI Runtime](https://github.com/flamoris-jp/flamoris-ai-runtime) owns model-adjacent inference and supported active state/control points, inference Workflow validation/compilation/execution, Jobs/Continuations, interrupts, resource accounting and execution events. It is neither the ComfyUI JSON builder nor the durable Agent-memory store.

```text
Studio raw inference -> internal runtime / API / vendor interface
Studio Agent Support -> AI Agent -> internal execution interface
```

Internal FLAMORIS calls do not use MCP/Hub. Ordinary API calls need not be forced through AI Runtime. A future inference-to-generation interaction is an optional internal capability, not a condition for the basic generation path. Existing external Agent MCP compatibility is reviewed independently from internal adapter migration.

## Models and specializations: unchanged by this correction

### Maidionis

[Maidionis](https://github.com/flamoris-jp/Maidionis) owns specialization-neutral model/training/evaluation/artifact machinery and bounded inference contracts: reusable model and training mechanisms, dataset/manifest contracts, education, checkpointing, reproducibility, serialization, calibration and specialization identity. It does not own inference Workflow scheduling, tool authorization, retry/fallback orchestration, product state or host lifecycle.

### Arbitrium

[Arbitrium](https://github.com/flamoris-jp/Arbitrium) is the first Maidionis Decision specialization. It owns Decision-specific TaskSpecs, curricula, experiments, measured evidence, failure analysis and specialization metadata for bounded advisory judgments. Shared machinery belongs to Maidionis; active execution belongs to Runtime. Actual quality and integration evidence remain in the owning repository.

### Oblivionis

[Oblivionis](https://github.com/flamoris-jp/Oblivionis) retains its independent name and experimental non-LLM model identity. Its intended path is experience -> evolving oscillatory state -> firing -> Runtime modulation -> changed behavior. This is a model concept, not a claim of deployed integration or biological fidelity.

Its planned authority includes Active Field dynamics, oscillation/resonance/coupling/fatigue/fluctuation, firing responses, forgetting, snapshots and association/recall through Profundumis. Runtime owns the bounded mapping, timing and application of modulation. The model is not just a memory lookup, independent noise generator or Workflow-start detector.

Firing or modulation of existing execution is distinct from triggering new work. Keep [sensor/trigger #3](https://github.com/flamoris-jp/Oblivionis/issues/3) and [latent-recall #4](https://github.com/flamoris-jp/Oblivionis/issues/4) independent. Max-state search and percentage reactivation apply after latent storage during association/recall, not ordinary firing thresholds or continuous restoration. Detailed semantics stay in the [owning model concept](https://github.com/flamoris-jp/Oblivionis/blob/main/docs/MODEL.md).

This architecture correction does not redesign these models, their native inference support or their research plans.

## Host and product boundaries

[GPU Node Manager](https://github.com/flamoris-jp/flamoris-gpu-node-manager) owns host-wide configured runtime/GPU lifecycle and transition coordination. CLI/HTTP/MCP may adapt the same manager. Internal callers use an internal non-MCP interface; external ChatGPT access may use the MCP surface through Hub. Do not duplicate systemd/GPU state machines in Controller, Runtime, Agent or Studio. Lifecycle readiness and per-generation-workflow qualification remain separate.

[Studio](https://github.com/flamoris-jp/flamoris-studio) owns its authenticated creative UI, drafts, account/session mappings and per-user access. It uses the internal Controller/runtime/API/Agent boundaries and does not inspect raw ComfyUI nodes or own provider execution. Asset/conversation references do not by themselves grant authorization.

Desktop products retain their own document/project state and editing semantics. Generic logging, MCP foundations, diagnostics and security primitives remain with [FLAMORIS Commons](https://github.com/flamoris-jp/flamoris-commons) or dedicated shared packages, not with an AI domain merely because multiple applications need them.

## 日本語

内部の生成制御はGeneration Controller、外部のMCP公開はGeneration MCP / Intelligence MCP、人格は必要な場合だけAI Agent、推論と推論WorkflowはAI Runtimeです。ComfyUI用Workflow JSONの生成はController側に残ります。内部の呼び出しにMCP Hubを使いません。

Maidionis・Arbitrium・Oblivionisのモデル責務、GPU Node Managerのhost lifecycle、各アプリの制作データ所有権は維持します。ここは修正後の責務地図であり、移設や実機検証が完了したという記録ではありません。

See the [organization map](https://github.com/flamoris-jp/.github) and [desktop ecosystem](https://github.com/flamoris-jp/flamoris-commons/blob/main/docs/desktop-ecosystem.md) for adjacent projects. Release status, test results and live readiness remain with each owner.
