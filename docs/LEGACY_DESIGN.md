# Historical architecture records

The corrected target is [ARCHITECTURE.md](ARCHITECTURE.md), coordinated by [AI #18](https://github.com/flamoris-jp/flamoris-ai/issues/18). Earlier dependency diagrams and Work handoff sequences are preserved here for traceability, not as instructions to continue them.

## Immutable pre-correction snapshot

The following links pin commit `b7ddfe1982efdc6f1a99727e406945acc6ffe368`, the main baseline inspected before this documentation correction:

- [Original README](https://github.com/flamoris-jp/flamoris-ai/blob/b7ddfe1982efdc6f1a99727e406945acc6ffe368/README.md)
- [Original AGENTS instructions](https://github.com/flamoris-jp/flamoris-ai/blob/b7ddfe1982efdc6f1a99727e406945acc6ffe368/AGENTS.md)
- [Original AI ecosystem map](https://github.com/flamoris-jp/flamoris-ai/blob/b7ddfe1982efdc6f1a99727e406945acc6ffe368/docs/ai-ecosystem.md)
- [Original roadmap and track history](https://github.com/flamoris-jp/flamoris-ai/blob/b7ddfe1982efdc6f1a99727e406945acc6ffe368/docs/ROADMAP.md)
- [Original Multimodal Studio proposal and inspected baseline](https://github.com/flamoris-jp/flamoris-ai/blob/b7ddfe1982efdc6f1a99727e406945acc6ffe368/docs/MULTIMODAL_STUDIO_ARCHITECTURE.md)

These records include useful feature requirements, inspected revisions and safeguards as well as the superseded assumptions. Preserve evidence; do not recreate stale implementation status as current fact.

## What is superseded

Internal `Agent -> Intelligence MCP` and `Studio -> MCP Hub/MCP` paths are not the target. Generation-domain logic is to be separated into Generation Controller, rather than making Generation MCP the permanent internal owner. ComfyUI Workflow construction and AI Runtime inference Workflows are distinct; the former is not transferred to the latter. Agent is optional personality.

Old include/composition/Runtime-bridge plans do not automatically become prerequisites for simple ComfyUI JSON construction or for Controller extraction. Existing code is not reverted by changing this documentation.

## What is not discarded

Valid authorization, input/reference confinement, identity/provenance, uncertain-submit protections, bounded behavior and automated verification requirements remain. Model-specialization and native inference work retain their owners. Each old Issue's remaining feature/acceptance scope must be reviewed explicitly; no mass-close, silent deletion or reopening of completed work is implied.

## Review status versus execution permission

The current user instruction is documentation and Issue organization in Chat. Historical Work assignments are inactive for this correction. Merging new documentation does not authorize code extraction, further feature development or live operations. Follow #18 and obtain explicit resumption instructions.
