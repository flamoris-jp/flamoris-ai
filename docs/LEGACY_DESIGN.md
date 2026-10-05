# Historical architecture records

The current target is [ARCHITECTURE.md](ARCHITECTURE.md), coordinated by [AI #18](https://github.com/flamoris-jp/flamoris-ai/issues/18). Earlier diagrams and Work sequences are traceability, not instructions to continue them.

## Immutable pre-correction snapshot

These links pin the inspected pre-correction main commit `b7ddfe1982efdc6f1a99727e406945acc6ffe368`:

- [Original README](https://github.com/flamoris-jp/flamoris-ai/blob/b7ddfe1982efdc6f1a99727e406945acc6ffe368/README.md)
- [Original AGENTS](https://github.com/flamoris-jp/flamoris-ai/blob/b7ddfe1982efdc6f1a99727e406945acc6ffe368/AGENTS.md)
- [Original ecosystem map](https://github.com/flamoris-jp/flamoris-ai/blob/b7ddfe1982efdc6f1a99727e406945acc6ffe368/docs/ai-ecosystem.md)
- [Original roadmap](https://github.com/flamoris-jp/flamoris-ai/blob/b7ddfe1982efdc6f1a99727e406945acc6ffe368/docs/ROADMAP.md)
- [Original Studio proposal and baseline evidence](https://github.com/flamoris-jp/flamoris-ai/blob/b7ddfe1982efdc6f1a99727e406945acc6ffe368/docs/MULTIMODAL_STUDIO_ARCHITECTURE.md)

Historical files retain original terms, source references, safeguards and status reports. Do not rewrite that history to pretend a naming or deployment migration already occurred.

## Superseded assumptions

Internal Agent -> Intelligence MCP and Studio -> MCP Hub/MCP routes are not the target. MCP is the external facade, Agent is optional personality and internal execution uses non-MCP boundaries.

Use ExecuteFlow for Runtime inference flow, preserve the compiled ExecutionPlan
representation, and use ComfyWorkFlow for ComfyUI graph/JSON. The added Generation
MCP custom subsystem is retired, not transferred to Controller or Runtime. Original
builtin/native recipes remain. Controller stays unimplemented; no automatic
replacement is ordered. Old composition/bridge/parallel-work sequences do not
override the correction.

## Evidence and authorization

Retain valid authorization, immutable references, provenance, uncertain-submit protections, limits and real acceptance records. Deleting obsolete code is not deleting user data, source history or proof of completed work. Review unfinished Issue scope individually; do not mass-close or reopen completed work.

The subsequent explicit instruction authorized deletion-first implementation,
internal Intelligence connections, tests, review/fixes and source merges under #18.
It does not authorize deployment, DB/data/grant/credential changes, provider calls
or new Controller/reference-image development. The [source matrix](../README.md#implemented-source-boundaries)
links exact acceptance without rewriting the historical records.
