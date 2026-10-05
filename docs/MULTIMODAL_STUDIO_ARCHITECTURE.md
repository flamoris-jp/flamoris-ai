# Studio integration boundaries

Current integration boundaries under [AI #18](https://github.com/flamoris-jp/flamoris-ai/issues/18). Source contracts and pending live acceptance are recorded separately in [PROGRESS.md](../PROGRESS.md). Earlier proposals remain in [pinned history](LEGACY_DESIGN.md).

## Three separate internal paths

| Studio capability | Current source path |
| --- | --- |
| Image, Speech and Music | Authenticated Controller HTTP v1 → same Controller → providers |
| Raw Intelligence | Shared `flamoris_intelligence` provider adapters → approved providers |
| Agent Support | Agent JSON HTTP `/api/v1` → shared provider adapters → approved providers |

Studio's raw Intelligence and Agent Support now use replaceable non-MCP gateways.
Generation now uses direct authenticated Controller HTTP in the matched open
implementation PRs; live cutover has not occurred. Studio does not acquire provider graph internals, inference
scheduling or host lifecycle. Ordinary generation and inference need no Agent.

External access uses MCP Hub and the appropriate Generation / Intelligence MCP facade. External Agent MCP is also retained. Controller is the common non-MCP generation owner in the matched source; the [implemented retained-domain contract](https://github.com/flamoris-jp/flamoris-generation-controller/blob/640a5bd48c76e4bf736e9a3589c216123ccd18b3/docs/IMPLEMENTATION.md) defines the core/API and shared runtime. Source PR merge and live acceptance remain pending.

## Terminology and scope

ComfyWorkFlow is a ComfyUI execution graph/API-format JSON. Its construction is not ExecuteFlow or compiled ExecutionPlan. Non-ComfyUI generation providers use their own declared request/recipe contracts; do not rename all media requests ComfyWorkFlow.

Studio selects authorized generation metadata and supplies declared values. Provider adapters own graph construction and their declared request/recipe contracts. Image currently offers bounded builtin txt2img templates; reference-image generation is unavailable. Speech and Music use retained native contracts. Unsupported selections are explicitly unavailable. Concrete catalog and parameter support remain with the Studio and Generation owners.

ExecuteFlow controls inference; ExecutionPlan remains Runtime's compiled representation. Future inference-to-generation interaction is an optional capability with its own scope.

## Valid product requirements

Retain dedicated editors, supported-parameter discovery, metadata-first catalogs, bounded previews/downloads and explicit unavailable states. This is not a claim of support for every provider/media type.

Preserve Studio ownership on every input/asset/conversation operation, CSRF protection, immutable server-side mappings and opaque browser handles. Provider paths/keys do not become browser input. Preview failure must not invent completion or erase valid catalog metadata.

For Agent Support, retain principal isolation, explicit attachment scope, untrusted draft/history handling, personality snapshots, model/consent binding and complete-context export policy. A transport change does not authorize a new provider. Revision-checked proposals, Apply and actual execution remain separate.

Static JSON validation, provider availability, real generation qualification and caller authorization are distinct. Retained functionality must keep its protections. Removal of obsolete functionality must fail explicitly for unsupported calls, not bypass validation or fabricate success.

## Implemented Intelligence contracts

[Studio #62](https://github.com/flamoris-jp/flamoris-studio/issues/62),
[Studio #63](https://github.com/flamoris-jp/flamoris-studio/pull/63),
[Agent #40](https://github.com/flamoris-jp/flamoris-ai-agent/pull/40) and
[Intelligence #12](https://github.com/flamoris-jp/flamoris-intelligence-mcp/pull/12)
record the implemented interfaces. Studio calls Agent JSON HTTP `/api/v1`;
Agent and raw Studio inference use the shared neutral provider library. The real
HTTP/service/adapter contract is tested with synthetic stores/providers, while the
owning PostgreSQL suites test durable authorization and request fences.

Concrete routes/configuration are defined and tested by the owning repositories. Keep uncertain outcomes unknown; do not silently replay, fallback or activate a GPU. Provider selection, grants and complete-context remote consent remain enforced through the existing contracts.

Studio #65 replaces `GenerationGateway`'s upstream MCP transport with direct Controller HTTP. Controller owns domain constraints/profiles; Studio retains browser DTOs, consumer limits and validation of untrusted results. Its local request/history records are owner-scoped fences/projections, not a second generation reservation authority.

The exact non-MCP contracts, core/API and matched callers are implemented in Controller #5, Generation #71 and Studio #65, with final-head CI successful. Source merge is pending; reference-image expansion and live configuration/cutover/rollback are separate scopes. Code acceptance does not prove a deployment changed.

## Owning tasks

Agent [#38](https://github.com/flamoris-jp/flamoris-ai-agent/issues/38), Intelligence MCP [#10](https://github.com/flamoris-jp/flamoris-intelligence-mcp/issues/10) and Studio #62 link the accepted Intelligence contracts. Generation Controller [#1](https://github.com/flamoris-jp/flamoris-generation-controller/issues/1), Generation MCP [#67](https://github.com/flamoris-jp/flamoris-generation-mcp/issues/67), Runtime [#23](https://github.com/flamoris-jp/flamoris-ai-runtime/issues/23), Hub [#36](https://github.com/flamoris-jp/flamoris-mcp-hub/issues/36) and GPU Manager [#11](https://github.com/flamoris-jp/flamoris-gpu-node-manager/issues/11) retain their separately scoped work. [PROGRESS.md](../PROGRESS.md) distinguishes each accepted baseline from remaining phases.

## 日本語

通常生成、素の推論、人格つきAgent Supportは別経路です。raw IntelligenceとAgent内部は共通provider adapter、Agent SupportはAgent HTTP、GenerationはControllerの認証付きHTTPを使うソースへ変更しました。ExecuteFlow・ExecutionPlan・ComfyWorkFlowはそれぞれの責務を区別します。Controllerと対応する内部接続は実装PRで確認できますが、まだ未マージです。Studioの権限・履歴・ブラウザ境界は維持します。新しい参照画像は別scope、実機受け入れは未完了です。
