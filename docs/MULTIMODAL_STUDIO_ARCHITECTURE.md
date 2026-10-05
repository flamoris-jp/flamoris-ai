# Studio integration boundaries

Corrected target under [AI #18](https://github.com/flamoris-jp/flamoris-ai/issues/18), 2026-10-04. This is active design guidance, not deployed behavior. The earlier #15 proposal remains in [pinned history](LEGACY_DESIGN.md); its internal-MCP routes and compulsory bridge sequence are superseded.

## Three separate internal paths

```text
Studio generation -------> future Generation Controller -> generation providers
Studio raw intelligence -> internal runtime / API / vendor interface
Studio Agent Support ----> AI Agent -> internal execution interface
```

Studio's raw Intelligence and Agent Support now use replaceable non-MCP gateways.
Generation still uses the retained MCP compatibility route while its Controller
target is deferred. Studio does not acquire provider graph internals, inference
scheduling or host lifecycle. Ordinary generation and inference need no Agent.

External access is separate: ChatGPT -> MCP Hub -> Generation MCP / Intelligence MCP -> the appropriate internal capability. The future Controller is not running or being implemented as part of this pass.

## Terminology and scope

ComfyWorkFlow is a ComfyUI execution graph/API-format JSON. Its construction is not ExecuteFlow or compiled ExecutionPlan. Non-ComfyUI generation providers use their own declared request/recipe contracts; do not rename all media requests ComfyWorkFlow.

Studio selects authorized generation metadata and supplies declared values; it does not parse raw ComfyUI nodes or compile them into Runtime IR. The added MCP-side ComfyWorkFlow registry/composition/delegation is retired, without transfer into Controller. Retired selections become explicitly unavailable. Original basic generation and generic provider recipes remain until a later Controller integration; no replacement builder or generation UI expansion is started now.

ExecuteFlow controls inference; ExecutionPlan remains Runtime's compiled representation. Optional future inference-to-generation interaction is not a prerequisite for ordinary generation, reference images or Intelligence cleanup.

## Valid product requirements

Retain dedicated editors, supported-parameter discovery, metadata-first catalogs, bounded previews/downloads and explicit unavailable states. This is not a claim of support for every provider/media type.

Preserve Studio ownership on every input/asset/conversation operation, CSRF protection, immutable server-side mappings and opaque browser handles. Provider paths/keys do not become browser input. Preview failure must not invent completion or erase valid catalog metadata.

For Agent Support, retain principal isolation, explicit attachment scope, untrusted draft/history handling, personality snapshots, model/consent binding and complete-context export policy. A transport change does not authorize a new provider. Revision-checked proposals, Apply and actual execution remain separate.

Static JSON validation, provider availability, real generation qualification and caller authorization are distinct. Retained functionality must keep its protections. Removal of obsolete functionality must fail explicitly for unsupported calls, not bypass validation or fabricate success.

## Intelligence-first implementation

[Studio #62](https://github.com/flamoris-jp/flamoris-studio/issues/62),
[Studio #63](https://github.com/flamoris-jp/flamoris-studio/pull/63),
[Agent #40](https://github.com/flamoris-jp/flamoris-ai-agent/pull/40) and
[Intelligence #12](https://github.com/flamoris-jp/flamoris-intelligence-mcp/pull/12)
record both retired internal MCP hops. Studio calls Agent JSON HTTP `/api/v1`;
Agent and raw Studio inference use the shared neutral provider library. The real
HTTP/service/adapter contract is tested with synthetic stores/providers, while the
owning PostgreSQL suites test durable authorization and request fences.

The code task implements exact removal targets, retained callers and functioning non-MCP contracts: internal Agent HTTP and a shared provider-adapter package. Concrete routes/configuration are defined and tested by the owning repositories, not invented by this coordination document. Keep uncertain outcomes unknown; do not silently replay, fallback or activate a GPU.

Controller implementation, generation gateway rewiring and reference-image expansion remain deferred. Retiring old generation calls is part of this cleanup. Live configuration/cutover and rollback remain a separate operational task; code acceptance does not prove a deployment changed.

## Owning tasks

Agent [#38](https://github.com/flamoris-jp/flamoris-ai-agent/issues/38), Intelligence MCP [#10](https://github.com/flamoris-jp/flamoris-intelligence-mcp/issues/10) and Studio #62 are the Intelligence-first preparation. Generation Controller [#1](https://github.com/flamoris-jp/flamoris-generation-controller/issues/1), Generation MCP [#67](https://github.com/flamoris-jp/flamoris-generation-mcp/issues/67), Runtime [#23](https://github.com/flamoris-jp/flamoris-ai-runtime/issues/23), Hub [#36](https://github.com/flamoris-jp/flamoris-mcp-hub/issues/36) and GPU Manager [#11](https://github.com/flamoris-jp/flamoris-gpu-node-manager/issues/11) retain their separately scoped work. Creating or merging documentation does not complete those Issues.

## 日本語

通常生成、素の推論、人格つきAgent Supportは別経路です。内部MCP依存の整理ではStudio側とAgent側の両方を確認します。ExecuteFlowとExecutionPlanとComfyWorkFlowは混同せず、Intelligenceを先行し、Generation Controllerや参照画像の開発はまだ再開しません。
