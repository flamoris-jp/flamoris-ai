# FLAMORIS AI

Architecture, responsibility boundaries and roadmap coordination for the AI-facing parts of FLAMORIS. This repository does not implement the services it coordinates.

## Architecture correction

[Issue #18](https://github.com/flamoris-jp/flamoris-ai/issues/18) records the corrected target architecture. **The current stage is documentation and Issue organization in Chat only.** Existing implementations and deployments have not been migrated by these documents. ComfyWorkFlow/reference-image development remains paused; merging documentation does not authorize implementation or deployment.

Start with the [architecture](docs/ARCHITECTURE.md), [repository map](docs/ai-ecosystem.md), [roadmap](docs/ROADMAP.md) and [Studio integration boundaries](docs/MULTIMODAL_STUDIO_ARCHITECTURE.md). Earlier designs remain available as [pinned historical records](docs/LEGACY_DESIGN.md); their internal-MCP dependency diagrams are not the target architecture.

## The essential separation

```text
External MCP access:
ChatGPT -> MCP Hub -> Generation MCP -> Generation Controller -> providers
                  -> Intelligence MCP -> internal intelligence capabilities

Internal application access:
Studio generation -------> Generation Controller -> generation providers
Studio raw intelligence -> internal runtime / API / vendor interface
Studio Agent Support ----> AI Agent -> internal execution interface
```

Internal FLAMORIS components do not call one another through MCP or MCP Hub. A replaceable gateway is an internal interface, not a synonym for an MCP server or a new central service.

## Two named execution concepts, two owners

| Term | Meaning | Owner |
| --- | --- | --- |
| ComfyWorkFlow | Build a provider execution definition, especially ComfyUI API-format workflow JSON with declared parameter/reference bindings | Generation Controller; ComfyUI executes the graph |
| ExecutionPlan | Control inference execution and its active steps/state | AI Runtime |

**Do not move the ComfyWorkFlow Builder into AI Runtime.** JSON construction does not require an Agent, ExecutionPlan engine, MCP Hub or GPU inference. Actual provider submission and production qualification are separate operations.

## Repository responsibilities

| Repository | Role |
| --- | --- |
| [flamoris-generation-controller](https://github.com/flamoris-jp/flamoris-generation-controller) | Internal generation domain: provider adapters, ComfyUI workflow construction, generation jobs, inputs/references and assets. The repository is documentation-only. The existing Generation MCP ComfyWorkFlow implementation is not planned for code transfer; when Generation work resumes it should be retired from the MCP side, and any future Controller implementation must be designed separately. |
| [flamoris-generation-mcp](https://github.com/flamoris-jp/flamoris-generation-mcp) | External MCP adapter for generation capabilities; existing co-located domain code is the extraction baseline. |
| [flamoris-intelligence-mcp](https://github.com/flamoris-jp/flamoris-intelligence-mcp) | External MCP facade for intelligence capabilities, not the internal execution gateway for Agent or Studio. |
| [flamoris-ai-agent](https://github.com/flamoris-jp/flamoris-ai-agent) | Optional personality, conversations, memory, principal/session and context policy. |
| [flamoris-ai-runtime](https://github.com/flamoris-jp/flamoris-ai-runtime) | Model-adjacent inference Runtime with ExecutionPlans, active jobs/state, control points and resources. |
| [flamoris-mcp-hub](https://github.com/flamoris-jp/flamoris-mcp-hub) | External MCP aggregation, catalog, connection and routing boundary. |
| [flamoris-gpu-node-manager](https://github.com/flamoris-jp/flamoris-gpu-node-manager) | Host-wide configured runtime/GPU lifecycle authority; its CLI/HTTP/MCP adapters share that authority. |
| [flamoris-studio](https://github.com/flamoris-jp/flamoris-studio) | Authenticated creative UI, product drafts and user-scoped access to internal capabilities. |

[Maidionis](https://github.com/flamoris-jp/Maidionis), [Arbitrium](https://github.com/flamoris-jp/Arbitrium) and [Oblivionis](https://github.com/flamoris-jp/Oblivionis) retain their independent model/training/specialization responsibilities. Their boundaries are summarized in the [ecosystem map](docs/ai-ecosystem.md); this correction is not a model or native-kernel redesign.

## Change discipline

Preserve useful tests, identities, data and evidence, but do not preserve misplaced code merely to avoid deletion. In particular, the current Generation MCP ComfyWorkFlow implementation is not a migration source for Controller code. Separate architecture cleanup from new features. Keep authorization, bounded inputs/outputs, automated qualification and uncertain-request/no-replay protections. Do not duplicate a generation store or host lifecycle merely because there are two frontends.

Exact internal APIs, package/service layout and migration details belong in the owning child Issues. Do not invent deployment commands, endpoints or ports in the architecture map. No new Intelligence Controller repository is implied.

Follow the [FLAMORIS Repository Policy](https://github.com/flamoris-jp/flamoris-commons/blob/main/docs/repository-policy.md) and [organization map](https://github.com/flamoris-jp/.github). Feature completion and live readiness remain authoritative in the owning repository, not in this map.

## 日本語

FLAMORIS AIはAI関連の責務と依存方向を整理する管制塔です。現在は[#18](https://github.com/flamoris-jp/flamoris-ai/issues/18)に基づく文書・Issue整理の段階で、実装や実機の移行はしていません。Generation Controllerもまだ実装しません。次の実装優先はIntelligence関連の内部MCP依存整理です。

**MCPはChatGPT側の外部入口。内部通信には使いません。** 生成domainはGeneration Controller、人格が必要なときだけAI Agent、推論とExecutionPlanはAI Runtime、hostのGPU/runtime起動停止はGPU Node Managerが担当します。

**ComfyUI系は `ComfyWorkFlow`、AI Runtime側は `ExecutionPlan` と呼びます。単独の `Workflow` は使いません。** 同じ名前でも別物であり、ComfyUIのbuilderをAI Runtimeへ移しません。マージ済みの成果や安全対策は残し、責務の分離と機能追加を別に扱います。

## FLAMORIS and license

FLAMORIS is open-source software for creative work and AI-native production. Commercial use of the licensed code is welcome without individual permission. Software is provided as-is, without guaranteed individual support. Documentation, Issues, tests and source are the primary self-support references.

Code and documentation are licensed under [Apache License 2.0](LICENSE), unless otherwise noted. Models, weights, datasets, prompts, provider-hosted assets, generated media and other non-code materials may have separate terms.
