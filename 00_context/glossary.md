# Glossary

Use these terms precisely and consistently throughout all requirements, design, and code.

---

## Primary Objects

**Concept** — A single clinical term or coded entry within a Source. Has names, descriptions, concept class, datatype, mappings, and optional custom attributes. The primary unit of content in OCL.

**Mapping** — A typed relationship between two concepts. Can be internal (within the same OCL instance) or external (pointing to a code in a system outside OCL). Map types include SAME-AS, BROADER-THAN, NARROWER-THAN, Q-AND-A, CONCEPT-SET, and others.

**Repository (Repo)** — The top-level container for terminology content. Either a Source or a Collection.

**Source** — A repository that directly owns and manages concepts and mappings. Content is authored in OCL or imported from an external code system (e.g., CIEL, LOINC).

**Collection** — A repository that contains references pointing to concepts and mappings in Sources. Collections do not own their content — they curate it via references.

**Reference** — A pointer from a Collection to content in a Source (or another Collection). See full definition under Reference Types below.

**Version (Repo Version)** — A named, immutable snapshot of a repository at a point in time. The **HEAD** version is the live mutable workspace and is not a real version.

**Expansion** — The computed result of evaluating all references in a collection version. A resolved list of concepts and mappings ready for browsing, download, or FHIR serving.

**Concept Proposal** — A request submitted by a user to add or modify a concept in a source they do not own. Tracked as a first-class resource with status, comments, and audit trail.

**Organization (Org)** — A named namespace that owns repositories and has members with defined roles.

**Owner** — An Organization or User that owns a Repository.

**Canonical URL** — A globally unique URI identifying a resource across OCL instances and FHIR servers.

---

## Reference Types

**Extensional Reference** (Static) — Points to a specific resource. Best practice: use unversioned form.
- Unversioned: `/orgs/:org/sources/:source/concepts/:concept/` ← **preferred**
- Repo-versioned: `/orgs/:org/sources/:source/:repoVersion/concepts/:concept/` ← use only when necessary
- Resource-versioned: `/orgs/:org/sources/:source/concepts/:concept/:resourceVersion/` ← **deprecated, do not create**

**Intensional Reference** (Dynamic) — A query or rule evaluated at expansion time.
- Query: `/orgs/:org/sources/:source/concepts/?q=...` ← **preferred for intensional**
- Versioned query: `/orgs/:org/sources/:source/:repoVersion/concepts/?q=...` ← use only when necessary

**Canonical Source Version** — The source version a collection has locked to for consistent reference resolution. Set on first resolution of an unversioned reference.

---

## Reference Transforms

Operations that change the type of an existing reference without changing its target content:

- **Transform to Unversioned** — resource-versioned → unversioned (cleans up deprecated patterns)
- **Transform to Repo-Versioned** — resource-versioned → repo-versioned
- **Lock to Repo Version** — unversioned → repo-versioned (pins to a specific source version)

---

## Expansion Parameters

Settings controlling how a collection expansion is evaluated:

- **Repo Version(s)** — Pin which version of each referenced source to use
- **Date** — Evaluate using released versions as of this date
- **Filter** — Include/exclude criteria for intensional references
- **Active Only** — Exclude retired concepts

---

## FHIR Equivalents

| OCL Term | FHIR Resource |
|---|---|
| Source | CodeSystem |
| Collection | ValueSet or ConceptMap |
| Expansion | ValueSet expansion |
| `$expand` | Evaluate a ValueSet |
| `$lookup` | Get a concept from a CodeSystem |
| `$validate-code` | Check if a code is valid |
| Cascade | (OCL-only; not standard FHIR) |

---

## Version and State Terms

**HEAD** — The live, mutable workspace version of a repository. Not a real version. Should not be referenced from other repos.

**Released Version** — A named, immutable snapshot that has been made publicly accessible.

**Auto-expansion** — The expansion automatically generated for every collection version. Users do not need to manually trigger this unless they want custom expansion parameters.

**Version Locking** — The behavior where a collection locks to a specific resolved source version after its first reference resolution, preventing future references from silently resolving to a different source version.

---

## UI Terms

**Split View** — A panel that opens to the right of a list, showing resource detail without navigating away. The list remains interactive.

**Comparison Drawer** — A panel that slides up from the bottom of the screen to show a side-by-side comparison. The background view remains visible and interactive.

**Object Chip** — A compact inline representation of an OCL object (concept, repo, org, etc.) with a short form displayed by default and a long form on hover.

**Searchlite** — The inline search/autocomplete that appears as the user types in the global search bar, before the full search results page loads.

**Cascade** — An operation that traverses mappings from a starting concept to gather related concepts and mappings (e.g., collecting question-answer sets in OpenMRS). Not natively supported by FHIR.

**Checksum** — A hash computed from a resource's content. Used for equality comparison without full attribute diff. Two types: *standard* (all attributes) and *smart* (clinically significant attributes only).
