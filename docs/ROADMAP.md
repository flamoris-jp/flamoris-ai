# FLAMORIS AI Roadmap

Updated: 2026-10-04. Active authority: [AI #18](https://github.com/flamoris-jp/flamoris-ai/issues/18). [ARCHITECTURE.md](ARCHITECTURE.md) defines terminology and boundaries.

## Current scope

Review, correct and explicitly merge documentation in Chat. This does not start a Work task, source deletion, deployment, service restart, DB/credential change or paid inference. Generation and reference-image development remain paused at the user-merged baseline.

**Intelligence first. Controller not yet. Existing MCP-side ComfyWorkFlow implementation: later removal, not transfer or automatic recreation.**

## Sequence

1. Finish the current documentation review/fix/merge pass.
2. Under Agent #38, Intelligence MCP #10 and Studio #62, inventory internal MCP-only configuration, discovery, dispatch and translation; identify retained callers, execution contracts and safeguards. Specify the smallest functioning non-MCP path before deleting a currently used path.
3. Hand a bounded deletion-first Intelligence task to Work only under explicit implementation authorization. Keep Agent personality/conversations/principals, model provenance, complete-context consent and durable request fences. Do not remove the external Intelligence MCP facade simply because internal clients stop using it.
4. Keep Generation Controller unimplemented and Generation #67 paused. Later inventory the MCP-side ComfyWorkFlow removal, affected public tools/callers/tests and retained data separately. Do not migrate that subsystem into Controller or make a replacement a prerequisite for Intelligence work.
5. Perform live migration, configuration changes, reconciliation and rollback only under separate operational approval.

Deleting obsolete code is not permission to delete historical records or leave retained calls silently succeeding without execution. Do not introduce a fallback to the deprecated MCP route. Do not choose endpoints, ports, new repositories or a universal gateway framework in this roadmap.

## Owning correction tasks

| Repository | Issue | Scope / sequencing |
| --- | --- | --- |
| AI Agent | [#38](https://github.com/flamoris-jp/flamoris-ai-agent/issues/38) | First: optional personality, retained ExecutionClient and removal inventory for internal MCP coupling |
| Intelligence MCP | [#10](https://github.com/flamoris-jp/flamoris-intelligence-mcp/issues/10) | First: external facade, no internal gateway requirement |
| Studio | [#62](https://github.com/flamoris-jp/flamoris-studio/issues/62) | Intelligence caller audit first; generation rewiring deferred |
| AI Runtime | [#23](https://github.com/flamoris-jp/flamoris-ai-runtime/issues/23) | ExecuteFlow distinct from compiled ExecutionPlan and ComfyWorkFlow; no kernel redesign |
| Generation Controller | [#1](https://github.com/flamoris-jp/flamoris-generation-controller/issues/1) | Future boundary documentation only; implementation held |
| Generation MCP | [#67](https://github.com/flamoris-jp/flamoris-generation-mcp/issues/67) | Later MCP-side ComfyWorkFlow retirement; no transfer to Controller |
| MCP Hub | [#36](https://github.com/flamoris-jp/flamoris-mcp-hub/issues/36) | External routing audit; no internal service bus |
| GPU Node Manager | [#11](https://github.com/flamoris-jp/flamoris-gpu-node-manager/issues/11) | Host lifecycle/qualification boundary audit only |

These are linked design/correction Issues, not proof that implementation or inventories are complete.

## Existing tracker reconciliation

| Existing trackers | Treatment |
| --- | --- |
| AI #1/#4/#5/#6/#7/#15/#17 | Preserve requirements/history; supersede internal-MCP diagrams and automatic parallel-work instructions |
| Generation #19/#42/#30 | Hold ComfyWorkFlow/reference work; assess remaining scope only when Generation cleanup resumes |
| Generation #25/#31 and #45-related work | Preserve provider/acceptance evidence; no new expansion, mandatory bridge or blanket extraction |
| Agent #24/#35 and Intelligence #8 | Replace internal-MCP assumptions through the Intelligence-first tasks |
| Studio #39/#56/#36/#21/#11/#3 | Preserve valid UI/authorization/result requirements; do not start generation rewiring yet |
| Hub #26/#28 and Runtime #19 | Hold conflicting rollout/bridge assumptions; audit separately |

Retain history and links; do not mass-close, reopen completed work or erase evidence. Code cleanup Issues stay open until their actual scope is accepted. Independent native-model, personality, GPU lifecycle and model-research tasks are not invalidated or started by this pass.

The [historical roadmap](LEGACY_DESIGN.md) is traceability, not active execution guidance. This file records sequencing rather than duplicating test counts, release status or live host facts.
