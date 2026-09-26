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
- `flamoris-net/flamoris-lime-manager`
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
Generation MCP  │   Agent MCP   LIME Manager
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

### LIME Manager

LIME Manager remains the sole runtime/GPU transition authority for LIME.

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

This lets Agent modernization proceed before Intelligence MCP is complete, while still converging on Intelligence MCP later.

---

# Track A — ecosystem stabilization and generation expansion

## A0 — small deployment / contract debt

Generation MCP:

- [#24 single-instance reservation contract](https://github.com/flamoris-jp/flamoris-generation-mcp/issues/24)

MCP Hub:

- [#2 align Generation catalog authentication contract](https://github.com/flamoris-jp/flamoris-mcp-hub/issues/2)
- [#7 external exposure defaults and client authentication](https://github.com/flamoris-jp/flamoris-mcp-hub/issues/7)

## A1 — finish the Generation workflow foundation

Primary gate:

- [generation-mcp #19 versioned trusted ComfyUI workflow registry](https://github.com/flamoris-jp/flamoris-generation-mcp/issues/19)

Do this before substantial AnimeGen / SeeThrough integration so workflow graphs do not continue to accumulate as Python construction logic.

Then:

- [generation-mcp #22 provider output retention / cleanup](https://github.com/flamoris-jp/flamoris-generation-mcp/issues/22)

## A2 — multi-provider Generation expansion

Umbrella:

- [generation-mcp #25 YuE2 / SheetSage2 / Irodori / AnimeGen / SeeThrough](https://github.com/flamoris-jp/flamoris-generation-mcp/issues/25)

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

LIME Manager:

- [#13 managed persistent Irodori runtime](https://github.com/flamoris-net/flamoris-lime-manager/issues/13)
- [#10 runtime sleep/wake lifecycle](https://github.com/flamoris-net/flamoris-lime-manager/issues/10)

## A4 — Studio capability unlocks

- [studio #3 Music editor](https://github.com/flamoris-jp/flamoris-studio/issues/3) after the Generation music contract exists
- [studio #1 Image vertical slice](https://github.com/flamoris-jp/flamoris-studio/issues/1) should close as remaining acceptance criteria are satisfied

Studio Intelligence belongs primarily to Track B.

---

# Track B — Intelligence MCP and raw LLM access

## B0 — status hygiene

- [intelligence-mcp #2 implementation maturity/status](https://github.com/flamoris-jp/flamoris-intelligence-mcp/issues/2)

This should not block runtime implementation.

## B1 — first real Intelligence MCP

Primary implementation authority:

- [intelligence-mcp #3 Phase 1 llama.cpp / GPT-OSS provider](https://github.com/flamoris-jp/flamoris-intelligence-mcp/issues/3)

The first stable contract should cover health, capability/model discovery, bounded provider-neutral inference, request/output limits, timeout/cancellation, and normalized provider errors/metadata.

Do not add Agent memory/conversation authority here.

## B2 — MCP Hub integration

After Phase 1 is merged, create an owning-repository Issue in `flamoris-mcp-hub` to expose Intelligence MCP lazily under:

```text
intelligence.*
```

Hub must not become an inference state machine.

## B3 — Studio Intelligence

- [studio #2 Intelligence editor](https://github.com/flamoris-jp/flamoris-studio/issues/2)

Begin only after the Intelligence MCP public contract exists. Studio must not call llama.cpp directly.

## B4 — future provider layer

A future `flamoris-llm` may become a local provider/runtime layer beneath Intelligence MCP if a concrete reusable boundary appears.

Do not create it merely to rename the current llama.cpp adapter.

---

# Track C — Agent runtime and Agent MCP

## C0 — normalize the imported Agent baseline

Recommended order:

1. [ai-agent #6 repository layout / CI](https://github.com/flamoris-jp/flamoris-ai-agent/issues/6)
2. [ai-agent #7 previous-conversation trust boundary](https://github.com/flamoris-jp/flamoris-ai-agent/issues/7)
3. [ai-agent #2 run baseline on GPT-OSS via llama.cpp](https://github.com/flamoris-jp/flamoris-ai-agent/issues/2)

The direct llama.cpp path in #2 is intentionally temporary so Track C can progress independently of Track B.

## C1 — stabilize the Agent intelligence client boundary

After #2, create an owning-repository Issue to:

- define a narrow intelligence client interface;
- normalize request/response/error behavior;
- remove accidental provider coupling;
- add provider-failure tests;
- preserve existing Agent-owned PostgreSQL state unless evidence requires migration.

## C2 — expose Agent MCP

Create an owning-repository Issue in `flamoris-ai-agent`.

Start deliberately small.

Initial candidate surface:

```text
agent.health
agent.ask
```

Do not expose every internal Memory/Knowledge operation merely because MCP exists.

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

# First parallel Work wave

| Track | Repository / Issue | Recommended model | Reasoning |
|---|---|---|---|
| A | `flamoris-generation-mcp#19` | GPT-5.6 Sol | High |
| B | `flamoris-intelligence-mcp#3` | GPT-5.6 Sol | High |
| C | `flamoris-ai-agent#6` then `#7` | GPT-5.6 Sol | Medium |

Likely second wave:

| Track | Next direction |
|---|---|
| A | Generation #25 Phase B: YuE2 + SheetSage2 |
| B | Hub `intelligence.*` integration |
| C | Agent #2 GPT-OSS baseline |

Use Medium for focused local changes and tests. Use High for public contracts, state-machine changes, provider architecture, cross-repository boundaries, and final review.

Do not auto-merge implementation PRs unless explicitly requested.

---

# Roadmap maintenance rule

This file is a map, not a duplicate Issue database.

Update it when repository ownership, dependency direction, major phases, cross-repository gates, or stable public boundaries change.

Fast-changing implementation details belong in the owning repository's Issues and PRs.
