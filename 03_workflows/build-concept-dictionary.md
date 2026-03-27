# Workflow: Build a Concept Dictionary

## Purpose

A Terminology Implementer (TI) is starting a new concept dictionary for their OpenMRS (or similar) deployment. They will create a collection, populate it with concepts from CIEL (and possibly other sources), and prepare it for use in their system.

---

## Roles

- **Primary actor**: Terminology Implementer
- **Supporting actors**: CIEL source (as the reference source), system (auto-expansion, validation)

---

## Entry Conditions

- User is authenticated
- User has or will create a Collection in OCL (their own org or user workspace)
- CIEL (or target source) is available in OCL as a public Source with at least one released version

---

## Steps

### 1. Create the Collection

1. Navigate to user or org profile → "New Repository"
2. Select type: **Collection**
3. Fill in:
   - **ID**: e.g., `my-org-dict-2024`
   - **Full Name**: e.g., "My Organization Dictionary 2024"
   - **Collection Type**: Dictionary
   - **Default Locale**: e.g., "en"
   - **Validation Schema**: Select a schema to enforce content rules automatically. Common presets:
     - **OpenMRS** — requires a Fully Specified name per locale, validates concept class and datatype against allowed values, enforces Q-AND-A mapping rules for coded concepts. Recommended for OpenMRS implementations using CIEL.
     - **None** — no schema-level enforcement; validation errors will not block reference addition or version creation.
   - Schema can be changed later in Collection Settings → Validation, but changing it does not retroactively validate existing references.
4. Save → user is taken to the new (empty) collection

**System behavior:** Collection is created with HEAD version only. No references, no expansion yet.

---

### 2. Find Concepts in CIEL

The user browses or searches CIEL to find concepts they need.

**Path A — Search globally:**
1. Global search → type a clinical term (e.g., "Malaria")
2. Filter results by Source = "CIEL"
3. Scan results; open concept detail in split view to evaluate
4. Add desired concepts one by one or in bulk

**Path B — Browse CIEL directly:**
1. Navigate to CIEL source page (`/orgs/CIEL/sources/CIEL/`)
2. TBv3 redirects to latest released version automatically
3. Browse concept list or use in-repo search
4. Select and add concepts

**Path C — Add from a saved list:**

Used when the TI already has a known list of concept IDs (e.g., from a spreadsheet, a prior system, or a shared concept set) and wants to add them all to the collection at once.

1. Navigate to collection → References tab → "New Reference"
2. Select **Bulk Add by Concept IDs**
3. In the dialog:
   - **Source**: select the target source (e.g., CIEL)
   - **Source Version**: select a specific released version, or leave blank to use the collection's canonical source version
   - **Concept IDs**: paste a comma-separated list of concept IDs (e.g., `1234, 5678, 91011`)
4. Click **Preview** — the system resolves each ID and shows:
   - ✅ Found and will be added
   - ⚠️ Already in this collection (will be skipped — duplicate references with the same extensional target are not added)
   - ❌ Not found in the specified source/version
5. Review the preview; adjust the list if needed
6. Click **Add to Collection** — references are created for all resolved, non-duplicate concepts; auto-expansion is queued

**Cascade behavior:** The same cascade options available in Path A/B (Source-to-Concept cascade, none) apply here. If cascade is selected, associated mappings are included for each concept in the list.

---

### 3. Add Concepts to the Collection

For each concept (or selection of concepts):
1. User clicks "Add to Collection" (from search result, concept detail, or hierarchy node)
2. **Add to Collection dialog** opens:
   - Collection selector: picks the target collection (defaults to last-used if any)
   (To Do: Create list of all cascade options that OCL can offer, e.g. Source-to-Mappings, Source-to-Concept, Custom, etc.)
   - Cascade option:
     - **None** — adds the concept only
     - **Source-to-Concept cascade** — adds the concept plus all concepts and mappings reachable via that concept's mappings within the same source. For Q-AND-A sets, this captures the question concept, all answer concepts, and the mappings between them. For most CIEL concepts, this also pulls in cross-reference mappings (e.g., SAME-AS SNOMED).
     - Each option includes a plain-language description in the dialog so users understand what will be added.
   - Preview: user may optionally expand **"Preview results"** to see the full cascade chain before confirming:
     - For a Q-AND-A concept: shows the question → each answer concept → each answer's mappings
     - Summary counts: "X concepts, Y mappings will be added"
     - Already-in-collection items shown separately ("X already in your collection — will be skipped")
3. User confirms → reference(s) added to HEAD; auto-expansion queued

**System behavior on first add:**
- System resolves the concept URL against CIEL
- The resolved CIEL version becomes the **canonical source version** for this collection
- All subsequent CIEL references will resolve to this same version

**Duplicate handling:**
- If the concept is already in the collection: show an inline warning ("This concept is already in your collection.")
- Duplicate references with the same extensional or intensional target are not added — the system silently skips them upon resolving the references

**Version consistency warning:**
- If the user is browsing CIEL HEAD or a different version from the collection's canonical version: warn before adding (see `02_capabilities/manage-references.md` for details)

---

### 4. Review the Collection

1. Navigate to collection → **Concepts tab**
2. Browse or search within the collection to verify concepts are present
3. Check the **References tab** to see the reference expressions that were created
4. Check the **Versions + Expansions tab → HEAD → auto-expansion** to see the current resolved count

---

### 5. Create a Version and Release

When the dictionary is ready for use:
1. Collection → Versions + Expansions tab → "New Version"
2. Enter a version ID (e.g., "v1.0" or "2024-01-15")
3. Check "Released" to publish immediately (or leave unchecked and release manually later)
4. Save → version created; auto-expansion computed asynchronously
5. When expansion is complete: version is ready for downstream use (e.g., OpenMRS FHIR API)

---

## Post-Completion State

- Collection has at least one released version
- The released version has a completed auto-expansion
- Collection is publicly accessible (if Access Level = Public)
- Other users can now browse, search, and download the collection
- Canonical source version (e.g., "CIEL v2024-01-15") is locked for this collection

---

## Related Workflows

- [Add to Existing Dictionary](add-to-existing-dictionary.md) — for extending the collection later
- [Update Collection to New Source Version](update-collection-source-version.md) — for keeping current with CIEL

---

## Error and Edge Cases

| Situation | Behavior |
|---|---|
| Concept not found in CIEL | "No results" in search; suggest refining terms or browsing hierarchy |
| CIEL has no released version | Show HEAD; warn user that the collection will reference a draft source |
| Concept already in collection | Warning in Add to Collection dialog; proceed anyway if user confirms and if new reference is not duplicative|
