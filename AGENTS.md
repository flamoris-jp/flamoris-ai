# Contributor and AI-agent instructions

This repository coordinates FLAMORIS AI architecture and cross-repository work. It is not a shared runtime implementation repository. Read README.md, docs/ARCHITECTURE.md, docs/ai-ecosystem.md, docs/ROADMAP.md and [AI #18](https://github.com/flamoris-jp/flamoris-ai/issues/18) before changing integration.

## Authorization and sequencing

The current pass covers documentation review, fixes and explicitly user-authorized documentation merges in Chat. It does not start Work implementation, code deletion, deployment, restarts, DB or credential changes, paid inference, or Generation/reference-image development. Documentation merge does not lift those holds.

The next implementation priority is Intelligence cleanup. Prepare a precise removal inventory and minimal retained execution contract before handing that work to Work. Generation Controller remains documentation-only. The existing Generation MCP ComfyWorkFlow subsystem is for later removal, not migration into Controller. Do not interpret this as authorization to recreate it elsewhere or erase retained data.

## Canonical terminology

- `ExecuteFlow`: AI Runtime's inference dependency/data/control flow.
- `ExecutionPlan`: the existing compiled Runtime representation. Preserve its distinct meaning and current code spelling.
- `ComfyWorkFlow`: ComfyUI graph/API-format JSON. Other provider requests are not automatically ComfyWorkFlow.
- Avoid bare `Workflow` as a new FLAMORIS architecture term. Literal existing symbols, wire fields, paths, external names and historical quotations may retain their real spelling. Never document a configuration/API rename as implemented without changing and testing the implementation.

## Dependency boundaries

Generation MCP and Intelligence MCP are external MCP adapters/facades. Internal Studio, Agent, Controller and Runtime calls use non-MCP interfaces. MCP Hub handles external catalog/routing/connections, not application orchestration or an internal service bus.

Agent is optional personality, conversation/memory, principal/session and context policy. Raw inference and generation do not require Agent. Keep a narrow replaceable internal execution boundary; do not introduce a universal gateway service or duplicate provider adapters merely to remove MCP.

AI Runtime remains model-adjacent inference with ExecuteFlow, compiled ExecutionPlan, supported control points, Jobs/Continuations and resource accounting. It is not a ComfyUI JSON builder or merely an outer loop over opaque APIs. ComfyUI executes its own graphs. Building JSON does not require Agent, AI Runtime, Hub or live GPU inference.

Generation Controller is only a future generation-domain owner. No new Controller code, framework, endpoint or service is authorized now. GPU Node Manager retains host-wide runtime/GPU lifecycle authority. Its CLI/HTTP/MCP adapters may share one manager; an external MCP surface does not imply internal MCP dependencies.

## Current implementation versus target

Current source/tests establish what exists; the latest explicit decision in #18 defines the target. Old internal MCP adapters are migration inputs, not permission to extend the rejected direction. Mark as-built contracts and future design separately. Historical guidance must not contradict active instructions without an explicit superseded label and link.

Preserve user data, identities and evidence. Deleting obsolete source or tests is different from deleting persisted records. On retained paths preserve principal isolation, consent for the complete remotely sent context, immutable input/reference identity, bounded decoding/staging/transfer, safe errors/paths, provenance and uncertain-submit/no-replay behavior. Internal callers still require authorization. Removing a feature must not create a bypass or claim unavailable behavior succeeded.

One state owner remains required. Multiple frontends must not create independent competing stores or reservations. Do not equate lifecycle READY, static JSON validity, real provider qualification and caller authorization.

## Independent model boundaries

Maidionis owns specialization-neutral model/training/evaluation/artifact and bounded inference contracts. Arbitrium owns Decision-specific semantics, curricula and evidence. Neither owns ExecuteFlow scheduling, tool authorization or host lifecycle.

Oblivionis retains its independent name and experimental model authority: Active Field dynamics, firing, forgetting, snapshots and Profundumis recall. Runtime owns bounded application of modulation. Modulating existing inference and triggering new work are distinct. Max-state search and percentage reactivation apply after latent storage during association/recall, not normal firing thresholds or continuous restoration. Do not redesign these models in this correction.

## Review and documentation discipline

Architecture lives in docs/ARCHITECTURE.md; the ecosystem map is orientation and the roadmap records sequencing. Detailed source/test inventories belong to the owning Issues. Retain old Issue history and state, link the new authority, and re-scope unfinished requirements instead of mass-closing or reopening completed work.

Use focused commits and review the resulting text, not only search/replace matches. Check links, code/config literals, dependency direction, terminology and current-versus-target claims. Merges require explicit user authorization; do not confuse mergeability with passing CI or deployment readiness. Do not claim tests that were not run.

Read each target repository before editing. Public documentation must contain no private topology, user data, model weights or secrets. Follow the [shared repository policy](https://github.com/flamoris-jp/flamoris-commons/blob/main/docs/repository-policy.md). Generic infrastructure belongs in Commons or its dedicated packages; domain logic stays with its owner.

Code and documentation are Apache-2.0 unless otherwise noted. Model/data/media/provider terms remain separate. FLAMORIS is provided as-is without guaranteed individual support.
