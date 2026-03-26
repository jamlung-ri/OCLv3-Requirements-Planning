# Requirements To Do Tracker

Suggestions from Jon:
* Separate between documentation and planning very clearly - there will be aspects that are an MVP feature for v3 vs. how we intend to get to the full feature over time. Documentation is immediate, solid, and actionable. Planning is fluid, ambitious, but generally on hold. 
   * Example: Canonical URL requirements will touch everything. Building the approach out should be done without hitting the documentation badly. Planning will require documents, specs, etc.
* Consider: After refining this documentation, supplement the human walkthrough of OCLv3 with some way of Claude actually visualizing Oclv3 on its own.

This file consolidates every `To Do` comment found across the requirements repository as of commit `54e9107` (March 19, 2026). Each item is categorized by priority, file location, and recommended action type.

---

## How to Use This File

- **Priority tiers:** P1 = must resolve before dev handoff | P2 = should resolve before handoff | P3 = post-v3 or low-urgency
- **Action types:** `DECIDE` = open design question requiring a judgment call | `WRITE` = content gap needing new content | `VERIFY` = cross-check against OCLv2 behavior or existing implementation | `CLEANUP` = minor editorial/cosmetic fix
- When an item is resolved, check the box and add a brief resolution note.

---

## P1 — Resolve Before Dev Handoff

These are open questions or blank sections that could affect implementation scope, misalign dev work, or leave a spec incomplete in a way that blocks design or engineering.

### Open Architectural / Design Decisions

- [ ] **`DECIDE`** `03_workflows/build-concept-dictionary.md` — **HEAD references in collections (Linked Sources concept):** OCL currently only allows adding concepts from *released* source versions to a collection. Collection managers who also own the source need a way to reference HEAD content. Options: enable HEAD-resolution for sources the user owns, a "Linked Source" pattern, or a working collection concept. Has scope and data model implications.
  - Decision: Standard is to not use HEAD, but we'll carve out a space to use this. It's not just one feature, and it's not MVP.

  - [ ] **`DECIDE`** `02_capabilities/manage-references.md` — **Resolve to HEAD — own vs. not-own source:** Clarify whether HEAD-resolution during updates applies only to sources the user owns, or also to sources they do not own (e.g., CIEL).

- [ ] **`DECIDE`** `03_workflows/update-collection-source-version.md` — **Version skipping:** When a source goes v1 → v2 → v3 → v4, does the collection update workflow require stepping through each intermediate version, or can the user squash and go directly v1 → v4? Affects workflow UX and diff computation logic.
  - Decision: Go straight from v1 --> v4. Also don't call this squashing - call it skipping.

- [ ] **`DECIDE`** `05_decisions/adrs.md` — **ADR-002 (Comparison as Bottom Drawer) is still "Pending".** Needs a formal acceptance decision before the comparison surface is built.
  - Decision: This component should be independent and not necessarily tied to a place on the screen. Could be a model, a drawer, a full page, etc. 

- [ ] **`DECIDE`** `05_decisions/adrs.md` — **ADR-005 (Default Repository Version on Navigation) is still "Pending".** Also: the inline To Do suggests a new approach — an explicit "Edit" action on the repo that switches the user to HEAD, rather than requiring manual version dropdown navigation. Decide whether to adopt this and update the ADR.
  - Decision: Keep it simple for now, but Edit Mode will be a thing in the future.

- [ ] **`DECIDE`** `02_capabilities/manage-versions-and-expansions.md` — **Expansion list layout:** Should TBv3 use the two-panel layout (versions left, expansions for selected version right) as currently specced, or continue OCLv2's approach of showing all expansions for all versions together? This affects `04_surfaces/versions-expansions-tab.md`.
  - Decision: We shouldn't be hiding expansions. Needs design work. Overall, expansions should be a child of versions, where versions are at the forefront but it's clear there's an expansion if you want to look into it more. Also, a version should be clear about what expansion it's using, what resolved repo versions are in it, etc.

### Blank / Incomplete Sections

- [ ] **`WRITE`** `01_objects/concept-proposal.md` — **Proposer view for outstanding proposals** — this section header exists but the body is completely empty. Must specify: how a TI reviews their submitted proposals, what they see, what actions are available (withdraw, comment, track status). 
  - Decision: Model this simply, like a GitHub issue ticket or something. Proposal can have narrative (the concept and mappings, what to adjust, etc.), have discussion, then approve/reject. Submissions have to go toward something though - likely to the repo for now. It should refer to a concept (most common) or mapping (can be referenced independently or bundled in with concept). 
  - Outstanding decision: There might be a bit of this that's outside of OCL Core. This first pass is the formal submission system that needs to be simple though. The other is an informal submission system.

- [ ] **`WRITE`** `03_workflows/build-concept-dictionary.md` — **Path C: bulk add from saved list** — add a step covering the case where a user has a list of concept IDs and wants to add them all at once by specifying a source/version. Also note: duplicate references with the same extensional/intensional target are not added.
  - See documentation here: https://github.com/OpenConceptLab/ocl_issues/issues/1359 and here: https://github.com/OpenConceptLab/ocl_issues/issues/1361

### Key Business Rule Clarifications

- [ ] **`DECIDE`** `01_objects/concept-proposal.md` — **Concept proposals off by default:** Confirm and document that concept proposals must be explicitly enabled by a source admin, and are not active by default for all sources. Propagate this into `03_workflows/manage-concept-proposals-workflow.md`.
  - Decision: Yes, they are off by default. If turned on, they're open to any authenticated user (for now).

- [ ] **`DECIDE`** `01_objects/concept-proposal.md` — **Source admin can edit proposals before approving:** Confirm this is the intended behavior. Document any constraints (e.g., is the edit history tracked? is the proposer notified of edits?).
  - Decision: Yes, source admin can edit before approving. The proposed concept's edit history (i.e. proposer's proposed concept version --> source admin's accepted concept version) should be tracked, and the proposer should be notified when their proposal is accepted (edits to the proposal are supplemental info e.g. "Accepted with edits")

- [ ] **`VERIFY`** `01_objects/reference.md` — **Version consistency rules:** Cross-check the "canonical source version" locking behavior against OCLv2's current implementation and ADR-003. Resolve any discrepancies before dev handoff.
  - Current state: Backend capability is out there and deployed, and seems stable enough to continue forward. Just make sure that it tells the user what it's doing, so that it doesn't feel like a bug when really it's just an imperfectly modeled collection
      - Likely requires a couple of workflows to be implemented for collections with older CIEL version concepts when the user tries to add a concept from the latest CIEL version. If someone encounters this "bug", then we want to offer them the pathway(s) to resolve it.
      - This is one of multiple workflow design areas that deserve dedicated time in the next couple of months.
  - Review documentation here: https://github.com/OpenConceptLab/ocl_issues/issues/2236 and UI requirements here: https://github.com/OpenConceptLab/ocl_issues/issues/2312



---

## P2 — Should Resolve Before Dev Handoff

Gaps in coverage, specification completeness, or cross-references that should be closed before engineering, but don't block architectural decisions.

### Cascade Specification (Underspecified Across Multiple Files)

- [ ] **`WRITE`** `01_objects/reference.md` — **All cascade options and parameters:** Document all available cascade options and their exact parameters. Currently only `sourcemappings` is named; others are referenced but unspecified.
  - Add reference workflow is an area that deserves dedicated design time in the near future.

- [ ] **`WRITE`** `03_workflows/build-concept-dictionary.md` — **Cascade label and preview for Q-AND-A sets:** Update the Add to Collection dialog to say "Source-to-Concept cascade" rather than just "Source Mappings". Ensure the preview flow shows the full cascade chain (question → answers → their mappings).
  - See line 1061 of this file for requirements: tbv3\tbv3-knowledge-base.md

- [ ] **`WRITE`** `01_objects/reference.md` — **In-list summary for grouped resolved resources:** Describe how the References list shows a collapsed/expanded summary of resolved concepts/mappings grouped under each reference row.
  - Might be in knowledge base? tbv3\tbv3-knowledge-base.md

### OCLv2 Behavior Documentation

- [ ] **`VERIFY`** `02_capabilities/author-concept.md` — **OCLv2 Add Mapping behavior:** Document OCLv2's current behavior for populating "Source name" via search, canonical URL handling, and external code entry. Needed before finalizing the v3 Add Mapping form design.

- [ ] **`VERIFY`** `01_objects/concept.md` — **Business rules vs. OpenMRS validation schema:** Tag each business rule in `concept.md` as OCL-platform-level or OpenMRS-schema-specific. Affects what is enforced universally vs. conditionally.

- [ ] **`VERIFY`** `01_objects/concept.md` — **OpenMRS Dictionary validation rules:** Fill in the incomplete OpenMRS schema validation section with the actual rules (fully specified name per locale, coded concepts require Q-AND-A or CONCEPT-SET mapping, etc.).
  - See documentation here: https://docs.openconceptlab.org/en/latest/oclapi/openmrsvalidationschema.html (and respective spots in code) and here: https://talk.openmrs.org/t/defining-our-concept-validation-rules-for-ocl/33508 

- [ ] **`VERIFY`** `01_objects/concept.md` — **Display name resolution algorithm:** Confirm the 5-step fallback logic documented in `concept.md` matches OCLv2 behavior.

### Expansion Specification

- [ ] **`VERIFY`** `01_objects/expansion.md` — **Complete expansion parameters list:** Fill in all supported parameters (currently flagged as incomplete).
  - Check oclapi2 repo for supported expansion parameters

- [ ] **`VERIFY`** `01_objects/expansion.md` — **Default auto-expansion parameters:** Confirm exact defaults for `active_only`, `repo_versions`, and any other parameters.
  - Check oclapi2 repo for supported expansion parameters

- [ ] **`DECIDE`** `01_objects/expansion.md` — **Auto-expand save-as-version behavior:** Clarify whether saving a collection version copies the HEAD auto-expansion as-is (frozen snapshot) or re-evaluates all references at the moment of version creation.
  Decision: Default behavior will be to copy HEAD auto-expansion as-is, but user can elect to re-evaluate all references instead.

- [ ] **`WRITE`** `01_objects/expansion.md` — **Expansion comparison:** Add a section covering how two expansions are compared (entry points, what is shown, summary statistics). Cross-reference with `02_capabilities/compare-resources.md`.
  - See documentation here (along with linked documents within the ticket): https://github.com/OpenConceptLab/ocl_issues/issues/1853

### Mapping Specification Gaps

- [ ] **`DECIDE`** `01_objects/mapping.md` — **Within-source internal mapping distinction (Q&A):** Clarify and document the three-way distinction: (a) mappings within the same source (Q-AND-A, CONCEPT-SET), (b) mappings between different OCL sources, (c) mappings to external code systems. Update the "Internal vs. External" section.
  - Get documentation from https://github.com/OpenConceptLab/ocl_issues/issues/1972, but this does not cover mappings to "external" code systems. Let's document this as unfinished and out of scope for now.

- [ ] **`VERIFY`** `01_objects/mapping.md` — **From/To field combination rules:** Document which field combinations are valid for a mapping (e.g., when is `to_concept_url` required vs. optional? What is the minimum required for an internal vs. external mapping?).
  - Learn from the experience used in the Quick Add Mappings feature in the Concept Details --> Associations panel. Overall, `to_concept_url` is enough on its own. If only a concept ID is provided, it needs to also have a source URL to point to (with an optional source version id). Either way, a concept name can be provided to "overwrite" OCL's resolved concept name.

### Validation Specification

- [ ] **`DECIDE`** `02_capabilities/validate-content.md` — **Validation rule configurability:** Decide whether v3 should include user-defined validation schemas and per-rule severity overrides, or defer post-v3. Document the decision.
  - Not in scope for v3 launch - document in TBv3 knowledge base: tbv3\tbv3-knowledge-base.md

- [ ] **`VERIFY`** `02_capabilities/validate-content.md` — **Built-in rule categories accuracy:** Verify that the listed built-in rules are accurate and complete. Cross-reference with the OpenMRS validation schema and any FHIR CodeSystem constraints.

- [ ] **`DECIDE`** `02_capabilities/validate-content.md` — **Validation Report Panel for Sources:** The panel is currently described only for collections. Confirm and document how validation applies to source authoring (concept/mapping editor context) as well.
  - Queue up validation as a dedicated topic for TBv3.

### Concept Proposals Enhancements

- [ ] **`WRITE`** `03_workflows/manage-concept-proposals-workflow.md` — **$match operation for proposers:** Document how the proposal form uses OCL's `$match` operation to surface potential existing concepts before submission, reducing duplicate proposals.
  - Not in scope for v3 launch - document in TBv3 knowledge base: tbv3\tbv3-knowledge-base.md

- [ ] **`DECIDE`** `03_workflows/manage-concept-proposals-workflow.md` — **Group similar proposals for admins:** Add a mechanism for admins to detect and group duplicate/similar proposals in the review queue before processing.
  - Not in scope for v3 launch - document in TBv3 knowledge base: tbv3\tbv3-knowledge-base.md

- [ ] **`WRITE`** `03_workflows/manage-concept-proposals-workflow.md` — **Reviewer approval reason/comment:** Add the ability for a reviewer to include an optional reason or comment when *approving* (not just when rejecting).

- [ ] **`WRITE`** `02_capabilities/manage-concept-proposals.md` — **Post-approval actions for proposer:** Add actions available to the proposer after their concept is approved — e.g., view the new concept, add it to their collection, etc.
- Not in scope for v3 launch - document in TBv3 knowledge base: tbv3\tbv3-knowledge-base.md

### Update Collection Workflow

- [ ] **`WRITE`** `03_workflows/update-collection-source-version.md` — **Tailored view of modified concepts in collection:** Add a sub-view that shows only CIEL concepts that were *modified AND already referenced by this collection* — the most actionable portion of the diff.

- [ ] **`WRITE`** `03_workflows/update-collection-source-version.md` — **Selective acceptance is the norm:** Add a design note that most users will NOT want to accept newly added CIEL concepts wholesale — the UI should not make bulk-accept the easiest path. Default behavior should favor cautious/selective acceptance.

- [ ] **`DECIDE`** `03_workflows/update-collection-source-version.md` — **Lock retired concept reference to pre-retirement version:** Add a third option (beyond remove/keep) that lets the user lock their reference to the last CIEL version where the concept was still active.

### Author Concept Gaps

- [ ] **`DECIDE`** `02_capabilities/author-concept.md` — **Sort weight UX:** Remove sort weight from the per-mapping form field. Decide and document a better interaction pattern (e.g., drag-and-drop reordering within the Associations panel, or an edit-order mode).
  - Decision: v2 already implements this. See code and see documentation here: https://github.com/OpenConceptLab/ocl_issues/issues/1682

- [ ] **`DECIDE`** `02_capabilities/author-concept.md` — **Dropdown config linked to OCL source/collection:** Document how users can configure concept class / datatype / map type dropdowns by pointing to an existing OCL source rather than managing values manually. Specify what OCL's default sources are for each field type.
  - Not in scope for v3 launch - document in TBv3 knowledge base: tbv3\tbv3-knowledge-base.md

### Reference Transforms

- [ ] **`DECIDE`** `02_capabilities/manage-references.md` — **Transform to update pinned version:** Consider adding a transform that lets a user update a repo-versioned reference from `v2023` to `v2024` without changing the reference pattern. Decide whether to include in v3 or defer.
- Not in scope for v3 launch - document in TBv3 knowledge base: tbv3\tbv3-knowledge-base.md

- [ ] **`WRITE`** `02_capabilities/manage-references.md` — **Before/after comparison for transforms:** Add a comparison view shown before a transform is applied, so the user can see the old vs. new reference expression side by side.

### Repository Configuration and Schema

- [ ] **`WRITE`** `03_workflows/build-concept-dictionary.md` — **Validation schema setup in collection creation:** Add a step in "Create the Collection" covering validation schema selection, including common presets (OpenMRS) and custom configuration.

- [ ] **`WRITE`** `03_workflows/author-and-publish-source.md` — **Canonical URL encouragement:** Add a UI nudge (warning or banner) for sources that don't have a canonical URL set. Document this guidance in the workflow.

- [ ] **`WRITE`** `01_objects/repository.md` — **Extra attribute template configuration:** Document how a repository can define templates for Extra attribute keys (pre-defined keys with types and labels).
- Not in scope for v3 launch - document in TBv3 knowledge base: tbv3\tbv3-knowledge-base.md

- [ ] **`DECIDE`** `01_objects/repository.md` — **Repository Subtypes table:** Resolve whether ConceptMap belongs as a Source subtype, a Collection subtype, or both — and whether CodeSystem deserves a top-level repo type distinct from Source/Collection. Update the table.
  - Not in scope for v3 launch - document in TBv3 knowledge base: tbv3\tbv3-knowledge-base.md

### Surfaces Review (Flagged as Incomplete in Commit Message)

- [ ] **`VERIFY`** `04_surfaces/concept-detail.md` — **Review against current v3 implementation:** Joe to provide current v3 screenshots/designs for comparison. Reconcile gaps.

- [ ] **`DECIDE`** `04_surfaces/concept-detail.md` — **Full Page vs. Split View only:** Decide whether to eliminate the Full Page concept detail view and commit to Split View only. Update the spec accordingly.
  - Decision: Deserves its own component, which will generally display via Split View but could be shown in a full page that's accessed via URL.

- [ ] **`VERIFY`** `04_surfaces/repository-page.md` — **Review against current v3 implementation:** Joe to provide current v3 screenshots/designs for comparison.

- [ ] **`VERIFY`** `04_surfaces/versions-expansions-tab.md` — **Review against current v3 implementation:** Joe to provide current v3 screenshots/designs for comparison.

- [ ] **`VERIFY`** `04_surfaces/design-system.md` — **Full design system review:** Joe to review the entire doc for accuracy and completeness against current v3 designs.

- [ ] **`VERIFY`** `01_objects/repository.md` — **Repository header vs. new Version design:** Cross-check header attributes and version dropdown rules against the current v3 Version design and `04_surfaces/repository-page.md`.

- [ ] **`WRITE`** `01_objects/concept.md` — **Concept chip spec:** Add the concept chip display format (short and long form) to the UI Display Rules section. Cross-reference with `04_surfaces/design-system.md`.

---

## P3 — Post-v3 or Low Priority

- [ ] **`CLEANUP`** `03_workflows/build-concept-dictionary.md` — **Remove all "SOW" mentions** throughout workflow docs. Replace with neutral language.

- [ ] **`DECIDE`** `02_capabilities/manage-concept-proposals.md` — **Public-facing proposal view:** Explore whether non-admins should be able to see and endorse other users' proposals (community visibility / upvoting). Defer if not in current scope.
  - Not in scope for v3 launch - document in TBv3 knowledge base: tbv3\tbv3-knowledge-base.md

- [ ] **`WRITE`** `01_objects/concept.md` — **Hierarchy attributes:** Add `parent_concept_url` and any related hierarchy attributes to the concept schema once the hierarchy model is confirmed.

- [ ] **`WRITE`** `02_capabilities/manage-versions-and-expansions.md` — **Source version creation processing state:** Add a note that source version creation also has an async processing state (indexing, export generation), visible in the UI.

- [ ] **`WRITE`** `01_objects/mapping.md` — **Additional mapping list filters:** Document additional filter options for the Mappings list view beyond map_type, from/to source, and retired status.

- [ ] **`WRITE`** `05_decisions/adrs.md` — **Jon to review all ADRs; add additional ADRs** for open architectural questions surfaced by the To Dos above (particularly: Linked Sources, version squashing, proposal defaults, collection-level validation schema).

---

## Items That Can Be Dismissed

These were flagged inline but are already adequately addressed elsewhere in the documentation:

| To Do Location | Reason for Dismissal |
|---|---|
| `expansion.md` — stale expansion behavior | Fully specced in `02_capabilities/manage-versions-and-expansions.md` under Staleness Indicator |
| `repository.md` — version dropdown: do not wrap | Already specified in `04_surfaces/versions-expansions-tab.md` and `01_objects/version.md` |
| `manage-references.md` — bulk transform via checkbox | Already documented in the Bulk Transform section of `02_capabilities/manage-references.md` |

---

## Summary

| Priority | Items |
|---|---|
| P1 | 10 |
| P2 | 32 |
| P3 | 6 |
| Dismissed | 3 |
| **Total** | **51** |

| Action Type | Count |
|---|---|
| DECIDE | 19 |
| WRITE | 19 |
| VERIFY | 10 |
| CLEANUP | 3 |
