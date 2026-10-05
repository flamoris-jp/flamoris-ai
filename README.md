# FLAMORIS AI

Architecture, responsibility boundaries and roadmap coordination for the AI-facing parts of FLAMORIS. This repository does not implement the services it coordinates.

## Current direction

Start with [PROGRESS.md](PROGRESS.md) for the cross-thread handoff, detailed phases,
accepted source baselines and next steps. It distinguishes completed source work,
pending live acceptance and held Controller work; architecture remains defined below.

[Issue #18](https://github.com/flamoris-jp/flamoris-ai/issues/18) defines the corrected target and records source acceptance. The cleanup removes obsolete internal MCP clients and the mistaken generation registry/composition/Runtime bridge, and implements the retained internal Intelligence connections. Source acceptance and deployed routes remain separate.

**Intelligence cleanup comes first. Generation Controller remains unimplemented. Retire the added MCP-side ComfyWorkFlow registry, versioning, composition and Runtime delegation; do not transfer or recreate them in Controller.** Preserve the original bounded image-generation templates, generic provider recipes, jobs/assets/inputs and uncertain-request fences. Source retirement does not delete retained data or change a running installation.

Read the [architecture](docs/ARCHITECTURE.md), [ecosystem map](docs/ai-ecosystem.md), [roadmap](docs/ROADMAP.md) and [Studio boundaries](docs/MULTIMODAL_STUDIO_ARCHITECTURE.md). Earlier designs are preserved in [pinned history](docs/LEGACY_DESIGN.md), not as current implementation instructions.

## Target access paths

```text
External:
ChatGPT -> MCP Hub -> Generation MCP -> future Generation Controller -> providers
                  -> Intelligence MCP -> internal intelligence capabilities

Internal:
Studio generation -------> future Generation Controller -> generation providers
Studio raw intelligence -> internal runtime / API / vendor interface
Studio Agent Support ----> AI Agent -> internal execution interface
```

These are target boundaries, not a claim of deployed routes. Internal Intelligence/Agent paths now use non-MCP contracts. Generation still uses its retained MCP compatibility path until separately authorized Controller work. A replaceable gateway is an internal interface, not a synonym for an MCP server or an additional central service.

## Implemented source boundaries

| Source change | Owning PR / contract |
| --- | --- |
| Shared `flamoris_intelligence` provider library, with MCP as an optional external extra | [Intelligence #12](https://github.com/flamoris-jp/flamoris-intelligence-mcp/pull/12) |
| Agent JSON HTTP `/api/v1` and approved non-MCP execution client; external Agent MCP retained | [Agent #40](https://github.com/flamoris-jp/flamoris-ai-agent/pull/40) |
| Studio raw provider adapter, Agent HTTP gateway and retired custom Image dispatch | [Studio #63](https://github.com/flamoris-jp/flamoris-studio/pull/63) |
| Custom registration/versioning/v3/qualification/Runtime delegation removed; builtin/native generation and data protections retained | [Generation #69](https://github.com/flamoris-jp/flamoris-generation-mcp/pull/69) |
| Generation-specific lowering removed; native ExecuteFlow/ExecutionPlan retained | [Runtime #25](https://github.com/flamoris-jp/flamoris-ai-runtime/pull/25) |
| Retired custom/v3 catalog removed; external routing remains | [Hub #37](https://github.com/flamoris-jp/flamoris-mcp-hub/pull/37) |

The linked PRs and #18 record exact review/CI/merge evidence. Studio's contract
tests connect its real gateway to the real Agent HTTP service/session and shared
provider adapter with synthetic state and HTTP. Live providers and deployment
were not exercised. Generation Controller remains unimplemented, and no stored
data, grants or credentials are migrated by this cleanup.

## Names and responsibilities

| Term | Meaning | Owner |
| --- | --- | --- |
| `ExecuteFlow` | Inference dependency/data/control flow | AI Runtime |
| `ExecutionPlan` | Existing compiled Runtime representation; not renamed to ExecuteFlow | AI Runtime |
| `ComfyWorkFlow` | ComfyUI execution graph / API-format JSON and its declared bindings | Generation domain; ComfyUI executes the graph |

Use the specific names, not bare `Workflow`, in new FLAMORIS design prose. Existing code, wire names and historical quotations retain their literal spelling until an explicit compatibility-reviewed change. Non-ComfyUI provider requests are generation requests/recipes, not automatically ComfyWorkFlow.

## Repository map

| Repository | Target role |
| --- | --- |
| [Generation Controller](https://github.com/flamoris-jp/flamoris-generation-controller) | Future internal generation-domain boundary; documentation only, not an implementation task now |
| [Generation MCP](https://github.com/flamoris-jp/flamoris-generation-mcp) | External generation MCP adapter; current co-located domain code is an audit baseline, not a blanket migration source |
| [Intelligence MCP](https://github.com/flamoris-jp/flamoris-intelligence-mcp) | External intelligence MCP facade, not Agent/Studio's internal execution gateway |
| [AI Agent](https://github.com/flamoris-jp/flamoris-ai-agent) | Optional personality, conversations, memory, principal/session and context policy |
| [AI Runtime](https://github.com/flamoris-jp/flamoris-ai-runtime) | Model-adjacent inference, ExecuteFlow, compiled ExecutionPlan, active jobs/state and resources |
| [MCP Hub](https://github.com/flamoris-jp/flamoris-mcp-hub) | External MCP catalog, connections and routing |
| [GPU Node Manager](https://github.com/flamoris-jp/flamoris-gpu-node-manager) | Host-wide configured runtime/GPU lifecycle authority |
| [Studio](https://github.com/flamoris-jp/flamoris-studio) | Authenticated creative UI, drafts and user-scoped access |

Maidionis, Arbitrium and Oblivionis retain their independent model/training/specialization responsibilities; see the [ecosystem map](docs/ai-ecosystem.md). This correction is not a native-kernel or model redesign.

Preserve authorization, complete-context remote consent, bounded I/O, input/reference safety, provenance and uncertain-request/no-replay guarantees. Removing obsolete code does not authorize deleting user data or bypassing protections on retained paths. Concrete internal APIs and code-removal inventories belong in the owning Issues. No new Intelligence Controller repository or deployment topology is implied.

Follow the [repository policy](https://github.com/flamoris-jp/flamoris-commons/blob/main/docs/repository-policy.md) and [organization map](https://github.com/flamoris-jp/.github). Exact implementation and live acceptance remain with each owner.

## 日本語

MCPはChatGPT側の外部入口で、内部通信には使いません。AI Agentは人格が必要なときだけ使います。Runtimeの推論フローは `ExecuteFlow`、既存のコンパイル済み内部表現は `ExecutionPlan`、ComfyUIのグラフ・JSONは `ComfyWorkFlow` と区別します。

Intelligenceの内部MCP経路を直接接続へ置き換え、後付けのComfyWorkFlow登録・版管理・合成・Runtime委譲を削除したソースと検証記録は上記PRにまとまっています。基本生成と保存済みデータの保護は維持します。Generationの接続は互換経路を残しており、Generation Controllerは未実装のままです。実機移行は別工程です。

## FLAMORIS and license

FLAMORIS is open-source software for creative work and AI-native production. Commercial use of the licensed code is welcome without individual permission. Software is provided as-is, without guaranteed individual support. Documentation, Issues, tests and source are the primary self-support references.

Code and documentation are licensed under [Apache License 2.0](LICENSE), unless otherwise noted. Models, weights, datasets, prompts, provider-hosted assets, generated media and other non-code materials may have separate terms.
