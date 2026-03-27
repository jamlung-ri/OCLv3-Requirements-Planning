# Requirements To Do Tracker

> **Guidance from Jon:**
> - Separate documentation (immediate, solid, actionable) from planning (fluid, ambitious, on hold). Features may be MVP-for-v3 or aspirational — document accordingly.
> - After refining this documentation, supplement the human walkthrough of OCLv3 with some way for Claude to visualize OCLv3 on its own.

_Last updated: 2026-03-27 (cascade spec, in-list reference summary, expansion comparison, concept.md cleanup)_

---

## How to Use This File

- **Priority tiers:** P1 = must resolve before dev handoff | P2 = should resolve before handoff | P3 = post-v3 or low-urgency
- **Action types:** `DECIDE` = open design question | `WRITE` = content gap | `VERIFY` = cross-check against OCLv2 or implementation | `CLEANUP` = minor editorial fix | `SESSION` = needs dedicated design time with stakeholders
- Check the box and add a brief resolution note when done.

---

## Dedicated Sessions Required

These items have been explicitly called out as needing focused design time — not just writing. They are blocking or near-blocking for dev handoff.

- [ ] **`SESSION`** **Add Reference / Cascade Specification** — The cascade options and parameters are underspecified across multiple files. A dedicated design session should define: all cascade options and their exact parameters, the Q-AND-A cascade chain, and the in-list summary for grouped resolved resources. Feeds `01_objects/reference.md` and `03_workflows/build-concept-dictionary.md`.

- [ ] **`SESSION`** **Version Consistency / Collection Update Workflows** — The canonical source version locking behavior is deployed but not intelligently communicated to users. Needs design for: how the UI tells the user what it's doing (so version mismatches don't feel like bugs), the pathways offered when a mismatch is encountered, and whether HEAD-resolution during updates applies only to owned sources or also to CIEL. References: [ocl_issues#2236](https://github.com/OpenConceptLab/ocl_issues/issues/2236), [ocl_issues#2312](https://github.com/OpenConceptLab/ocl_issues/issues/2312).

- [ ] **`SESSION`** **Validation (TBv3 Design)** — How validation applies to sources (not just collections) and the Validation Report Panel for sources need dedicated design work before TBv3 planning. See `tbv3-knowledge-base.md` for context.

---

## P1 — Resolve Before Dev Handoff

### Open Architectural / Design Decisions

- [ ] **`DECIDE`** `03_workflows/build-concept-dictionary.md` + `02_capabilities/manage-references.md` — **HEAD references / Linked Sources:** OCL currently only allows adding concepts from released source versions. Collection managers who also own the source need a way to reference HEAD content. Decision so far: not MVP, carve out space for it. Two sub-questions remain:
  - [ ] **`DECIDE`** — **Own vs. not-own scope for HEAD-resolution:** Does HEAD-resolution during collection updates apply only to sources the user owns, or also to sources they do not own (e.g., CIEL)? Needed before the Update Collection workflow can be fully specced. (Noted as open in `manage-references.md`.)

### Blank / Incomplete Sections

### Key Business Rule Clarifications

- [ ] **`VERIFY`** `01_objects/reference.md` — **Version consistency rules:** Cross-check canonical source version locking behavior against OCLv2's current implementation and ADR-003. Backend is deployed and stable, but the UI must communicate what it's doing so it doesn't feel like a bug. Requires workflow design for version mismatch pathways. See Dedicated Sessions above.

- [ ] **`DECIDE`** `03_workflows/update-collection-source-version.md` — **Lock retired concept to pre-retirement version:** Add a third option (beyond remove/keep) allowing the user to lock their reference to the last CIEL version where the concept was still active. Needs a yes/no decision before the Update Collection workflow UI is built.

---

## P2 — Should Resolve Before Dev Handoff

### Cascade Specification

_Recommend handling as part of the dedicated Add Reference / Cascade session above._

- [x] **`WRITE`** `01_objects/reference.md` — **All cascade options and parameters:** Documented four presets (None, Source to Mappings, Source to Concepts/OpenMRS, Source to Concepts + OpenMRS Transform, Custom) with full parameter table (`method`, `map_types`, `cascade_levels`, `return_map_types`, `transform`). ✅ 2026-03-27

- [x] **`WRITE`** `01_objects/reference.md` — **In-list summary for grouped resolved resources:** Documented collapsible ▶ rows, concept/mapping grouping, orphaned mappings, two-tab detail panel (Reference details + View expansion), repository chip with tooltip, expansion selection context. ✅ 2026-03-27

### OCLv2 Behavior Documentation

- [ ] **`VERIFY`** `02_capabilities/author-concept.md` — **OCLv2 Add Mapping behavior:** Document OCLv2's current behavior for populating "Source name" via search, canonical URL handling, and external code entry. Needed before finalizing the v3 Add Mapping form design. (Flagged as a Verify note in the file.)
See resources\Add Mappings Recording 2026-03-27 135245.gif

- [x] **`VERIFY`** `01_objects/concept.md` — **Business rules vs. OpenMRS validation schema:** Names section split into OCL-platform rules vs. OpenMRS-schema-specific rules. ✅ 2026-03-27

- [x] **`VERIFY`** `01_objects/concept.md` — **OpenMRS Dictionary validation rules:** Full per-concept and dictionary-wide rule set integrated. Two rules flagged as not yet implemented in OCL validation. ✅ 2026-03-27

- [x] **`VERIFY`** `01_objects/concept.md` — **Display name resolution algorithm:** Confirmed 5-step fallback logic matches OCLv2 behavior; To Do comment removed. ✅ 2026-03-27

### Expansion Specification

- [ ] **`VERIFY`** `01_objects/expansion.md` — **Complete expansion parameters list + defaults:** Schema currently includes `active_only` (default true) and `repo_versions`. Confirm these are accurate and complete against oclapi2 implementation. Additional parameters (if any) can be added post-V3 via tbv3-knowledge-base.md.

- [x] **`WRITE`** `01_objects/expansion.md` — **Expansion comparison (full spec):** Documented entry points (checkbox selection per expansion row, Compare button activates at 2+ selected), summary statistics bar (Added/Removed/Unchanged counts), concept diff list, resolved repo versions display, and unchanged-collapse-by-default behavior. ✅ 2026-03-27

### Validation Specification

- [ ] **`VERIFY`** `02_capabilities/validate-content.md` — **Built-in rule categories accuracy:** Verify that the listed built-in rules are accurate and complete. Cross-reference with the OpenMRS validation schema and FHIR CodeSystem constraints.

### Repository Configuration and Schema

### Surfaces Review (Joe-Gated — Needs Current v3 Screenshots)

All four surface files need a reconciliation pass against current v3 designs. Joe to provide screenshots or Figma links before these can be completed.

- [ ] **`VERIFY`** `04_surfaces/concept-detail.md` — Review against current v3 implementation. Reconcile gaps.
- [ ] **`VERIFY`** `04_surfaces/repository-page.md` — Review against current v3 implementation.
- [ ] **`VERIFY`** `04_surfaces/versions-expansions-tab.md` — Review against current v3 implementation.
- [ ] **`VERIFY`** `04_surfaces/design-system.md` — Full design system review for accuracy and completeness.
- [ ] **`VERIFY`** `01_objects/repository.md` — Cross-check repository header attributes and version dropdown rules against the current v3 Version design and `04_surfaces/repository-page.md`.

### Remaining Content Gaps

- [ ] **`WRITE`** `01_objects/concept.md` — **Concept chip spec:** Add the concept chip display format (short and long form) to the UI Display Rules section. Cross-reference with `04_surfaces/design-system.md`.
  - See 'resources\Chip and Tooltip.png' for design, plus 'resources\Objects (Chip view) · Issue #1776 · OpenConceptLab_ocl_issues.pdf' for additional detail

---

## P3 — Post-v3 or Low Priority

- [ ] **`CLEANUP`** Workflow docs (`03_workflows/`) — **Remove all "SOW" mentions** throughout and replace with neutral language.

- [ ] **`WRITE`** `01_objects/concept.md` — **Hierarchy attributes:** Add `parent_concept_url` and any related hierarchy attributes to the concept schema once the hierarchy model is confirmed.

- [ ] **`WRITE`** `05_decisions/adrs.md` — **New ADRs:** Jon to review all existing ADRs for accuracy and add new records for: Linked Sources design, Canonical URL registry approach, and collection-level validation schema. (Flagged at the top of `adrs.md`.)

---

## Items That Can Be Dismissed

These were flagged inline but are already adequately addressed elsewhere:

| To Do Location | Reason for Dismissal |
|---|---|
| `expansion.md` — stale expansion behavior | Fully specced in `02_capabilities/manage-versions-and-expansions.md` under Staleness Indicator |
| `repository.md` — version dropdown: do not wrap | Already specified in `04_surfaces/versions-expansions-tab.md` and `01_objects/version.md` |
| `manage-references.md` — bulk transform via checkbox | Already documented in the Bulk Transform section of `02_capabilities/manage-references.md` |

---

## Resolved (Documentation Pass — 2026-03-27)

| Item | File(s) Updated | Notes |
|---|---|---|
| ADR-002: Comparison as Bottom Drawer (Pending) | `adrs.md` | Accepted as independent component — modal/drawer/full page |
| ADR-005: Default Repo Version on Navigation (Pending) | `adrs.md` | Accepted; Edit Mode noted as post-v3 |
| Expansion list layout (two-panel vs. OCLv2) | `manage-versions-and-expansions.md`, `versions-expansions-tab.md` | Two-panel confirmed; version rows now show default expansion + resolved repo versions |
| Concept proposals off by default | `concept-proposal.md`, `manage-concept-proposals-workflow.md` | Off by default; on when enabled, open to any authenticated user |
| Source admin can edit proposals before approving | `concept-proposal.md` | Edit history tracked; proposer notified "Accepted with edits" |
| Proposer view for outstanding proposals (empty section) | `concept-proposal.md` | Written as GitHub issue-style ticket |
| Version skipping (squash terminology) | `update-collection-source-version.md` | "Skip" language adopted; direct v1→v4 documented |
| Selective acceptance as norm (Added concepts) | `update-collection-source-version.md` | Default changed to none-selected; tailored modified-concepts sub-view added |
| Tailored view of modified concepts in collection | `update-collection-source-version.md` | Added as prominently surfaced sub-view |
| Auto-expand save-as-version behavior | `expansion.md`, `manage-versions-and-expansions.md` | Copy as-is default; re-evaluate option in Create Version dialog |
| Expansion comparison stub | `expansion.md` | Section added referencing issue #1853; full spec still needed (P2) |
| Mapping target types (within-source / cross-source / external) | `mapping.md` | Three-way distinction documented; external code system model deferred |
| Mapping from/to field combination rules | `mapping.md` | Documented based on Quick Add Mappings behavior |
| Sort weight UX | `author-concept.md` | Removed from form; drag-and-drop reorder documented |
| Before/after comparison for transforms | `manage-references.md` | Added to transform preview dialog spec |
| Source version creation processing state | `manage-versions-and-expansions.md` | Documented as async processing state in Create Version flow |
| Validation rule configurability (v3 scope) | `validate-content.md`, `tbv3-knowledge-base.md` | Not in scope for v3; deferred |
| Validation Report Panel for Sources | `validate-content.md`, `tbv3-knowledge-base.md` | Queued as TBv3 dedicated topic |
| Reviewer approval reason/comment | `manage-concept-proposals-workflow.md` | Added to approval flow |
| Full Page vs. Split View only | `concept-detail.md` | One component; Split View default; Full Page via direct URL |
| $match operation for proposers | `tbv3-knowledge-base.md` | Deferred post-v3 |
| Group similar proposals for admins | `tbv3-knowledge-base.md` | Deferred post-v3 |
| Post-approval actions for proposer | `tbv3-knowledge-base.md` | Deferred post-v3 |
| Public-facing proposal view | `tbv3-knowledge-base.md` | Deferred post-v3 |
| Transform to update pinned version | `tbv3-knowledge-base.md` | Deferred post-v3 |
| Dropdown config linked to OCL source | `tbv3-knowledge-base.md` | Deferred post-v3 |
| Extra attribute template configuration | `tbv3-knowledge-base.md` | Deferred post-v3 |
| Repository Subtypes (ConceptMap, CodeSystem) | `tbv3-knowledge-base.md` | Deferred post-v3 |
| Additional mapping list filters | `mapping.md` | Added to UI Display Rules |
| Cascade options and parameters | `reference.md` | All presets documented with full parameter table |
| In-list summary for grouped resolved resources | `reference.md` | Collapsible rows, two-tab detail panel, repo chip spec |
| Expansion comparison (full spec) | `expansion.md` | Entry points, summary stats, diff list, unchanged collapse |
| Business rules vs. OpenMRS validation schema | `concept.md` | Names split into OCL-platform vs. OpenMRS-schema sections |
| Display name resolution algorithm | `concept.md` | Confirmed; To Do comment removed |
| OpenMRS Dictionary validation rules | `concept.md` | Full per-concept and dictionary-wide rule set; two rules flagged as not yet implemented |

---

## Summary

| Priority | Open | Resolved / Deferred |
|---|---|---|
| Dedicated Sessions | 3 | — |
| P1 | 3 | 8 |
| P2 | 8 | 24 |
| P3 | 3 | 3 |
| Dismissed | — | 3 |
| **Total active** | **14** | **38** |

| Action Type | Open |
|---|---|
| SESSION | 3 |
| DECIDE | 3 |
| WRITE | 2 |
| VERIFY | 5 |
| CLEANUP | 1 |
