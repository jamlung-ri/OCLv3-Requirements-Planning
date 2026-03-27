# TBv3 Knowledge Base — Post-v3 Features and Deferred Design Topics

This file documents features, design ideas, and open questions that are explicitly **out of scope for v3 launch** but are worth preserving for future planning cycles. Each entry is a stub — enough context to resume the discussion without losing the original intent.

---

## Validation

### User-Defined Validation Schemas and Per-Rule Severity
**Origin:** `02_capabilities/validate-content.md`

Allow source/collection admins to define their own validation schemas (custom rule sets) and override the severity level of individual built-in rules (e.g., downgrade an Error to a Warning for their use case). The v3 system uses a fixed set of built-in rules only.

**Relevant context:** The `validate-content.md` file describes a full rule list UI with toggles and severity selectors — this is the intended future state. For v3, only predefined schema templates (OpenMRS, FHIR) are supported.

---

### Validation Report Panel for Sources
**Origin:** `02_capabilities/validate-content.md`

The Validation Report Panel is currently specified only for collections. How validation applies in the source authoring context (concept/mapping editor), and what a source-level report panel looks like, requires dedicated design work. This should be queued as a TBv3 topic.

---

## Concept Proposals

### `$match` Operation for Proposers
**Origin:** `03_workflows/manage-concept-proposals-workflow.md`

Before submitting a proposal, the proposal form should use OCL's `$match` operation to surface potential existing concepts that match the proposed concept — reducing duplicate proposals. The proposer sees "Did you mean one of these?" before finalizing their submission.

---

### Grouping Similar Proposals for Admins
**Origin:** `03_workflows/manage-concept-proposals-workflow.md`

A mechanism for source admins to detect and group duplicate or similar proposals in the review queue (e.g., multiple users proposing the same concept in different locales). Could be based on name similarity scoring or the same `$match` logic used for proposers.

---

### Post-Approval Actions for Proposers
**Origin:** `02_capabilities/manage-concept-proposals.md`

After a concept proposal is approved, give the proposer direct actions from the proposal detail page: view the newly created concept, add it to one of their collections, clone it to their own source, etc. Currently the proposer receives a notification with a link to the concept but no additional workflow from the proposal UI.

---

### Public-Facing Proposal View (Community Visibility / Upvoting)
**Origin:** `02_capabilities/manage-concept-proposals.md`

Allow non-admin, non-proposer users to view a source's open proposals and endorse or upvote them — giving source admins signal about community demand. This is a more open/community-model approach than the current proposer-to-admin flow.

---

## References

### Transform: Update Pinned Source Version
**Origin:** `02_capabilities/manage-references.md`

A transform that lets a user update the pinned version on a repo-versioned reference (e.g., from `orgs/CIEL/sources/CIEL/2023-01-01/` to `orgs/CIEL/sources/CIEL/2024-08-01/`) without changing the reference pattern or the concept it points to. Useful for collections that have deliberately pinned to a specific CIEL version and want to advance the pin without going through the full Update Collection workflow.

---

## Repository Configuration

### Extra Attribute Template Configuration
**Origin:** `01_objects/repository.md`

Allow a repository to define templates for Extra attribute keys — pre-defined keys with data types and human-readable labels. This would make the "Extras" section of concept/mapping forms more structured and consistent within a repository. Currently Extras are fully freeform.

---

### Repository Subtypes: ConceptMap and CodeSystem
**Origin:** `01_objects/repository.md`

Two open questions about how FHIR-aligned repository types map to OCL's Source/Collection model:
1. **ConceptMap**: Should it be a Source subtype, a Collection subtype, or valid as both? Currently modeled as a Collection subtype.
2. **CodeSystem**: Should it be a distinct top-level repo type (separate from Source), or remain a Source subtype? Currently modeled as a Source subtype (equivalent to FHIR CodeSystem).

These decisions have implications for FHIR API conformance and should be revisited before TBv3 introduces deep FHIR support.

---

## Dropdown Configuration

### Link Dropdowns to an OCL Source or Collection
**Origin:** `02_capabilities/author-concept.md`

Instead of manually managing dropdown values for concept class, datatype, name types, and map types in repository settings, allow owners to point to an existing OCL source or collection as the authoritative list of values. This enables centralized vocabulary management — e.g., an organization maintains a "Concept Classes" source and all their repos reference it.

OCL's own default sources for each field type also need to be documented.

---

## Linked Sources

### HEAD-Resolution Scope: Own vs. Not-Own Sources
**Origin:** `02_capabilities/manage-references.md`, `03_workflows/build-concept-dictionary.md`

When a collection is being updated and HEAD-resolution is used (to pick up new content before locking to a released version), it is not yet decided whether this applies:
- Only to sources the collection owner also owns (e.g., their own custom source)
- Or also to sources they do not own (e.g., CIEL)

The Linked Source pattern (allowing HEAD references for sources the user owns) is also not MVP. These questions deserve a dedicated design session with implications for the Update Collection workflow and the data model.

---
