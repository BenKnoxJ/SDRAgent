# Commit: repo bootstrap
- Created repo `sdr-agent-internal`
- Added `/docs/PLAN.md` with end-to-end tutor-led plan
- Established folder structure for prompts/schemas/samples/knowledge

# Commit: output contracts
- Added `/schemas/sdr-package.schema.json` (master schema for SDR outputs)
- Added `/samples/campaign.input.sample.json` (seed input)
- Decision: all agent outputs must validate against these schemas

# Action: OpenAI Files & Vector Store
- Created Vector Store: sdr-agent-knowledge-base
- Uploaded persona sheets, case studies, sales frameworks, and product brochures
- Recorded all File IDs and Vector Store ID in `docs/files.md`
- Decision: This Vector Store will be linked to each campaign run for grounding

# Commit: prompts
- Added `/prompts/system.sdr.md` (role, style, workflow, grounding)
- Added `/prompts/fewshots.md` (example outputs)
- Decision: all campaign runs will use these prompts to standardise tone & structure
