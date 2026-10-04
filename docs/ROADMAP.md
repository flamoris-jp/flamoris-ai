# FLAMORIS AI Roadmap

Updated: 2026-10-04. Active architecture correction: [#18](https://github.com/flamoris-jp/flamoris-ai/issues/18).

## Current stage: documentation and Issues in Chat

Workflow and reference-image development is paused at the user's merged baseline. Current work does not authorize implementation, Work delegation, merge, deployment, service restart, DB/credential changes or paid calls. Merging documentation does not lift the hold. Explicit user direction is required to resume implementation or operations.

The target is [ARCHITECTURE.md](ARCHITECTURE.md): external MCP adapters, internal Generation Controller, optional personality Agent and model-adjacent inference Runtime. Generation Workflow means ComfyUI/provider JSON construction, not AI Runtime's inference Workflow.

## Owning correction tasks

These are design/documentation tasks now, not an automatically executing implementation queue.

| Repository | Issue | Scope |
| --- | --- | --- |
| Generation Controller | [#1](https://github.com/flamoris-jp/flamoris-generation-controller/issues/1) | Identity, domain extraction inventory, minimal internal contract and migration plan |
| Generation MCP | [#67](https://github.com/flamoris-jp/flamoris-generation-mcp/issues/67) | External adapter versus generation domain; ComfyUI builder stays with Controller |
| AI Agent | [#38](https://github.com/flamoris-jp/flamoris-ai-agent/issues/38) | Optional personality and non-MCP internal execution boundary |
| Intelligence MCP | [#10](https://github.com/flamoris-jp/flamoris-intelligence-mcp/issues/10) | External intelligence facade; reusable adapter ownership review |
| AI Runtime | [#23](https://github.com/flamoris-jp/flamoris-ai-runtime/issues/23) | Inference Workflow semantics, no compulsory ComfyUI bridge |
| Studio | [#62](https://github.com/flamoris-jp/flamoris-studio/issues/62) | Separate internal generation, raw inference and Agent Support paths |
| MCP Hub | [#36](https://github.com/flamoris-jp/flamoris-mcp-hub/issues/36) | External routing only; remove internal consumer assumptions |
| GPU Node Manager | [#11](https://github.com/flamoris-jp/flamoris-gpu-node-manager/issues/11) | Host lifecycle versus generation qualification audit, not core redesign |

The parent links each child, and each child links the parent. Detailed source/test inventories remain child deliverables; creating them does not complete them.

## Sequence

### Now: make the boundary unambiguous

Update the central docs and new Controller identity, preserve history, and annotate affected old Issues with hold/correction links. Distinguish current code from the target. No code moves in this stage.

### Next design review

Inspect exact current source/test paths and actual callers. Classify retain/extract/defer behavior. Decide the smallest internal contract and a migration strategy with one generation state owner. Do not invent a generic framework, service topology or new Intelligence Controller repository.

Builder-only, domain/provider, MCP-mapping and live acceptance are separate evidence layers. Existing authorization and qualification requirements stay intact. Optional v3 composition and Runtime bridge ideas are not prerequisites for ordinary ComfyUI JSON construction.

### After explicit implementation approval

Extract existing generation logic without simultaneous feature expansion, adapt the external MCP facade, and change internal consumers to the non-MCP contract. Preserve job/asset/input IDs, state, reservations, unknown outcomes, provenance, safety and compatibility. Review narrow PRs; do not treat this paragraph as an instruction to start.

### After separate operational approval

Plan one-authority cutover, retained-data/uncertain-work reconciliation, client isolation, actual provider qualification and rollback. Offline tests are not live evidence. Do not activate runtimes or run paid calls as part of a documentation task.

## Existing tracker reconciliation

| Existing trackers | Treatment |
| --- | --- |
| AI #1/#4/#5/#6/#7/#15/#17 | Preserve requirements/history; old internal-MCP diagrams and automatic parallel-work sequence are superseded by #18 |
| Generation #19/#42/#30 | Hold further Workflow/reference work; re-scope remaining generation requirements through Controller #1 and Generation MCP #67 |
| Generation #25/#31 and work tracked by #45 | Retain provider and completed v3 evidence; separate expansion/optional composition from extraction |
| Agent #24/#35 and Intelligence #8 | Do not extend internal MCP execution; re-scope under Agent #38 / Intelligence #10 |
| Studio #39/#56/#36/#21/#11/#3 | Retain valid UI/authorization/result requirements; replace internal transport assumptions through #62 |
| Hub #26/#28 and Runtime #19 | Hold old rollout/bridge assumptions; audit through #36 / #23 |

Do not mass-close or reopen completed work. Add explicit links to affected unfinished work, then decide item by item whether to revise, transfer remaining scope, or supersede with a linked replacement. Keep merged PRs and evidence.

Independent Agent personality/principal work, Runtime native-model issues, GPU sleep/wake, and Maidionis/Arbitrium/Oblivionis research are not invalidated by this correction. They remain with their owners and are not started by this documentation pass.

The [previous roadmap](LEGACY_DESIGN.md) records the old tracks and implementation history. It must not be used to restart the rejected architecture.

## Maintenance

This file tracks boundaries and sequencing, not every build result. Exact completion, test counts, open bugs and live deployment state belong in the owning Issues/PRs. Changes to those facts do not require duplicating a second status database here.
