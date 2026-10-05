# FLAMORIS AI Roadmap

Updated: 2026-10-05. Active authority: [AI #18](https://github.com/flamoris-jp/flamoris-ai/issues/18). [ARCHITECTURE.md](ARCHITECTURE.md) defines terminology and boundaries.

The detailed phase checklists, accepted source/CI baselines, pending operational
steps and cross-thread resumption/update rules are in [PROGRESS.md](../PROGRESS.md).
This roadmap lists remaining work; future phases do not lift
the Controller or live-operation holds.

## Current scope

The [README source matrix](../README.md#implemented-source-boundaries) links accepted internal Intelligence/Agent contracts and bounded generation. The [2026-10-05 review](REVIEW_2026-10-05.md) and PROGRESS §4.4 record accepted residual fixes and organization/coordination documentation. Their reviewed source acceptance is complete; live acceptance and future Controller work remain separate.

Studio generation still uses the retained MCP compatibility route. Controller design/implementation remains held. Production deployment, restarts, DB/data/credential changes, paid inference and new generation/reference-image development require their own resumed scope.

## Remaining phases

| Phase | Remaining work | Current status / start condition |
| --- | --- | --- |
| Live inventory and migration plan | Confirm installed versions, configuration, retained jobs/data, backup and rollback | Pending; resume only for an explicitly selected deployment scope (PROGRESS E1) |
| Intelligence/Agent live acceptance | Configure the accepted provider-library and Agent HTTP contracts; verify grants, complete-context consent, model identity and request fences | Pending operational work (PROGRESS E2) |
| Generation live acceptance | Verify retained Image/Speech/Music providers, inputs/assets and unresolved-job protections through the current compatibility route | Pending operational work (PROGRESS E3) |
| Controller contract | Review one minimal shared generation boundary and its state ownership | Held; library/service, API, port and migration form remain undecided (PROGRESS F) |
| Internal Generation cutover | Implement the reviewed Controller contract and connect internal callers and the external MCP adapter to one domain authority | Unimplemented; depends on contract review and explicit resumption (PROGRESS G) |
| Product extensions | Define reference-image, additional provider or multi-step generation requirements individually | Separate scopes; new reference-image/custom features remain held (PROGRESS H) |

Existing use of retained generation and Intelligence does not depend on Controller completion. Keep authorization, provenance, retained state and unknown/no-replay guarantees in each phase.

## Owning coordination Issues

| Repository | Issue | Responsibility / status |
| --- | --- | --- |
| AI Agent | [#38](https://github.com/flamoris-jp/flamoris-ai-agent/issues/38) | Optional personality, retained ExecutionClient, internal HTTP and direct execution adapters |
| Intelligence MCP | [#10](https://github.com/flamoris-jp/flamoris-intelligence-mcp/issues/10) | External facade over reusable non-MCP provider adapters |
| Studio | [#62](https://github.com/flamoris-jp/flamoris-studio/issues/62) | Implemented raw provider adapters and Agent HTTP; current Generation compatibility route |
| AI Runtime | [#23](https://github.com/flamoris-jp/flamoris-ai-runtime/issues/23) | ExecuteFlow distinct from compiled ExecutionPlan and ComfyWorkFlow; no kernel redesign |
| Generation Controller | [#1](https://github.com/flamoris-jp/flamoris-generation-controller/issues/1) | Future boundary documentation only; implementation held |
| Generation MCP | [#67](https://github.com/flamoris-jp/flamoris-generation-mcp/issues/67) | Bounded builtin/native generation, generic recipes and retained-state protections |
| MCP Hub | [#36](https://github.com/flamoris-jp/flamoris-mcp-hub/issues/36) | External routing audit; no internal service bus |
| GPU Node Manager | [#11](https://github.com/flamoris-jp/flamoris-gpu-node-manager/issues/11) | Host lifecycle/qualification boundary audit only |

Owning PRs and recorded checks establish source completion; an open coordination Issue alone does not make completed implementation pending. Accepted correction sequences and tracker history remain in PROGRESS.md, the linked Issues/PRs and Git history. The [historical roadmap](LEGACY_DESIGN.md) is traceability. Independent native-model, personality, GPU lifecycle and model-research tasks retain their own scopes.
