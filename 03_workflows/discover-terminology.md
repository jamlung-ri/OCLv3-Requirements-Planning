# Workflow: Discover and Evaluate Terminology

## Purpose

A Terminology Consumer (TC) or new Implementer wants to explore OCL — finding relevant code systems, evaluating whether a concept or collection is right for their use case, understanding what CIEL contains, or comparing two terminology options.

This workflow requires no authentication and no write access. It is the "wow factor" entry point that should make OCL immediately useful to new visitors.

---

## Roles

- **Primary actor**: Terminology Consumer (anonymous or authenticated)
- **Supporting actors**: Search index, concept detail views

---

## Entry Conditions

- User may be anonymous or authenticated
- User may arrive via direct URL, search engine, or OCL landing page

---

## Steps

### Option A: Start from Global Search

1. User types a clinical term in the global search bar (e.g., "Diabetes", "HIV", "Malaria")
2. Searchlite shows top concept results immediately (debounced)
3. User presses Enter → full search results page with tabs (Concepts, Repositories, etc.)
4. User browses concept results:
   - Clicks a row → split view opens with concept detail
   - Sees: concept names in multiple locales, concept class, mappings (SNOMED, ICD, LOINC equivalents)
   - Sees: which source this concept belongs to (chip linking to source)
5. User navigates to the source to see context

### Option B: Browse a Known Source

1. User navigates directly to a source URL (e.g., CIEL: `/orgs/CIEL/sources/CIEL/`)
2. TBv3 redirects to the latest released version
3. User browses concept list, searches within the source, or uses hierarchy view
4. Opens concept detail in split view
5. Evaluates the source's quality, coverage, and localization

### Option C: Navigate from Organization

1. User finds an organization (via search or direct URL)
2. Views org's public repositories
3. Selects a source or collection of interest
4. Explores content as in Option B

---

## Concept Evaluation (Detail View)

When evaluating a specific concept, the user wants to know:
- What is this concept? (names in all locales, descriptions)
- What category does it belong to? (concept class, datatype)
- How does it relate to other standards? (mappings: SAME-AS SNOMED, ICD-10, LOINC, etc.)
- Where is it used? (which collections include it)
- Has it changed recently? (history)
- Is it the canonical / recommended concept for this clinical meaning?

The concept detail view must surface all of this without requiring navigation.

---

## Comparing Two Concepts

1. User opens concept A → "Add to Comparison"
2. User finds concept B (via search or browse) → "Add to Comparison"
3. Comparison Drawer opens
4. User sees side-by-side: names, mappings, properties, highlighted differences
5. User can directly navigate to either source from the comparison drawer

---

## Evaluating a Collection

1. User navigates to a collection
2. Views About tab for context and intended use
3. Views Concepts tab to browse content
4. Checks Versions + Expansions tab to see what versions are available
5. May compare two versions to see what changed

---

## Post-Completion State

- User has enough information to decide whether to use this terminology in their system
- No account needed; no data written to OCL

---

## Design Notes

- Anonymous users must have a fully functional read experience — no login walls for public content
- The landing page for anonymous users should show curated "featured" sources and collections
- Search results must load fast (< 500ms for first results) to support exploratory discovery
- Concept detail must show mappings prominently — this is the primary value-add over a raw code browser
