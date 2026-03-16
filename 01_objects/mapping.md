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

(To Do: Verify rules for Mapping From/To concepts e.g. field combinations.)

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

## Internal vs. External Mappings (To Do: Add in the distinction for within-source Internal mapping e.g. Q&A mappings (unsure how we should classify this - within a source, within ownership, etc.))

**Internal mapping:** `to_concept_url` points to a concept in the same OCL instance. The target concept can be resolved and its details shown inline.

**External mapping:** `to_concept_url` is null; `to_source_url` points to a code system outside OCL (e.g., SNOMED CT, ICD-10). The target cannot be resolved within OCL; only the code and source name are shown.

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
- Filtering: by map_type, from/to source, retired status (To Do: add other filters)
- Retired mappings hidden by default, toggle filters to show

**Mapping detail (split view):**
- Show all core fields
- Show both from and to concepts as clickable chips (for internal mappings)
- Show sort weight if set
- Show full history (version log)
