# Workflow: Author and Publish a Source

## Purpose

A Terminology Publisher (TP) creates and maintains a canonical source of concepts and mappings — for example, a national health authority publishing a local code system, or the CIEL team maintaining and releasing new versions of CIEL.

---

## Roles

- **Primary actor**: Terminology Publisher (source owner/admin)
- **Supporting actors**: Community submitters (concept proposals), system (validation, versioning, notifications)

---

## Entry Conditions

- User is authenticated and owns an Organization (or operates in their user workspace)
- For new source: no prior source exists
- For ongoing maintenance: a source with an existing HEAD version

---

## Steps

### Phase 1: Create and Configure the Source (one-time setup)

1. Navigate to org or user profile → "New Repository" → Source
2. Configure:
   - ID, Full Name, Description
   - Source Type (e.g., "Dictionary")
   - Custom Validation Schema (e.g., "OpenMRS" for CIEL)
   - **Canonical URL** — strongly recommended for all sources. Required for FHIR-compatible sources and for registration in OCL's Global Canonical URL Registry. If left blank, a yellow banner is shown on the source page:
     > ⚠️ **No canonical URL set.** This source will not be accessible via FHIR endpoints and cannot be registered in the global registry. [Add a canonical URL →]
     (To Do: Add a similar warning for Collections too)
   - Default Locale and Supported Locales
   - Hierarchy Meaning (if applicable)
   - Auto-ID Assignment (if auto-assigning concept IDs)
3. Configure Dropdown Vocabulary (Settings → Dropdowns):
   - Concept Class values
   - Datatype values
   - Name Types
   - Map Types
4. Write the About page (Settings → About): describes the source's purpose, scope, and how to use it

---

### Phase 2: Author Concepts

#### Adding a New Concept
1. Source → Concepts tab → "New Concept"
2. Fill in form: class, datatype, names, descriptions, extras
3. If using templates: select a concept template to pre-fill common fields
4. Save → concept appears in HEAD with inline confirmation

#### Adding Mappings
1. From the newly created concept detail: "Add Mapping"
2. Select map type (e.g., SAME-AS SNOMED CT)
3. If mapping to an internal OCL concept: search and select
4. If mapping to an external code: enter code and source manually
5. For Q-AND-A / CONCEPT-SET: set sort weight by rearranging mappings to control answer order

#### Editing Existing Concepts
1. Find concept via search or browse
2. Concept detail → "Edit"
3. Update any fields (not the ID)
4. Save → new resource version created; history preserved, update comment provided

#### Retiring Concepts
1. Concept detail → action menu → "Retire"
2. Confirm; retired concepts remain in history and released versions but are excluded from active expansions

#### Handling Concept Proposals
1. Source → "Concept Proposals" tab
2. Review queue (see `02_capabilities/manage-concept-proposals.md` for full spec)
3. Approve → concept created in HEAD automatically
4. Reject → proposer notified with reason

---

### Phase 3: Validate Content

Before creating a version, the publisher verifies content quality:

1. **Schema validation** (if OpenMRS or FHIR schema configured):
   - Source → Concepts tab → "Validate All" (batch check)
   - Results panel: shows validation errors and warnings per concept
   - Click any error to navigate to that concept's edit form
   - Must resolve all blocking errors before releasing a version

2. **Spot-check via Search**:
   - Search for a representative set of concepts
   - Verify display names, concept classes, and mappings look correct

---

### Phase 4: Create and Release a Version

1. Source → Versions tab → "New Version"
2. Enter version ID (convention: date-based e.g. "v2025-01-15" or semantic e.g. "2.1.0")
3. Enter description (release notes):
   - What was added, changed, retired
   - Any breaking changes
4. "Released" checkbox: check to publish immediately; or uncheck and release manually
5. On release:
   - Version is immutable; HEAD continues as new working copy
   - **Notifications sent** to subscribers and to owners of collections that reference this source
   - Version appears in version selectors and FHIR endpoints

---

### Phase 5: Ongoing Maintenance

After release, the cycle continues:
1. New concepts are authored or accepted via proposals in HEAD
2. Periodic releases (e.g., monthly for CIEL) repeat Phase 3–4
3. Each release triggers downstream notifications to collection owners

---

## Post-Completion State

- Source has at least one released, immutable version
- Source is publicly browsable and FHIR-accessible
- Collection owners have been notified of the new version
- HEAD is ready for the next authoring cycle

---

## Error and Edge Cases

| Situation | Behavior |
|---|---|
| Duplicate concept ID | Blocked at save with inline error; user must choose a different ID |
| Schema validation error on release | Block release; show list of errors with "Fix Now" links |
| Concept proposal approved that creates duplicate | System detects duplicate ID; admin prompted to assign a different ID |
| Source has no Canonical URL | Source can still be used within OCL. A persistent yellow banner is shown on the source page prompting the owner to add one. FHIR endpoints are unavailable and the source cannot be registered in the global canonical URL registry until a canonical URL is set. |
