# FLAMORIS AI Roadmap

Updated: 2026-09-26

This is the cross-repository roadmap for the AI- and MCP-facing parts of FLAMORIS.

`flamoris-ai` is the coordination and architecture control tower. It does not replace implementation authority in the repositories it coordinates.

## Trackers

- [#4 FLAMORIS AI integrated roadmap](https://github.com/flamoris-jp/flamoris-ai/issues/4)
- [#5 Track A — ecosystem stabilization and generation expansion](https://github.com/flamoris-jp/flamoris-ai/issues/5)
- [#6 Track B — Intelligence MCP and raw LLM access](https://github.com/flamoris-jp/flamoris-ai/issues/6)
- [#7 Track C — Agent runtime and Agent MCP](https://github.com/flamoris-jp/flamoris-ai/issues/7)

Implementation specifications remain in the repository that owns the behavior. This document records sequencing, boundaries, dependencies, and cross-repository gates.

## Scope

Included:

- `flamoris-generation-mcp`
- `flamoris-intelligence-mcp`
- `flamoris-ai-agent`
- `flamoris-mcp-hub`
- `flamoris-jp/flamoris-gpu-node-manager`
- AI-facing `flamoris-studio` integration
- future local / remote intelligence provider boundaries

Excluded:

- Windows image/video editor implementation such as FLAMORIS 2D, Cutwork, and Kachinco
- FLAMORIS Commons and generic shared infrastructure
- product-specific document/editing authority

## Target architecture

```text
ChatGPT / Studio / FLAMORIS clients
                │
                ▼
             MCP Hub
       ┌────────┼────────┬────────┐
       │        │        │        │
       ▼        ▼        ▼        ▼
 generation.* intelligence.* agent.* lime.*
       │        │        │        │
       ▼        │        ▼        ▼
Generation MCP  │   Agent MCP   GPU Node Manager
                │        │
                └───┬────┘
                    ▼
             Intelligence MCP
                    │
                    ▼
           local / remote providers
```

Core dependency direction:

```text
Agent → Intelligence MCP → model/provider
```

### Intelligence

Intelligence MCP is the provider-neutral boundary for raw language, reasoning, and coding execution.

It may be called directly by ChatGPT, Studio, Agent, or another FLAMORIS component when persistent Agent state is not required.

It does not own persistent conversations, Agent memory/knowledge, personality, Agent policy, generation assets, or LIME runtime switching.

### Agent

The Agent is a persistent FLAMORIS-aware entity. It owns Agent identity, conversations/messages, memory, knowledge context, personality/character context, prompts/Agent policy, and later durable tool/orchestration state.

The Agent exposes a bounded MCP surface separate from Intelligence MCP.

For now, Agent MCP belongs inside `flamoris-ai-agent`. Do not create a separate `flamoris-agent-mcp` repository unless a future Issue demonstrates a real deployment or ownership boundary.

### Generation

Generation MCP owns provider-neutral generative-media and closely related media-analysis workflows, jobs, and assets.

Its central execution model remains:

```text
Capability
    ↓
Workflow
    ↓
Job
    ↓
Provider
    ↓
Assets
```

### GPU Node Manager

`flamoris-gpu-node-manager` is the provider-neutral local runtime/GPU transition authority. LIME is one configured deployment rather than the identity of the service.

Other services may request activation/status through its bounded interfaces, but must not reproduce its systemd/GPU state machine.

### MCP Hub

MCP Hub is the aggregation/routing boundary.

Target namespaces:

```text
generation.*
intelligence.*
agent.*
lime.*
```

Hub must not become a hidden workflow or runtime orchestration authority.

---

# Operating model — three parallel tracks

Keep one useful unit of work moving in each track whenever dependencies allow.

| Track | Responsibility | Tracker |
|---|---|---|
| A | existing Issue burn-down, Generation/Hub/LIME/Studio integration | [#5](https://github.com/flamoris-jp/flamoris-ai/issues/5) |
| B | provider-neutral Intelligence MCP / raw LLM access | [#6](https://github.com/flamoris-jp/flamoris-ai/issues/6) |
| C | persistent Agent runtime / Agent MCP | [#7](https://github.com/flamoris-jp/flamoris-ai/issues/7) |

This keeps the three tracks independently reviewable while still converging on the shared Intelligence MCP and Generation MCP boundaries.

---

# Track A — ecosystem stabilization and generation expansion

## A0 — cross-repository asset and input contracts

Current contract work spans the owning repositories rather than living in one service:

- [generation-mcp #30 managed input snapshots](https://github.com/flamoris-jp/flamoris-generation-mcp/issues/30)
- [mcp-hub #13 bounded Generation asset transfer schemas](https://github.com/flamoris-jp/flamoris-mcp-hub/issues/13)
- [mcp-hub #14 managed-input schemas](https://github.com/flamoris-jp/flamoris-mcp-hub/issues/14)
- [studio #20 bounded transfer authorization](https://github.com/flamoris-jp/flamoris-studio/issues/20)
- [studio #21 managed-input authorization](https://github.com/flamoris-jp/flamoris-studio/issues/21)

Keep authorization in Studio, transfer/materialization authority in Generation MCP, and schema routing in MCP Hub.

## A1 — finish the Generation workflow foundation

Primary gate:

- [generation-mcp #19 versioned trusted ComfyUI workflow registry](https://github.com/flamoris-jp/flamoris-generation-mcp/issues/19)

Do this before substantial AnimeGen / SeeThrough integration so workflow graphs do not continue to accumulate as Python construction logic.


## A2 — multi-provider Generation expansion

Umbrella:

- [generation-mcp #25 YuE2 / SheetSage2 / Irodori / AnimeGen / SeeThrough](https://github.com/flamoris-jp/flamoris-generation-mcp/issues/25)

Current first provider-contract slice:

- [generation-mcp #31 YuE2 + SheetSage2 provider contracts](https://github.com/flamoris-jp/flamoris-generation-mcp/issues/31)

Recommended implementation order:

1. YuE2 + SheetSage2
2. Irodori
3. AnimeGen
4. SeeThrough

Keep #25 as the umbrella authority. Split child Issues in `flamoris-generation-mcp` when a phase becomes large enough to review independently.

Before AnimeGen, SeeThrough, or reference-audio speech become public capabilities, establish a bounded managed-input asset contract that never accepts arbitrary host filesystem paths.

For large/multi-asset outputs coordinate:

- [mcp-hub #8 user-visible generated asset delivery](https://github.com/flamoris-jp/flamoris-mcp-hub/issues/8)
- [studio #11 large generated asset reliability](https://github.com/flamoris-jp/flamoris-studio/issues/11)

Do not solve this only by increasing base64 payload limits.

## A3 — runtime improvements when demanded by providers

GPU Node Manager:

- [#1 provider-neutral runtime sleep/wake lifecycle](https://github.com/flamoris-jp/flamoris-gpu-node-manager/issues/1)

Provider-specific runtime profiles should be added only when a real provider integration needs them.

## A4 — Studio capability unlocks

- [studio #3 Music editor](https://github.com/flamoris-jp/flamoris-studio/issues/3) after the Generation music contract exists
- [studio #1 Image vertical slice](https://github.com/flamoris-jp/flamoris-studio/issues/1) should close as remaining acceptance criteria are satisfied

Studio Intelligence belongs primarily to Track B.

---

# Track B — Intelligence MCP and raw LLM access

## B0 — implemented Phase 1, live acceptance remains

The provider-neutral Python runtime and public Phase 1 contract are implemented and mock/CI validated.

Live deployment acceptance remains under the owning implementation issue:

- [intelligence-mcp #3 Phase 1 llama.cpp / GPT-OSS provider](https://github.com/flamoris-jp/flamoris-intelligence-mcp/issues/3)

## B1 — live provider acceptance and stabilization

Primary implementation authority remains:

- [intelligence-mcp #3 Phase 1 llama.cpp / GPT-OSS provider](https://github.com/flamoris-jp/flamoris-intelligence-mcp/issues/3)

The implemented Phase 1 contract covers health, capability/model discovery, bounded provider-neutral inference, request/output limits, timeout/cancellation, and normalized provider errors/metadata.

Do not add Agent memory/conversation authority here.

## B2 — MCP Hub integration

The public contract is sufficient to design/register the Hub catalog. Create an owning-repository Issue in `flamoris-mcp-hub` to expose Intelligence MCP lazily under:

```text
intelligence.*
```

Hub must not become an inference state machine. Because Hub discovery is static and upstream connection is lazy, schema/catalog integration does not require the live llama.cpp provider to be available; live provider acceptance remains the gate for runtime-readiness claims, not for registering the reviewed MCP contract.

## B3 — Studio Intelligence

- [studio #2 Intelligence editor](https://github.com/flamoris-jp/flamoris-studio/issues/2)

The Intelligence MCP public contract exists, so Studio #2 may proceed against it. Studio must not call llama.cpp directly; unavailable deployment/provider state should remain an explicit Studio availability state.

## B4 — future provider layer

A future `flamoris-llm` may become a local provider/runtime layer beneath Intelligence MCP if a concrete reusable boundary appears.

Do not create it merely to rename the current llama.cpp adapter.

---

# Track C — Agent runtime and Agent MCP

## C0 — Agent foundation implemented

Repository layout/CI, previous-conversation trust hardening, packaged console entry points, and the bounded Agent MCP surface are present in current main.

The remaining Phase 0 deployment gate is:

- [ai-agent #2 run imported Agent baseline on GPT-OSS via llama.cpp](https://github.com/flamoris-jp/flamoris-ai-agent/issues/2)

Mock/CI success is not a substitute for the live PostgreSQL/restart/provider acceptance recorded by that issue.

## C1 — Agent intelligence client boundary

Keep the current narrow execution boundary provider-neutral and preserve Agent-owned PostgreSQL state. The direct llama.cpp path remains a Phase 0 compatibility path while Track B stabilizes.

## C2 — Agent MCP implemented

The repository now exposes the deliberately small Agent MCP surface:

```text
agent.health
agent.ask
```

Do not expand internal Memory/Knowledge operations merely because MCP exists.

## C3 — migrate Agent execution to Intelligence MCP

After Track B is stable:

```text
Agent MCP
   ↓
Agent runtime
   ↓
Intelligence MCP
   ↓
GPT-OSS / future providers
```

This is an execution-dependency migration, not a Memory/Conversation redesign.

## C4 — later Agent evolution

Only after runtime and MCP boundaries are stable:

- Memory / Knowledge lifecycle evolution
- retrieval/indexing
- tools
- Generation MCP use
- bounded multi-agent/orchestration behavior

---

# Cross-track convergence

The main convergence architecture topic is:

- [#1 FLAMORIS Studio AI Workbench](https://github.com/flamoris-jp/flamoris-ai/issues/1)

The Workbench should consume stable contracts from the tracks rather than invent private replacements.

Target distinction:

```text
intelligence.*  = raw LLM / reasoning / coding
agent.*         = persistent FLAMORIS-aware Agent
generation.*    = image / video / music / voice / media analysis
lime.*          = explicit runtime/GPU operations
```

---

# Current parallel Work wave

| Track | Repository / Issue | Recommended model | Reasoning |
|---|---|---|---|
| A | `flamoris-generation-mcp#31` | GPT-5.6 Sol | High |
| B | `flamoris-intelligence-mcp#3` live acceptance | GPT-5.6 Sol | High |
| C | `flamoris-ai-agent#2` live acceptance | GPT-5.6 Sol | Medium |

Likely next wave:

| Track | Next direction |
|---|---|
| A | Generation #25 provider expansion after #31 |
| B | Hub `intelligence.*` integration |
| C | Agent execution migration onto Intelligence MCP |

Use Medium for focused local changes and tests. Use High for public contracts, state-machine changes, provider architecture, cross-repository boundaries, and final review.

Do not auto-merge implementation PRs unless explicitly requested.

---

# Roadmap maintenance rule

This file is a map, not a duplicate Issue database.

Update it when repository ownership, dependency direction, major phases, cross-repository gates, or stable public boundaries change.

Fast-changing implementation details belong in the owning repository's Issues and PRs.
