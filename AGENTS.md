# AGENTS.md

This repository is the integration and architecture home for the FLAMORIS AI ecosystem.

AI agents and human contributors should treat it as a coordination repository, not as a dumping ground for every AI-related implementation.

## Core role

`flamoris-ai` documents:

- the boundaries between FLAMORIS AI repositories;
- shared AI architecture principles;
- repository ownership and dependency direction;
- cross-repository plans and decisions that do not belong to one runtime alone.

It should remain small.

Runtime implementations belong in their dedicated repositories unless an Issue explicitly establishes a new shared boundary.

## Repository boundaries

### `flamoris-ai-agent`

Owns persistent agent behavior and agent-facing state, including:

- conversations;
- memory;
- knowledge;
- prompts;
- tools;
- agent orchestration;
- pluggable local or remote intelligence access.

Do not move provider runtime ownership, media-generation job ownership, or product document authority into the Agent.

### `flamoris-intelligence-hub`

Owns the planned provider-neutral orchestration boundary for language, reasoning, coding, and multi-agent workloads.

Its intended responsibilities include model/provider routing, task coordination, scheduling, multi-agent workflows, and explicit access to shared memory or knowledge services. Keep model/runtime-specific behavior behind adapters.

The Hub may route to local or remote providers, but it must not silently become the owner of conversations, Agent memory, product documents, or generated-media workflows.

### `flamoris-generation-mcp`

Owns provider-neutral generative-media execution exposed through MCP, including generation jobs, workflows, and generated assets.

ComfyUI is the first provider, not the identity of the project.

The Generation Hub foundation currently belongs inside `flamoris-generation-mcp`. Do not create a separate Generation Hub repository unless a future Issue demonstrates a clear independent boundary.

### Product repositories

FLAMORIS 2D, Cutwork, Kachinco, Studio, and other applications remain authoritative for their own project/document state and editing behavior.

AI services may assist those applications but must not silently create a second source of truth.

### FLAMORIS Commons

Logging, MCP foundations, diagnostics, security primitives, and other infrastructure that is not specifically AI-domain logic belongs in FLAMORIS Commons or its dedicated shared repositories.

## Architecture principles

1. **One authority per domain**
   - Keep state ownership explicit.
   - Do not duplicate conversations, jobs, documents, or provider state across services without an explicit synchronization contract.

2. **Provider-neutral FLAMORIS boundaries**
   - Local and remote providers are replaceable implementation choices.
   - Avoid exposing provider-specific assumptions through public FLAMORIS contracts unless they are intentionally provider-specific.

3. **Local-first, not local-only**
   - Support local runtimes without hard-coding machine names, usernames, absolute developer paths, private network topology, tunnel IDs, or credentials.
   - Remote providers may be supported through the same explicit provider boundary when appropriate.

4. **No speculative mega-framework**
   - Prefer small adapters and explicit contracts.
   - Do not centralize code merely because it is AI-related.
   - Add abstractions only when real implementations demonstrate a reusable boundary.

5. **Bounded and inspectable behavior**
   - Generation, inference, tool execution, filesystem access, and network access should have explicit limits.
   - Avoid hidden retries of non-idempotent operations.
   - Keep errors actionable without exposing secrets or private provider details.

6. **AI-native, human-authoritative**
   - AI-assisted development is welcome.
   - Humans remain responsible for review, licensing, security, compatibility, and release decisions.

## Before changing architecture

For a substantial or cross-repository change:

- read this file and README.md;
- inspect the relevant child repositories and their AGENTS.md files;
- read the current Issues/design documents that define the requested scope;
- identify the current authority for each piece of state;
- identify dependency direction;
- confirm whether the change belongs here, in one child repository, or in FLAMORIS Commons;
- avoid creating a new repository unless the boundary is clear enough to stand on its own.

Prefer an Issue that records:

- the problem;
- the current repository ownership;
- the proposed boundary;
- dependency direction;
- migration impact;
- compatibility and security risks.

## Documentation discipline

Keep README.md's repository map accurate when repositories are added, renamed, split, or retired.

Do not document planned behavior as already implemented. Clearly distinguish current implementation, active work, and future direction.

Environment-specific examples must remain generic.

Never commit or document:

- API keys or tokens;
- passwords or private keys;
- private hostnames or network topology;
- tunnel identifiers or personal deployment details;
- model weights;
- generated private media;
- private datasets;
- local credentials or developer-specific absolute paths.

## Models, datasets, prompts, and generated media

Repository code licenses do not automatically cover AI models, model weights, datasets, generated media, third-party prompts, provider-hosted assets, or other non-code material.

Before adding any such material:

- verify redistribution rights;
- verify commercial-use restrictions;
- record attribution requirements;
- record the exact applicable license or terms;
- keep incompatible or uncertain assets out of the repository.

Do not assume that a model being downloadable means it is redistributable.

Do not assume that generated output inherits this repository's Apache License.

## Testing

This repository may remain documentation-only.

If executable code is introduced later, changes should include focused deterministic tests where practical and should not require live paid APIs, GPU hardware, private credentials, or locally installed model weights in normal CI.

## Licensing and support

Unless stated otherwise, code and documentation in this repository are licensed under Apache License 2.0.

FLAMORIS does not provide guaranteed individual support.

Repository documentation, Issues, tests, logs, and source code are the primary support references. AI-assisted self-support is encouraged.
