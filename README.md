# FLAMORIS AI

Architecture, responsibility boundaries and roadmap coordination for the AI-facing parts of FLAMORIS. This repository does not implement the services it coordinates.

## Current direction

Start with [PROGRESS.md](PROGRESS.md) for the cross-thread handoff, detailed phases,
accepted source baselines and next steps. It distinguishes completed source work,
pending live acceptance and open Controller implementation PRs; architecture remains defined below.

[Issue #18](https://github.com/flamoris-jp/flamoris-ai/issues/18) coordinates the architecture and accepted source boundaries. Deployment and live acceptance are recorded separately in PROGRESS.md.

**Studio Intelligence, Agent Support and generation use non-MCP internal contracts in the matched source. Controller implementation PRs are open; merge/live acceptance remains separate.** Bounded builtin/native generation, generic provider recipes, jobs/assets/inputs and uncertain-request protections remain supported by their existing owners.

Read the [architecture](docs/ARCHITECTURE.md), [ecosystem map](docs/ai-ecosystem.md), [roadmap](docs/ROADMAP.md) and [Studio boundaries](docs/MULTIMODAL_STUDIO_ARCHITECTURE.md). Earlier designs are preserved in [pinned history](docs/LEGACY_DESIGN.md), not as current implementation instructions.

## Current source access paths

| Caller / capability | Implemented path |
| --- | --- |
| External client such as ChatGPT: generation | MCP Hub → Generation MCP facade → shared Controller → providers |
| External client such as ChatGPT: Intelligence | MCP Hub → Intelligence MCP facade → shared provider adapters |
| Studio: Image, Speech and Music | Authenticated Controller HTTP v1 → same Controller → providers |
| Studio: raw Intelligence | Shared `flamoris_intelligence` provider adapters → approved providers |
| Studio: Agent Support | Agent JSON HTTP `/api/v1` → shared provider adapters → approved providers |

The generation rows describe the matched implementation in [Controller #5](https://github.com/flamoris-jp/flamoris-generation-controller/pull/5), [Generation #71](https://github.com/flamoris-jp/flamoris-generation-mcp/pull/71) and [Studio #65](https://github.com/flamoris-jp/flamoris-studio/pull/65), currently open and unmerged. Controller package `640a5bd48c76e4bf736e9a3589c216123ccd18b3` is pinned by both dependents. One runtime is hosted by the external facade; Studio calls authenticated POST `/api/v1/generation/{operation}` directly. Configuration/grants and live provider acceptance still determine usable features. External Agent MCP remains separate. See [implementation direction](docs/ARCHITECTURE.md#generation-controller-implementation-direction), [retained-source plan](https://github.com/flamoris-jp/flamoris-generation-controller/blob/640a5bd48c76e4bf736e9a3589c216123ccd18b3/docs/IMPLEMENTATION.md) and [API](https://github.com/flamoris-jp/flamoris-generation-controller/blob/640a5bd48c76e4bf736e9a3589c216123ccd18b3/docs/API.md).

## Implemented source boundaries

| Implemented contract | Owning PR / contract |
| --- | --- |
| Shared `flamoris_intelligence` provider library, with MCP as an optional external extra | [Intelligence #12](https://github.com/flamoris-jp/flamoris-intelligence-mcp/pull/12) |
| Agent JSON HTTP `/api/v1` and approved non-MCP execution client; external Agent MCP retained | [Agent #40](https://github.com/flamoris-jp/flamoris-ai-agent/pull/40) |
| Studio raw provider adapter and Agent HTTP gateway | [Studio #63](https://github.com/flamoris-jp/flamoris-studio/pull/63) |
| Bounded builtin/native generation, generic recipes and retained-data protections | [Generation #69](https://github.com/flamoris-jp/flamoris-generation-mcp/pull/69) |
| Shared Controller, external facade and direct Studio HTTP (open/unmerged) | [Controller #5](https://github.com/flamoris-jp/flamoris-generation-controller/pull/5), [Generation #71](https://github.com/flamoris-jp/flamoris-generation-mcp/pull/71), [Studio #65](https://github.com/flamoris-jp/flamoris-studio/pull/65) |
| Native ExecuteFlow and compiled ExecutionPlan | [Runtime #25](https://github.com/flamoris-jp/flamoris-ai-runtime/pull/25) |
| External catalog and namespaced routing | [Hub #37](https://github.com/flamoris-jp/flamoris-mcp-hub/pull/37) |

The linked PRs and #18 record exact review/CI/merge evidence. Studio's contract
tests connect its real gateway to the real Agent HTTP service/session and shared
provider adapter with synthetic state and HTTP. Live providers and deployment
were not exercised. The [follow-up review](docs/REVIEW_2026-10-05.md) records
accepted residual fixes and organization-documentation updates separately from
these original baselines; PROGRESS.md links the accepted merge commits.

## Names and responsibilities

| Term | Meaning | Owner |
| --- | --- | --- |
| `ExecuteFlow` | Inference dependency/data/control flow | AI Runtime |
| `ExecutionPlan` | Existing compiled Runtime representation; not renamed to ExecuteFlow | AI Runtime |
| `ComfyWorkFlow` | ComfyUI execution graph / API-format JSON | Generation domain; ComfyUI executes the graph |

Use the specific names, not bare `Workflow`, in new FLAMORIS design prose. Existing code, wire names and historical quotations retain their literal spelling until an explicit compatibility-reviewed change. Non-ComfyUI provider requests are generation requests/recipes, not automatically ComfyWorkFlow.

## Repository map

| Repository | Responsibility / status |
| --- | --- |
| [Generation Controller](https://github.com/flamoris-jp/flamoris-generation-controller) | MCP-free retained generation domain and authenticated HTTP adapter; matched implementation PRs, live cutover pending |
| [Generation MCP](https://github.com/flamoris-jp/flamoris-generation-mcp) | External MCP facade hosting one Controller and its internal HTTP adapter |
| [Intelligence MCP](https://github.com/flamoris-jp/flamoris-intelligence-mcp) | Shared non-MCP provider adapters and an optional external MCP facade |
| [AI Agent](https://github.com/flamoris-jp/flamoris-ai-agent) | Optional personality, conversations, memory, principal/session and context policy |
| [AI Runtime](https://github.com/flamoris-jp/flamoris-ai-runtime) | Model-adjacent inference, ExecuteFlow, compiled ExecutionPlan, active jobs/state and resources |
| [MCP Hub](https://github.com/flamoris-jp/flamoris-mcp-hub) | External MCP catalog, connections and routing |
| [GPU Node Manager](https://github.com/flamoris-jp/flamoris-gpu-node-manager) | Host-wide configured runtime/GPU lifecycle authority |
| [Studio](https://github.com/flamoris-jp/flamoris-studio) | Authenticated creative UI, drafts and user-scoped access |

Maidionis, Arbitrium and Oblivionis retain their independent model/training/specialization responsibilities; see the [ecosystem map](docs/ai-ecosystem.md).

Preserve authorization, complete-context remote consent, bounded I/O, input/reference safety, provenance and uncertain-request/no-replay guarantees. Removing obsolete code does not authorize deleting user data or bypassing protections on retained paths. Concrete internal APIs and code-removal inventories belong in the owning Issues. No new Intelligence Controller repository or deployment topology is implied.

Follow the [repository policy](https://github.com/flamoris-jp/flamoris-commons/blob/main/docs/repository-policy.md) and [organization map](https://github.com/flamoris-jp/.github). Exact implementation and live acceptance remain with each owner.

## 日本語

MCPはChatGPT側の外部入口です。Studioのraw IntelligenceとAgent内部の実行は共通provider adapterへ直接接続し、StudioのAgent SupportはAgent JSON HTTPを使います。2026-10-05の実装指示で、Generation Controllerと対応するMCP・Studio接続を実装しました。Studioは認証付きHTTP、外部MCPは同じControllerへ接続します。実装PRは未マージで、実機反映も別途必要です。AI Agentは人格が必要なときだけ使います。

Runtimeの推論フローは `ExecuteFlow`、コンパイル済み内部表現は `ExecutionPlan`、ComfyUIのグラフ・JSONは `ComfyWorkFlow` と区別します。保持された生成機能と保存データの保護は既存ownerが担当します。ソース受け入れ・追加レビューPR・実機受け入れは [PROGRESS.md](PROGRESS.md) で個別に記録します。

## FLAMORIS and license

FLAMORIS is open-source software for creative work and AI-native production. Commercial use of the licensed code is welcome without individual permission. Software is provided as-is, without guaranteed individual support. Documentation, Issues, tests and source are the primary self-support references.

Code and documentation are licensed under [Apache License 2.0](LICENSE), unless otherwise noted. Models, weights, datasets, prompts, provider-hosted assets, generated media and other non-code materials may have separate terms.
