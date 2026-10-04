# Contributor and AI-agent instructions

This repository coordinates FLAMORIS AI architecture and cross-repository work. It is not a shared runtime implementation repository.

Read README.md, docs/ARCHITECTURE.md, docs/ai-ecosystem.md, docs/ROADMAP.md and [Issue #18](https://github.com/flamoris-jp/flamoris-ai/issues/18) before changing AI integration.

## Current authorization

The active correction stage permits documentation and Issue organization in Chat only. Terminology is fixed: `ExecuteFlow` for AI Runtime execution plans and `ComfyWorkFlow` for ComfyUI execution definitions; do not use bare `Workflow` for either. Do not delegate this stage to Work, implement code, merge PRs, deploy, restart services, change credentials, run paid inference or resume Workflow/reference-image development. Existing merged work is preserved. Documentation merge and Issue creation do not lift the hold; implementation and operational resumption need explicit user direction.

## Canonical boundaries

- `flamoris-generation-mcp` and `flamoris-intelligence-mcp` are external MCP adapters/facades. Internal Studio/Agent/Controller/AI Runtime calls do not use MCP or MCP Hub.
- MCP Hub owns external aggregation/routing/catalog/connection behavior, not application orchestration or an internal service bus.
- Generation Controller is the future internal generation-domain owner, but it remains documentation-only. Do not implement it in the current phase. The existing Generation MCP ComfyWorkFlow subsystem is not to be copied into it.
- **ComfyWorkFlow means ComfyUI/provider execution-definition construction. ExecuteFlow means inference control. Never move the ComfyUI JSON builder into AI Runtime because the names match.**
- ComfyUI executes its graph. The Controller builds/validates/submits it; JSON construction does not require Agent, Runtime, Hub or live GPU inference.
- AI Agent is optional personality, conversations, memory, principal/session and context policy. It is not the mandatory center of all inference or generation.
- AI Runtime remains model-adjacent inference execution with ExecuteFlows, active state, supported control points, Jobs/Continuations and resource accounting. Do not reduce it to a generic provider-API orchestrator.
- GPU Node Manager remains host-wide runtime/GPU lifecycle authority. CLI/HTTP/MCP may adapt the same implementation; an external MCP surface does not justify internal MCP dependencies.
- Replaceable gateways are ordinary internal interfaces. Do not invent a new central gateway, Intelligence Controller repository, service hop or deployment topology without a concrete reviewed need.

## Current implementation versus target

Current source/tests show what exists. Issue #18 and the corrected architecture define where ownership must go. Old diagrams and implementation-specific MCP adapters are migration inputs, not permission to extend the rejected dependency direction.

Describe unimplemented extraction honestly. Do not state that services or live routes have changed because a documentation PR exists. Preserve original evidence through pinned history and owning Issues; do not erase completed work or invent completion.

## Domain and safety preservation

One owner per state: Agent conversation/personality; Controller generation jobs/inputs/assets; Runtime active inference; GPU Manager host lifecycle; each product its own documents and UI state. Multiple frontends must not instantiate independent competing stores/reservations.

Preserve principal isolation, consent for the complete remotely sent context, immutable input/reference identity, bounded decoding/staging/transfer, safe paths and errors, provenance, automated qualification and uncertain-submit/no-replay behavior. Internal callers still require authorization. Do not replace qualification with a hand-written ready flag or human approval. Distinguish static JSON validation, provider availability and real execution evidence.

## Independent model boundaries

Maidionis owns specialization-neutral model/training/evaluation/artifact and bounded inference contracts. Arbitrium owns Decision-specific semantics, curricula and research evidence. Neither owns ExecuteFlow scheduling, tool permission or host lifecycle.

Oblivionis retains its independent name and experimental model authority: Active Field dynamics, firing responses, forgetting, snapshots and Profundumis recall. Runtime owns bounded application of modulation. Firing/modulation of existing inference and triggering new work are distinct. Max-state search and percentage reactivation apply after latent storage during association/recall, not ordinary firing thresholds or continuous restoration. Keep its sensor/trigger and latent-recall issues separate; do not silently redesign these models in this correction.

## Documentation and Issue discipline

The architecture is docs/ARCHITECTURE.md; docs/ai-ecosystem.md is orientation; docs/ROADMAP.md holds sequencing. Detailed contracts and source inventories belong in the owning repositories. Avoid duplicated fast-changing test counts or deployment status.

For affected old Issues, retain history and state, add explicit hold/replacement links, then re-scope remaining requirements after review. Do not mass-close, reopen completed work or leave conflicting implementation instructions unmarked. Native model and unrelated maintenance issues are not invalidated by this correction.

Use small documentation PRs. Do not auto-merge. Verify the diff is documentation-only; distinguish text/link checks from runtime tests or live acceptance. Read the actual target repository and its AGENTS.md before later changes. Do not invent commands, paths, services, APIs or credentials.

Public docs must remain portable and contain no private topology, user data, model weights or secrets. Follow the [shared repository policy](https://github.com/flamoris-jp/flamoris-commons/blob/main/docs/repository-policy.md). Generic shared infrastructure belongs in Commons or its dedicated repositories; AI implementations stay with their domain owners.

Code and documentation are Apache-2.0 unless otherwise noted. Model/data/media/provider terms remain separate. FLAMORIS is provided as-is without guaranteed individual support.
