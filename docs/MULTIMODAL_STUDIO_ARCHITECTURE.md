# Status note: superseded terminology and sequencing

This proposal predates the 2026-10-04 correction in [AI #18](https://github.com/flamoris-jp/flamoris-ai/issues/18). Active terminology is `ExecutionPlan` for AI Runtime and `ComfyWorkFlow` for ComfyUI execution definitions. Generation Controller is not being implemented yet, and Intelligence boundary cleanup is the next implementation priority. Conflicting dependency directions or sequencing below are historical proposal material, not current implementation authority.

# Studio integration boundaries

Updated under [AI #18](https://github.com/flamoris-jp/flamoris-ai/issues/18), 2026-10-04. The previous integration proposal coordinated by #15 mixed internal MCP routing with generation and ExecutionPlan concepts. Its dependency direction and compulsory bridge sequence are superseded. The complete earlier proposal and baseline evidence remain available in [pinned history](LEGACY_DESIGN.md).

This is a documentation-stage target, not deployed behavior. Current implementation and live acceptance remain with their owning repositories.

## Three separate internal paths

```text
Studio generation -----> Generation Controller -> ComfyUI / other providers
Studio raw intelligence -> internal runtime / API / vendor interface
Studio Agent Support ---> AI Agent -> internal execution interface
```

Studio uses replaceable internal gateways without MCP or MCP Hub. It does not acquire provider graph internals, inference scheduling or host lifecycle authority. Agent Support is for personality-enabled interaction; ordinary generation and inference work without it.

The external route is separate:

```text
ChatGPT -> MCP Hub -> Generation MCP -> Generation Controller
                  -> Intelligence MCP -> approved internal capabilities
```

## ComfyWorkFlow UI does not edit Runtime IR

ComfyWorkFlow selection refers to provider metadata and ComfyUI execution-definition construction. The Controller applies declared parameter/reference bindings; ComfyUI executes the resulting JSON. Studio does not parse provider node graphs or translate every definition into AI Runtime IR.

AI Runtime's ExecutionPlan controls inference itself and is separately owned. Optional future inference-to-generation integration requires its own explicit use case; it is not a gate for ordinary generation or reference-image support.

## Valid product requirements to retain

Dedicated media editors, supported-parameter discovery, metadata-first result catalogs, bounded previews/downloads and explicit unsupported/unavailable states remain useful requirements. This correction does not claim all media/providers are implemented or approved.

Keep Studio user ownership on each input/asset/conversation operation, CSRF protections, immutable server-side mappings and safe browser handles. Provider paths, endpoints and keys do not become browser input. Preview/download failure should not invent completion or erase valid catalog state.

For Agent Support, retain principal isolation, explicit context attachment, untrusted draft/history handling, personality revision snapshots, model/consent binding and full-context remote-export authorization. Changing the internal transport must not change the Agent identity or silently authorize a new provider. Revision-checked proposals remain separate from applying changes or executing generation.

## Validation and qualification are different stages

Building valid provider JSON is not proof of successful generation or current model/node compatibility. Keep static validation, provider availability, actual qualification and user authorization distinct. Preserve existing automated verification and managed-reference protections until a separately reviewed change replaces them.

Do not require speculative include/composition engines, a Runtime bridge or new host instrumentation merely to declare a JSON-builder unit complete. Conversely, do not claim production readiness from a builder-only test or replace automated evidence with a manual ready flag.

## Migration discipline

Inventory the existing Studio gateway implementations under [Studio #62](https://github.com/flamoris-jp/flamoris-studio/issues/62). Controller/Agent contracts must retain identity, state, authorization and uncertain-request semantics before internal callers change. Multiple frontends must share one generation authority; metadata projection is not another generation engine.

Existing schemas, catalogs, data and evidence are not changed in this documentation pass. Keep unknown outcomes unknown; no implicit replay, fallback or GPU activation. Actual cutover and rollback are separately authorized operations, not consequences of merging these docs.

## Owning work

- [Generation Controller #1](https://github.com/flamoris-jp/flamoris-generation-controller/issues/1) and [Generation MCP #67](https://github.com/flamoris-jp/flamoris-generation-mcp/issues/67)
- [Studio #62](https://github.com/flamoris-jp/flamoris-studio/issues/62)
- [Agent #38](https://github.com/flamoris-jp/flamoris-ai-agent/issues/38) and [Intelligence MCP #10](https://github.com/flamoris-jp/flamoris-intelligence-mcp/issues/10)
- [Runtime #23](https://github.com/flamoris-jp/flamoris-ai-runtime/issues/23), [Hub #36](https://github.com/flamoris-jp/flamoris-mcp-hub/issues/36) and [GPU Manager #11](https://github.com/flamoris-jp/flamoris-gpu-node-manager/issues/11)

The old #15 children retain their requirements and evidence, with conflicting assumptions held for re-scoping. Detailed endpoint/schema/source contracts belong to the respective owners, not a second implementation specification here.

## 日本語

Studioの通常生成、素の推論、人格つきAgent Supportは別の内部経路です。ComfyUI用WorkflowのJSON生成をAI Runtimeへ移さず、内部でMCP Hubを経由しません。既存の認可・参照入力保護・検証・履歴は保持し、実装と実機変更は別途指示後に行います。
