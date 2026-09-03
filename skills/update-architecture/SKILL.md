---
name: update-architecture
description: Update the living architecture document and republish it. Use when the user says "update the architecture doc", "record this decision in the doc", or describes a pipeline/process change that should be documented.
---

# Update the architecture document

Source of truth: `docs/architecture.html` in this project.
Published version: https://claude.ai/code/artifact/c251137b-d797-4c6a-baaa-13e9788230e7

## Steps

1. **Read** `docs/architecture.html` before editing — understand the existing
   structure (layers, gates, tool map, roadmap, open decisions) and edit in
   place; don't restructure unless asked.
2. **Apply the change** where it belongs:
   - a decision made → resolve the matching item in "Decisions to make"
     (mark it decided, state the choice)
   - a pipeline/process change → update the relevant layer card AND the
     mermaid master-flow diagram so they stay consistent
   - a new tool or credential → update the "Tool map" table status
   - scope changes → update the roadmap phases and the lede if needed
3. **Keep the doc honest**: update the "Drafted/updated" date line in the
   header; never delete the compliance callouts or gate ordering without an
   explicit instruction — those encode safety decisions.
4. **Republish** with the Artifact tool, passing the published URL above as
   `url` so the link stays stable. If republishing to that URL fails (the
   artifact belongs to another account), publish as a new artifact and update
   the URL in this skill, `README.md`, and tell the user the link changed.
5. **Summarize** to the user what changed in the doc, in one or two sentences.
