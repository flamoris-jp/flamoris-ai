# AGENTS.md

This repository is the integration, architecture, and roadmap home for the FLAMORIS AI ecosystem.

AI agents and human contributors should treat it as a coordination repository, not as a dumping ground for every AI-related implementation.

## Core role

`flamoris-ai` documents and coordinates:

- boundaries between FLAMORIS AI repositories;
- shared AI architecture principles;
- repository ownership and dependency direction;
- the integrated cross-repository roadmap;
- major cross-repository gates and convergence decisions.

It should remain small.

Runtime implementations belong in their dedicated repositories unless an Issue explicitly establishes a new shared boundary.

The canonical roadmap is `docs/ROADMAP.md`. Fast-changing implementation status belongs in the owning repository's Issues and PRs.

## Core dependency direction

The provider-neutral service dependency direction remains:

```text
Agent → Intelligence MCP → model/provider
```

FLAMORIS AI Runtime adds a separate model-adjacent execution path when direct control over active inference/workflow state is required:

```text
Agent / Application → AI Runtime → controlled model/backend execution
                                   └─→ Oblivionis
```

The planned response path is Oblivionis firing/state-derived signals → Runtime-owned bounded modulation. This is not an inversion of execution ownership, and it is distinct from a condition that starts new work.

Do not invert the service path by treating the persistent Agent as just another model provider inside Intelligence MCP. Do not turn AI Runtime into the owner of durable Agent identity or long-term Agent memory.

## Repository boundaries

### `flamoris-ai-agent`

Owns persistent Agent behavior and Agent-facing state, including:

- identity / personality;
- conversations/messages;
- memory;
- knowledge context;
- prompts / Agent policy;
- later tools and durable Agent orchestration state;
- the bounded Agent MCP surface.

For now, the Agent MCP surface belongs inside `flamoris-ai-agent`.

Do not create a separate `flamoris-agent-mcp` repository unless an Issue demonstrates a real deployment, lifecycle, or ownership boundary that requires the split.

The Agent may call Intelligence MCP and Generation MCP, but those services must not become second owners of Agent state.

### `flamoris-ai-runtime`

Owns the model-adjacent active execution boundary for the models/backends it directly controls.

Planned scope includes:

- active inference lifecycle and backend state where supported;
- Workflow IR validation/compilation/execution;
- scheduler-visible Jobs and Job-owned Continuations;
- interrupts, pause/resume/cancellation where supported;
- registered local/external capabilities;
- resource accounting/scheduling;
- structured runtime events and traces.

It must not become the durable authority for Agent identity, conversation history, personality, goals, or long-term Agent memory.

External services called through Runtime capabilities retain their own authority. GPU Node Manager remains the host-wide runtime/GPU lifecycle authority.

For Oblivionis-derived modulation, Runtime owns the mapping, supported execution points, timing, limits, and decision to apply a signal. Do not imply that a firing response grants execution permission or can directly rewrite arbitrary runtime internals.

### `Oblivionis`

Owns an experimental non-LLM dynamic state and memory model exploring experience-dependent AI behavior. The intended path is experience → changing oscillatory state → firing → runtime fluctuation → changed behavior.

Planned ownership includes:

- the evolving Active Field;
- oscillation/resonance/coupling/fatigue/fluctuation dynamics;
- model-level firing responses and state-derived modulation signals;
- forgetting transitions and model-state snapshots;
- associative/resonance recall;
- Profundumis latent storage semantics.

Oblivionis must not absorb Agent identity, workflow scheduling, application of modulation inside Runtime, media-generation domains, or external asset lifecycle.

Do not reduce its role to a memory lookup, an independent noise generator, or only a Workflow-start trigger. Firing, modulation of existing execution, and triggering new work are distinct concepts; not every firing or recall starts work. These remain planned integration concepts, not biological-fidelity or shipped-capability claims.

Max may be recorded during experience, but Max-state search and percentage reactivation apply after storage in Profundumis during association/recall. Never use retained Max as an ordinary firing/trigger threshold or continuous restoration rule. Keep [sensor/trigger work](https://github.com/flamoris-jp/Oblivionis/issues/3) and [latent recall](https://github.com/flamoris-jp/Oblivionis/issues/4) independently scoped; detailed model semantics remain in the owning repository.

The repository name intentionally omits the `flamoris-` prefix. Preserve that independent model identity unless a reviewed architecture decision explicitly changes it.

### `flamoris-intelligence-mcp`

Owns provider-neutral raw intelligence execution for language, reasoning, coding, and related bounded workloads.

It may route to local or remote providers.

It must not silently own:

- persistent Agent conversations;
- Agent memory/knowledge;
- personality or Agent policy;
- generated-media workflows/assets;
- product documents;
- LIME runtime switching.

### `flamoris-generation-mcp`

Owns provider-neutral generative-media and closely related media-analysis execution exposed through MCP, including capabilities, workflows, jobs, providers, and generated/materialized assets.

ComfyUI is a provider, not the identity of the project.

The Generation Hub foundation belongs inside `flamoris-generation-mcp` unless a future Issue demonstrates an independent boundary.

### `flamoris-mcp-hub`

MCP Hub is the routing/aggregation boundary.

The intended public namespace family is:

```text
generation.*
intelligence.*
agent.*
lime.*
```

Hub should preserve lazy upstream composition and must not become a duplicate Agent, Generation, Intelligence, or runtime state machine.

### `flamoris-jp/flamoris-gpu-node-manager`

GPU Node Manager is the provider-neutral local runtime/GPU authority. LIME is one deployment of that service, not part of the public repository identity.

Other AI services may query or request bounded runtime transitions through its public interfaces but must not reproduce systemd/GPU switching logic.

### Product repositories

FLAMORIS Studio and other applications remain authoritative for their own product/document state and editing behavior.

AI services may assist those applications but must not silently create a second source of truth.

### FLAMORIS Commons

Logging, generic MCP foundations, diagnostics, security primitives, and other infrastructure that is not specifically AI-domain logic belongs in FLAMORIS Commons or its dedicated shared repositories.

## Roadmap discipline

The roadmap is organized into three parallel tracks:

1. **Track A — ecosystem stabilization / existing Issue burn-down**
2. **Track B — Intelligence MCP / raw LLM access**
3. **Track C — Agent runtime / Agent MCP**

Use `flamoris-ai` trackers to coordinate sequencing and dependencies.

Use the owning repository's Issue as the implementation authority.

Do not copy full implementation specifications into both places.

Update the roadmap when:

- repository ownership changes;
- dependency direction changes;
- major phases or tracks change;
- a cross-repository gate is introduced/resolved;
- a new stable public boundary is introduced;
- the AI Runtime / Oblivionis integration boundary materially changes.

Do not update it for every minor commit.

## Architecture principles

1. **One authority per domain**
   - Keep state ownership explicit.
   - Do not duplicate conversations, jobs, documents, runtime state, or provider state without an explicit synchronization contract.

2. **Provider-neutral FLAMORIS boundaries**
   - Local and remote providers are replaceable implementation choices.
   - Avoid leaking provider-specific assumptions through public contracts unless intentionally provider-specific.

3. **Local-first, not local-only**
   - Support local runtimes without hard-coding machine names, usernames, private topology, tunnel IDs, credentials, or developer-local absolute paths.
   - Remote providers may use the same explicit provider boundary where appropriate.

4. **No speculative mega-framework**
   - Prefer small adapters and explicit contracts.
   - Add abstractions only when real implementations demonstrate a reusable boundary.

5. **Bounded and inspectable behavior**
   - Generation, inference, tool execution, filesystem access, network access, and resource use should have explicit limits.
   - Avoid hidden retries of non-idempotent operations.

6. **AI-native, human-authoritative**
   - AI-assisted development is welcome.
   - Humans remain responsible for review, licensing, security, compatibility, and release decisions.

## Before changing architecture

For a substantial or cross-repository change:

- read this file, README.md, and `docs/ROADMAP.md`;
- inspect the relevant child repositories and their `AGENTS.md` files;
- read the current Issues/design documents that define the scope;
- identify the authority for each piece of state;
- identify dependency direction;
- confirm whether the change belongs here, in one child repository, or in FLAMORIS Commons;
- avoid creating a new repository unless the boundary is clear enough to stand alone.

Prefer an Issue that records:

- the problem;
- current ownership;
- proposed boundary;
- dependency direction;
- migration impact;
- compatibility and security risks.

## Work handoff guidance

When handing implementation to ChatGPT Work, include where possible:

- target repository / Issue / branch;
- in-scope and out-of-scope work;
- design documents to read first;
- existing architecture to reuse;
- recommended model / reasoning;
- commit strategy;
- required tests;
- whether PR creation/review is in scope.

Use **Medium** reasoning for narrow/local implementation and focused tests.

Use **High** reasoning for public contract design, provider architecture, runtime/state-machine changes, cross-repository boundaries, difficult debugging, and final architecture review.

Keep meaningful changes in small, single-purpose commits.

Do not auto-merge unless explicitly requested.

## Documentation discipline

Do not duplicate fast-changing child-repository implementation status in README.

Keep README focused on stable responsibility boundaries and links.

Do not document planned behavior as already implemented.

Never commit or document secrets, private hostnames/topology, tunnel identifiers, model weights, generated private media, private datasets, or local credentials.

## Models, datasets, prompts, and generated media

Repository code licenses do not automatically cover AI models, model weights, datasets, generated media, third-party prompts, provider-hosted assets, or other non-code material.

Verify redistribution rights, commercial-use restrictions, attribution, and applicable terms before adding such material.

## Testing

This repository may remain documentation-only.

If executable code is introduced later, changes should include focused deterministic tests where practical and should not require live paid APIs, GPU hardware, private credentials, or locally installed model weights in normal CI.

## Licensing and support

Unless stated otherwise, code and documentation in this repository are licensed under Apache License 2.0.

FLAMORIS does not provide guaranteed individual support.

Repository documentation, Issues, tests, logs, and source code are the primary support references. AI-assisted self-support is encouraged.
