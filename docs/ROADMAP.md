# FLAMORIS AI Roadmap

Updated: 2026-10-05. Active authority: [AI #18](https://github.com/flamoris-jp/flamoris-ai/issues/18). [ARCHITECTURE.md](ARCHITECTURE.md) defines terminology and boundaries.

The detailed phase checklists, accepted source/CI baselines, pending operational
steps and cross-thread resumption/update rules are in [PROGRESS.md](../PROGRESS.md).
This roadmap retains the responsibility/sequence map; future phases do not lift
the Controller or live-operation holds.

## Current scope

The documentation pass and deletion-first source implementation are recorded in
the [README source matrix](../README.md#implemented-source-boundaries) and owning
Issues/PRs. The internal Intelligence/Agent contracts and matching generation
retirement are implemented there; exact review/CI/merge evidence remains with each
owner. Production deployment, restarts, DB/data/credential changes, paid inference
and new generation/reference-image development remain separate.

**Internal Intelligence/Agent cleanup and custom generation retirement are implemented. Controller remains held; basic generation, generic recipes and retained-data fences remain. No transfer or recreation.**

## Accepted implementation sequence

The following source sequence is complete. Current follow-up review and remaining
operational/Controller work are recorded separately in PROGRESS.md.

1. Documentation review/fix/merge is complete.
2. Under Agent #38, Intelligence MCP #10 and Studio #62, inventory internal MCP-only configuration, discovery, dispatch and translation; identify retained callers, execution contracts and safeguards.
3. Implement the smallest functioning non-MCP routes: Studio-to-Agent HTTP and direct shared provider adapters for Agent/raw Studio inference. Keep personality/conversations/principals, model provenance, complete-context consent and durable request fences. External MCP remains separate.
4. Under Generation #67 and Runtime #23, retire added definition registration/versioning/composition/Runtime delegation and generation-specific lowering. Preserve native ExecuteFlow, original bounded generation templates, generic recipes and unresolved-job fences; update Studio/Hub callers and external catalogs. Controller remains unimplemented.
5. Verify, independently review, fix and merge. Record exact source acceptance without claiming a live cutover. Production migration, configuration and recovery/rollback remain separate.

Deleting obsolete code is not permission to delete historical records or leave retained calls silently succeeding without execution. Do not introduce a fallback to the deprecated MCP route. Do not choose endpoints, ports, new repositories or a universal gateway framework in this roadmap.

## Owning correction tasks

| Repository | Issue | Scope / sequencing |
| --- | --- | --- |
| AI Agent | [#38](https://github.com/flamoris-jp/flamoris-ai-agent/issues/38) | Optional personality, retained ExecutionClient, internal HTTP and direct execution adapters |
| Intelligence MCP | [#10](https://github.com/flamoris-jp/flamoris-intelligence-mcp/issues/10) | External facade over reusable non-MCP provider adapters |
| Studio | [#62](https://github.com/flamoris-jp/flamoris-studio/issues/62) | Direct raw inference and Agent HTTP; retired generation calls unavailable |
| AI Runtime | [#23](https://github.com/flamoris-jp/flamoris-ai-runtime/issues/23) | ExecuteFlow distinct from compiled ExecutionPlan and ComfyWorkFlow; no kernel redesign |
| Generation Controller | [#1](https://github.com/flamoris-jp/flamoris-generation-controller/issues/1) | Future boundary documentation only; implementation held |
| Generation MCP | [#67](https://github.com/flamoris-jp/flamoris-generation-mcp/issues/67) | Added registry/composition/delegation retirement; basic generation and state protections retained |
| MCP Hub | [#36](https://github.com/flamoris-jp/flamoris-mcp-hub/issues/36) | External routing audit; no internal service bus |
| GPU Node Manager | [#11](https://github.com/flamoris-jp/flamoris-gpu-node-manager/issues/11) | Host lifecycle/qualification boundary audit only |

These are linked correction Issues. Their PRs and recorded checks establish source completion; this table is not deployment evidence.

## Existing tracker reconciliation

| Existing trackers | Treatment |
| --- | --- |
| AI #1/#4/#5/#6/#7/#15/#17 | Preserve requirements/history; supersede internal-MCP diagrams and automatic parallel-work instructions |
| Generation #19/#42/#30 | Retire the mistaken custom subsystem; preserve historical evidence and keep new reference-image expansion held |
| Generation #25/#31 and #45-related work | Preserve provider/acceptance evidence; no new expansion, mandatory bridge or blanket extraction |
| Agent #24/#35 and Intelligence #8 | Replace internal-MCP assumptions through the Intelligence-first tasks |
| Studio #39/#56/#36/#21/#11/#3 | Preserve valid UI/authorization/result requirements; internal Intelligence/Agent cutover and old generation call retirement are recorded in #62; Controller rewiring remains deferred |
| Hub #26/#28 and Runtime #19 | Hold conflicting rollout/bridge assumptions; audit separately |

Retain history and links; do not mass-close, reopen completed work or erase evidence. Code cleanup Issues stay open until their actual scope is accepted. Independent native-model, personality, GPU lifecycle and model-research tasks are not invalidated or started by this pass.

The [historical roadmap](LEGACY_DESIGN.md) is traceability, not active execution guidance. This file records sequencing rather than duplicating test counts, release status or live host facts.
