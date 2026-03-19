# Workflow: Build a Concept Dictionary

(To Do: Document and work through the issue where OCL will only add concepts from released versions to a collection - Collection managers that also manage the source need a way to enable adding references to HEAD versions of their source. This might be enabling usage of HEAD for that source, or perhaps the use of "linked sources/collections" where the user works in the collection, can create new concepts from that collection (which actually creates them in the source), etc.)

## Purpose

A Terminology Implementer (TI) is starting a new concept dictionary for their OpenMRS (or similar) deployment. They will create a collection, populate it with concepts from CIEL (and possibly other sources), and prepare it for use in their system.

**This is the primary funded use case in the SOW.** (To Do: Remove all mentions of SOW in documentation)

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
3. Fill in: (To Do: Add something about validation schema setup, likely using common presets like OpenMRS)
   - **ID**: e.g., `my-org-dict-2024`
   - **Full Name**: e.g., "My Organization Dictionary 2024"
   - **Collection Type**: Dictionary
   - **Default Locale**: e.g., "en"
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
(To Do: Add this into the "Add Concepts to the Collection" step. User should be able to select a repo version, input comma-separated concept IDs, and create references in OCL from this. )

---

### 3. Add Concepts to the Collection

For each concept (or selection of concepts):
1. User clicks "Add to Collection" (from search result, concept detail, or hierarchy node)
2. **Add to Collection dialog** opens:
   - Collection selector: picks the target collection (defaults to last-used if any)
   - Cascade option:
     - For most CIEL concepts: select **Source Mappings** to also pull in related mappings
     - For specific Q-AND-A sets: select **Source Mappings** (To Do: Change to Source-to-Concept cascade) to capture the question and the concepts in its answer list
     - The dialog should explain what each cascade option does in plain language
   - Preview: user may optionally expand to see what will be added before confirming (To Do: make sure this documentation includes a way of showing cascade e.g. a question --> its answers --> their mappings)
3. User confirms → reference(s) added to HEAD; auto-expansion queued

**System behavior on first add:**
- System resolves the concept URL against CIEL
- The resolved CIEL version becomes the **canonical source version** for this collection
- All subsequent CIEL references will resolve to this same version

**Duplicate handling:**
- If the concept is already in the collection: show an inline warning ("This concept is already in your collection.") but do not block; user can still add if they want a separate reference (To Do: Note that duplicate references with the same extensional/intensional target are not added.)

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
