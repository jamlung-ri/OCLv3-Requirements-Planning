# Object: Reference

A Reference is a pointer from a Collection to content in a Source (or another Collection). References are the building blocks of a collection's "definition" — they specify what content should be included when the collection is expanded.

## Schema

```
reference:
  id: string                    # System-assigned
  expression: string            # The URL or query that defines this reference (see types below)
  reference_type: string        # "Concept", "Mapping", "Source", "Collection"
  include: boolean              # true = include (default); false = exclude
  cascade: string               # Cascade option: "none" | "sourcemappings" | custom string
  filter: Filter[]?             # Intensional filter criteria
  transformations: string[]?    # Applied transforms e.g. ["unversioned"]
  # Resolved at expansion time:
  resolved_resources: ResourceRef[]  # Concepts/mappings that this reference evaluates to
  # System-managed:
  collection: string
  created_on: datetime
  created_by: string
  updated_on: datetime
```

### Filter Sub-Object (Intensional References)
```
filter:
  property: string              # Concept attribute to filter on (e.g. "concept_class", "datatype")
  op: string                    # Operator: "=", "in", "is-a", "regex", etc.
  value: string
```

---

## Reference Types

### Extensional (Static) — Points to specific known resources

| Pattern | Recommended? | Notes |
|---|---|---|
| `/orgs/:org/sources/:source/concepts/:id/` | ✅ Best practice | Unversioned; resolves to latest released concept |
| `/orgs/:org/sources/:source/:version/concepts/:id/` | ⚠️ Use sparingly | Pins to a specific source version; prefer expansion params instead |
| `/orgs/:org/sources/:source/concepts/:id/:resourceVersion/` | ❌ Deprecated | Never create new references of this type |

### Intensional (Dynamic) — Query-based rules evaluated at expansion time

| Pattern | Recommended? | Notes |
|---|---|---|
| `/orgs/:org/sources/:source/concepts/?q=...` | ✅ Best practice | Query with any filter params |
| `/orgs/:org/sources/:source/:version/concepts/?q=...` | ⚠️ Use sparingly | Prefer expansion params for version control |

### Cascade References
- Applied to any reference via the `cascade` parameter
- Traverses mappings from the referenced concept(s) to gather related concepts/mappings
- Common use: collecting CIEL Q-AND-A sets and CONCEPT-SETs for OpenMRS dictionaries
- Not FHIR-compatible; use only for non-FHIR use cases

---

## Lifecycle / States

```
Active (default)
  └─ Retired / Deleted  (remove from collection definition)
```

References in HEAD are mutable. References in released versions are immutable.

---

## Business Rules

### Version Consistency (To Do: Check this against the current version locking behavior)
- When a collection's first unversioned reference to a source (e.g., CIEL) is resolved, the resolved source version becomes the **canonical source version** for that collection
- All subsequent unversioned references to the same source must resolve to that same canonical version
- Adding a concept from a different version of the same source triggers a conflict warning (see `02_capabilities/manage-references.md`)

### Exclusion References
- A reference with `include: false` acts as an exclusion rule
- Exclusion references are evaluated after inclusion references
- A resource excluded by one reference cannot be re-included by another

### Cascade Rules

Cascade determines how far OCL traverses relationships from a referenced concept when resolving it into an expansion. The following presets are supported:

| Preset | Method | Description |
|---|---|---|
| **None** | — | Adds the referenced concept(s) only; no traversal. Safe for simple concept references. |
| **Source to Mappings** | `sourcemappings` | Adds the concept's direct mappings (within the same source) but does not traverse to the mapped concepts. Safest option for including cross-reference mappings (e.g., SAME-AS SNOMED) without pulling in additional concepts. |
| **Source to Concepts (OpenMRS)** | `sourcetoconcepts` | Traverses from the referenced concept through its Q-AND-A and CONCEPT-SET mappings, recursively (all levels), and returns all reachable concepts and all map types. Used for OpenMRS Q-AND-A sets. |
| **Source to Concepts + OpenMRS Transform** | `sourcetoconcepts` + `transform: openmrs` | Same traversal as above, plus applies the OpenMRS transform (restructures the response into OpenMRS-compatible format). |
| **Custom** | `sourcetoconcepts` | User-specified map types, cascade levels, and return map types. |

**Cascade parameters:**

| Parameter | Values | Notes |
|---|---|---|
| `method` | `sourcemappings` \| `sourcetoconcepts` | Defines the traversal strategy |
| `map_types` | e.g. `Q-AND-A,CONCEPT-SET` | Comma-separated map types to follow during traversal (for `sourcetoconcepts`) |
| `cascade_levels` | integer or `*` | Number of hops to traverse; `*` = all levels |
| `return_map_types` | e.g. `*` or specific types | Which map types to include in the results |
| `transform` | `openmrs` | Optional post-processing transform applied to results |

- External mappings (cross-source) only come over with explicit cascade configuration
- Cascade results are returned as versionless resources tied to the relevant repo version

### Transform Rules
- Transforms change the reference type without changing the content it resolves to
- Transforms are only available on HEAD version references (released versions are immutable)
- After a transform, the new reference expression replaces the old one

### Pre-validation
- Before a new reference is auto-expanded into a HEAD collection, OCL runs pre-validation
- Adding a concept that already exists in the collection (via cascade overlap) is a **Warning**, not an Error
- Results summary groups: successes, warnings (e.g., duplicates or unresolved references), errors (e.g., validation conflicts)

---

## API Patterns

| Operation | Endpoint |
|---|---|
| List references | `GET /:owner/collections/:collection/references/` |
| Add reference(s) | `POST /:owner/collections/:collection/references/` |
| Remove reference | `DELETE /:owner/collections/:collection/references/?expression=:expr` |
| Preview (without saving) | `GET /:owner/collections/:collection/references/?expression=:expr&preview=true` |
| Transform reference | `PUT /:owner/collections/:collection/references/` with transform body |

---

## UI Display Rules

In the References list:
- Show expression (truncated if long, expandable)
- Show type badge: Extensional / Intensional / Cascade / Exclude
- Show resolved resource count (from latest expansion)
- Show version indicator if pinned to a source version
- Show "non-canonical version" warning badge if pinned to a version that differs from the collection's canonical source version
- Each reference row is collapsible/expandable (▶ toggle) to show inline the concepts and mappings it resolves to
- Within the expanded row, concepts and their associated mappings are grouped together; orphaned mappings (not associated with any concept in the reference) appear as a separate group
- Expansion results show: concept code, display name, mapping count, and a repository chip indicating the resolved source (with mouseover tooltip showing the full repository path). This is important because a canonical URL may resolve to different repositories depending on the Canonical URL Registry configuration.
- If a reference resolves to both concepts and mappings, the count display reflects both: e.g., "3 concepts, 10 mappings"
- Mapping sub-rows are expandable inline within the concept group

In Reference detail panel (two-tab layout):

**Tab 1 — Reference details:**
- Reference type (Extensional / Intensional)
- Source version (pinned or canonical)
- Cascade method (e.g., `sourcemappings`, `sourcetoconcepts`)
- Cascade map types (e.g., `Q-AND-A, CONCEPT-SET`)
- Applied filters (if intensional)

**Tab 2 — View expansion:**
- Concept count and mapping count resolved by this reference (within the selected expansion)
- Table columns: Code | Display name | Mappings (count) | Repository (chip with resolved source)
- Orphaned mappings shown as a separate row in the table
- Note: results shown here reflect a specific expansion. If no expansion is selected, the UI defaults to the collection's default expansion (if available).
