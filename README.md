# OCL TermBrowser v3 — Requirements Library

This library is the authoritative source for TBv3 requirements. It is structured for use by AI assistants, developers, designers, and testers.

## Structure

```
00_context/           ← Always load these as base context
  product-vision.md   ← Why TBv3 exists; success criteria; milestones
  user-roles.md       ← Who uses TBv3; permissions; auth behavior
  glossary.md         ← OCL terminology; use these terms precisely

01_objects/           ← What the system knows about (stable; load what is relevant)
  concept.md
  mapping.md
  repository.md
  version.md
  reference.md
  expansion.md
  concept-proposal.md

02_capabilities/      ← What users can do; prescriptive UI + business rules
  search-and-filter.md
  browse-repository.md
  author-concept.md
  manage-references.md
  manage-versions-and-expansions.md
  manage-concept-proposals.md
  compare-resources.md
  configure-repository.md

03_workflows/         ← End-to-end user journeys; sequences of capabilities
  discover-terminology.md
  build-concept-dictionary.md
  add-to-existing-dictionary.md
  update-collection-source-version.md
  author-and-publish-source.md
  manage-concept-proposals-workflow.md

04_surfaces/          ← Page and component-level UI specs
  concept-detail.md
  design-system.md

05_decisions/         ← Why we made key architectural choices
  adrs.md
```

## How to Use This Library

### For a UI feature
Load: `00_context/glossary.md` + relevant `01_objects/` + relevant `02_capabilities/` + `04_surfaces/design-system.md`

### For an end-to-end user story
Load: `00_context/` (all three) + relevant `03_workflows/` + relevant `02_capabilities/`

### For API / backend work
Load: `00_context/glossary.md` + relevant `01_objects/` (schemas + business rules + API patterns)

### For test case generation
Load: relevant `03_workflows/` (entry conditions + error cases) + relevant `02_capabilities/` (validation rules + permissions)

### For design review
Load: `04_surfaces/design-system.md` + relevant `02_capabilities/` + relevant `03_workflows/`

## SOW Traceability

| SOW Item | Tracker | Covered In |
|---|---|---|
| Collection Versions + Expansions Tab | 48 | `02_capabilities/manage-versions-and-expansions.md`, `04_surfaces/` (TBD) |
| Concept Proposals MVP | 27 | `01_objects/concept-proposal.md`, `02_capabilities/manage-concept-proposals.md`, `03_workflows/manage-concept-proposals-workflow.md` |
| Add to Collection (from outside) | 49 | `02_capabilities/manage-references.md` → "Adding from Outside" |
| New Reference (from inside) | 50 | `02_capabilities/manage-references.md` → "Creating from Inside" |
| Reference preview | 64, 58 | `02_capabilities/manage-references.md` → "Reference Preview" |
| Resolve to HEAD on updates | 60 | `02_capabilities/manage-references.md` → "Linked Source", `03_workflows/update-collection-source-version.md` |
| Reference transforms (HEAD only) | 61 | `02_capabilities/manage-references.md` → "Reference Transforms" |
| Dependency change notifications | 62 | `02_capabilities/manage-versions-and-expansions.md` → "Dependency Notifications" |
| Review + accept upstream updates | 63 | `03_workflows/update-collection-source-version.md` |
| Version locking controls | 1 | `02_capabilities/manage-versions-and-expansions.md` → "Version Locking Controls", `05_decisions/adrs.md` ADR-003 |
