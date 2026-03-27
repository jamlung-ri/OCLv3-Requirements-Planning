# Object: Expansion

An Expansion is the computed, resolved list of concepts and mappings that result from evaluating all references in a collection version. It is the "answer" to the question: "what concepts are in this collection right now, given these parameters?"

## Schema

```
expansion:
  id: string                    # e.g. "expansion1", "2024-01-01"
  canonical_url: string?        # Stable URL for this specific expansion
  is_default: boolean           # Whether this is the default expansion for its version
  parameters:
    repo_versions: Record<sourceUrl, versionId>  # Pinned source versions
    date: date?                 # Evaluate using versions released as of this date
    filter: Filter[]?           # Additional filter criteria
    active_only: boolean        # Default true; excludes retired concepts
    include_drafts: boolean     # Whether to include HEAD (draft) content
  # Computed / resolved:
  resolved_repo_versions: Record<sourceUrl, versionId>  # Actual versions used
  concept_count: integer
  mapping_count: integer
  excluded_resource_count: integer
  # System-managed:
  collection: string
  version: string               # The repo version this expansion belongs to
  created_on: datetime
  created_by: string
  is_processing: boolean        # True while the expansion is being computed
  error: string?                # Set if the expansion failed
```

---

## Lifecycle / States

```
Processing  →  Complete
               └─ Default (if set as default)
               
(any)       →  Error (if evaluation fails)
```

All collection versions will generally have at least one **auto-expansion** created automatically on version creation. Additional custom expansions can be created with different parameters.

---

## Business Rules

### Auto-Expansion
- Created automatically when a repo version is created
- Parameters default to: `active_only = true`, `repo_versions = {}` (use canonical source versions)
- Users do not need to create a custom expansion unless they want different behavior from the auto-expansion

### Default Expansion
- Each collection version has exactly one default expansion
- The default expansion is what users see when browsing a collection version
- The auto-expansion is the initial default; owners can change this
- When the HEAD collection is being browsed live, the auto-expansion updates as references are added

### Saving a Version: Expansion Snapshot vs. Re-Evaluation
When a collection version is created from HEAD, the default behavior is to **copy the HEAD auto-expansion as-is** (a frozen snapshot of the current resolved state). This ensures the version reflects exactly what HEAD looked like at the moment of version creation.

As an alternative, owners can elect to **re-evaluate all references** at the moment of version creation — useful when they want the new version to reflect the latest resolved content from all referenced sources rather than the snapshot. This option is surfaced as a setting in the Create Version dialog.

### Version Pinning via Parameters vs. References
- **Preferred**: control which source versions are used via expansion parameters (`repo_versions`)
- **Avoid**: embedding version constraints in individual reference expressions
- Exception: if a breaking change was made to a specific concept, that specific reference may need to pin a source version directly

### Canonical Source Version Interaction
- If no `repo_versions` parameter is set, the expansion uses the collection's canonical source version
- If `repo_versions` is explicitly set, it overrides the canonical source version for that expansion
- The `resolved_repo_versions` field always records what was actually used

### Excluded Resources
- References with `include: false` produce excluded resources
- The expansion tracks these separately so users can audit what was intentionally left out

### Stale Expansions
- An expansion may become stale if the underlying source content has changed since it was last evaluated
- The UI should indicate when a released source version has been updated after an expansion was created
- Users can refresh (re-evaluate) any expansion at any time

### Large Expansions
- Expansions over a threshold size (e.g., 10,000+ concepts) are evaluated asynchronously
- The UI should show `is_processing = true` with a non-blocking progress indicator
- The user should be able to navigate away and return; a notification fires on completion

---

## API Patterns

| Operation | Endpoint |
|---|---|
| List expansions for a version | `GET /:owner/collections/:collection/:version/expansions/` |
| Get expansion detail | `GET /:owner/collections/:collection/:version/expansions/:expansion/` |
| Create expansion | `POST /:owner/collections/:collection/:version/expansions/` |
| Delete expansion | `DELETE /:owner/collections/:collection/:version/expansions/:expansion/` |
| Set as default | `PUT /:owner/collections/:collection/:version/expansions/:expansion/` with `is_default: true` |
| Browse expansion concepts | `GET /:owner/collections/:collection/:version/expansions/:expansion/concepts/` |
| FHIR expand | `GET /fhir/ValueSet/:id/$expand` |

---

## UI Display Rules

In the Expansions list (within a version):
- Show expansion ID, default badge, concept/mapping counts, creation date, and who created it
- Show `resolved_repo_versions` as a summary chip (e.g., "CIEL v2024-08-01")
- Show processing indicator if `is_processing = true`
- Show error badge if evaluation failed
- Show "stale" indicator if underlying source content has changed since the expansion was created

Expansion detail:
- Show all parameters used
- Show all resolved repo versions (full list)
- Show list of excluded resources
- Allow browsing concepts/mappings within the expansion

### Expansion Comparison

Expansions can be compared against each other — for example, to review the diff between the current HEAD auto-expansion and a previously released version's expansion, or to compare two custom expansions with different parameters. Expansion comparison is surfaced via the comparison component (see `02_capabilities/compare-resources.md` and `05_decisions/adrs.md` ADR-002).

See also: [ocl_issues#1853](https://github.com/OpenConceptLab/ocl_issues/issues/1853) for linked documentation on expansion comparison entry points, what is shown, and summary statistics.