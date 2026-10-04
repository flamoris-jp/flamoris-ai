# FLAMORIS AI

Architecture, responsibility boundaries and roadmap coordination for the AI-facing parts of FLAMORIS. This repository does not implement the services it coordinates.

## Current direction

[Issue #18](https://github.com/flamoris-jp/flamoris-ai/issues/18) defines the corrected target. Documentation review, fixes and explicitly authorized documentation merges are the current work. They do not migrate code or deployments, start Work implementation, or resume Generation development.

**Intelligence cleanup comes first. Generation Controller remains unimplemented. The existing Generation MCP ComfyWorkFlow subsystem is marked for later removal, not transfer into Controller.** Its deletion scope and affected callers/data must be inventoried before a separately authorized code change; this is not permission to delete retained assets, definitions or evidence from a running installation.

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

These are target boundaries, not a claim of deployed routes. Internal FLAMORIS components do not call one another through MCP or MCP Hub. A replaceable gateway is an internal interface, not a synonym for an MCP server or an additional central service.

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

次の実装対象はIntelligence関連の余計な内部MCP経路の整理です。Generation Controllerはまだ実装しません。Generation MCPの既存ComfyWorkFlow実装は移植ではなく後の削除対象ですが、今回は文書レビュー・修正・マージまでで、コード削除や実機変更は行いません。

## FLAMORIS and license

FLAMORIS is open-source software for creative work and AI-native production. Commercial use of the licensed code is welcome without individual permission. Software is provided as-is, without guaranteed individual support. Documentation, Issues, tests and source are the primary self-support references.

Code and documentation are licensed under [Apache License 2.0](LICENSE), unless otherwise noted. Models, weights, datasets, prompts, provider-hosted assets, generated media and other non-code materials may have separate terms.
