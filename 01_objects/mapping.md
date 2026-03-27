# Object: Mapping

A Mapping is a typed directional relationship between two concepts. It is a first-class resource that can be browsed, searched, versioned, and referenced independently of the concepts it connects.

## Schema

```
mapping:
  id: string                    # Unique within the source
  external_id: string?

  map_type: string              # Required; from repo dropdown config e.g. "SAME-AS", "BROADER-THAN", "Q-AND-A"

  # From (always an OCL concept in this source)
  from_concept_url: string      # Canonical URL of the from-concept
  from_concept_code: string     # Code (ID) of the from-concept (denormalized for display)
  from_concept_name: string?    # Display name (denormalized; may lag source)
  from_source_url: string       # URL of the from-concept's source
  Future: canonical_URL         # source's canonical URL


  # To (may be internal OCL concept or external code)
  to_concept_url: string?       # Set if the target is an OCL concept
  to_concept_code: string?      # Code of the target concept
  to_concept_name: string?      # Display name of target (denormalized)
  to_source_url: string?        # URL of the target source (OCL or external)
  to_source_version_url: string?
  Future: canonical_URL         # source's canonical URL

  sort_weight: float?           # Controls ordering of mappings within display (e.g., answer lists)
  retired: boolean              # Default false
  extras: Record<string, any>

  checksums:
    standard: string
    smart: string

  # System-managed
  owner: string
  source: string
  url: string
  version_url: string
  created_on: datetime
  updated_on: datetime
  created_by: string
  updated_by: string
```

---

## Common Map Types

| Map Type | Meaning | Primary Use |
|---|---|---|
| `SAME-AS` | Exact equivalence | Cross-terminology alignment (CIEL ↔ SNOMED, LOINC, ICD) |
| `BROADER-THAN` | From is broader than To | Hierarchy approximation |
| `NARROWER-THAN` | From is narrower than To | Hierarchy approximation |
| `Q-AND-A` | From is a question; To is an answer | OpenMRS coded question/answer sets |
| `CONCEPT-SET` | From is a set; To is a member | OpenMRS concept sets |
| `ASSOCIATED-FINDING` | Clinical association | Diagnosis grouping |

Allowed map types are configured per repository via `dropdown_config.map_type`.

---

## Mapping Target Types

Mappings fall into three categories based on where the target concept lives:

**Within-source mappings:** Both the `from` and `to` concepts are in the same source. Used for Q-AND-A and CONCEPT-SET relationships (e.g., a coded question concept mapped to its answer concepts within CIEL). The `to_concept_url` points to a concept in the same source as the mapping. See [ocl_issues#1972](https://github.com/OpenConceptLab/ocl_issues/issues/1972) for additional detail.

**Cross-source OCL mappings:** The `to` concept lives in a different OCL source (e.g., CIEL → SNOMED via an OCL-hosted SNOMED mirror). `to_concept_url` points to a concept in a different source within the same OCL instance. The target can be resolved and its details shown inline.

**External code system mappings:** The target is not in OCL at all (e.g., SNOMED CT, ICD-10, LOINC hosted outside OCL). `to_concept_url` is null; `to_source_url` points to an external code system URI. The target cannot be resolved within OCL; only the code and source name are shown. Full documentation of the external code system mapping model is deferred — see `tbv3-knowledge-base.md`.

---

## From/To Field Combination Rules

The minimum required fields for a mapping depend on the target type:

| Target type | Minimum required |
|---|---|
| Within-source or cross-source OCL concept | `to_concept_url` (sufficient on its own; resolves the concept and infers the source) |
| OCL concept specified by ID without full URL | `to_concept_code` + `to_source_url` (source URL provides the namespace) |
| External code system | `to_concept_code` + `to_source_url` |

Optional override: `to_concept_name` may be provided to overwrite OCL's resolved display name for the target concept — useful when the locally preferred display differs from the source's canonical name.

`from_concept_url` is always required and must point to a concept in the mapping's own source.

---

## Business Rules

### Sort Weight
- Used to order Q-AND-A and CONCEPT-SET mappings within an answer/member list
- Lower values appear first
- Not required; unmapped items appear after sorted items in natural order

### Retired Mappings
- Soft-deleted; preserved in version history
- Excluded from active expansions by default (respects `active_only` expansion parameter)
- Still visible in concept detail Associations panel with a Retired badge
- A retirement reason (free-text, optional) can be recorded at the time of retirement; stored in the mapping version record
- Retirement can be initiated from the Associations panel context menu or from the mapping detail page

### Mapping Display in Concept Detail
- In a concept's detail view, show mappings in the **Associations** panel
- Group by map type
- Sort within each map type by `sort_weight` (ascending), then alphabetically by target code
- For internal mappings: show target concept as a clickable Object Chip
- For external mappings: show the target code and source name as static text
- Column headers: "From Concept" | "Map Type" | "Target Concept" (not raw API field names)

---

## API Patterns

| Operation | Endpoint |
|---|---|
| List mappings for a source | `GET /:owner/sources/:source/mappings/` |
| List mappings for a concept | `GET /:owner/sources/:source/concepts/:concept/mappings/` |
| Get mapping detail | `GET /:owner/sources/:source/mappings/:mapping/` |
| Create mapping | `POST /:owner/sources/:source/mappings/` |
| Update mapping | `PUT /:owner/sources/:source/mappings/:mapping/` |
| Retire mapping | `DELETE /:owner/sources/:source/mappings/:mapping/` |
| Search mappings globally | `GET /mappings/?q=&map_type=&from_concept=&to_source=` |

---

## UI Display Rules

**Mappings list view (within a repository):**
- This view requires a redesign from the current TBv2 layout — the existing list is unreadable
- Table columns: ID | From Concept | Map Type | Target Concept (to_concept_code + to_source)
- Row click opens split view with mapping detail
- Resolvable concepts appear as a clickable chip that opens the concept's detail view 
- Filtering: by map_type, from/to source, retired status, from/to source owner, updated by (user), updated since (date)
- Retired mappings hidden by default, toggle filters to show

**Mapping detail (split view):**
- Show all core fields
- Show both from and to concepts as clickable chips (for internal mappings)
- Show sort weight if set
- Show full history (version log)
