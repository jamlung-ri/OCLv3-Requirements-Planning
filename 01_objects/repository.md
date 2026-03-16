# Object: Repository

(To Do: Add in template configuration for specific Extra attribute?)

A Repository is the top-level container for terminology content. There are two subtypes: Source and Collection. They share a common schema but have meaningfully different behaviors.

## Schema

```
repository:
  # Identity
  id: string                        # Mnemonic; URL-safe; unique within owner namespace
  name: string                      # Full display name
  owner: string                     # Org or user mnemonic
  owner_type: string                # "Organization" | "User"

  # Classification
  repo_type: string                 # "Source" | "Collection" (may include other types e.g. Concept Dictionary in future)
  source_type: string?              # Source subtype: "Dictionary", "Interface Terminology", "Indicator Registry", etc.
  collection_type: string?          # Collection subtype: "Dictionary", "Value Set", etc.
  custom_validation_schema: string? # e.g. "OpenMRS" — activates schema-based validation

  # Access
  public_access: string             # "View" | "Edit" | "None"
  
  # Identity / FHIR
  canonical_url: string?            # Globally unique URI; used as FHIR identifier
  website: string?
  external_id: string?

  # Localization
  default_locale: string            # BCP-47 e.g. "en" 
  supported_locales: string[]

  # Configuration
  hierarchy_meaning: string?        # For CodeSystems: "grouped-by" | "is-a" | "part-of" | "classified-with"
  autoid_concept_scheme: string?    # "sequential" | "uuid" | "none" — auto-ID assignment strategy
  autoid_concept_start_from: int?
  autoid_mapping_scheme: string?
  dropdown_config: DropdownConfig?  # User-facing controlled vocabulary for concept class / datatype / name types

  # Description
  description: string?
  text: string?                     # Long-form "About" page text; markdown supported
  contact: string?
  meta: string?
  
  # Checksums
  checksums:
    standard: string
    smart: string                   # Derived from concept smart checksums; changes if any concept changes meaningfully

  # System-managed
  url: string                       # Canonical (unversioned) URL
  versions_url: string
  concepts_url: string?             
  mappings_url: string?             
  references_url: string?           # Collections only
  created_on: datetime
  updated_on: datetime
  created_by: string
  updated_by: string
  summary:
    active_concepts: integer
    active_mappings: integer
    versions: integer
```

### DropdownConfig Sub-Object
```
dropdown_config:
  concept_class: string[]           # Allowed values for concept_class field
  datatype: string[]                # Allowed values for datatype field
  name_type: string[]               # Allowed values for Name.name_type
  description_type: string[]        # Allowed values for Description.description_type
  locale: string[]                  # Suggested locales (does not restrict freeform entry)
  map_type: string[]                # Allowed values for Mapping.map_type
```

---

## Repository Subtypes (To Do: Refine this to show ConceptMap as a source/collection, CodeSystem as its own subtype?)

| Repo Type | Subtype | Primary Use | FHIR Equivalent |
|---|---|---|---|
| Source | Dictionary | Core clinical terminology (e.g., CIEL) | CodeSystem |
| Source | Interface Terminology | Point-of-care concepts | CodeSystem |
| Source | Indicator Registry | Health indicators | n/a |
| Source | Externally Defined | Mirrors of external code systems | CodeSystem |
| Collection | Dictionary | Curated concept dictionary for a deployment | ValueSet |
| Collection | Value Set | FHIR value sets | ValueSet |
| Collection | Concept Map | Terminology mappings between sources | ConceptMap |
| Collection | Interface Terminology | Curated interface vocabulary | ValueSet |

---

## Lifecycle / States

For repositories:
```
Active (default)
  └─ Retired  (soft; repo still accessible but flagged as no longer maintained)
```

For versions (see also `01_objects/version.md`):
```
HEAD (mutable draft, always exists)
Released versions (immutable snapshots)
```

---

## Business Rules

### Sources vs. Collections
- Sources **own** concepts and mappings; Collections **reference** them
- Sources can be added to, edited, and versioned independently
- Collections define their content by References; content changes when the underlying Source changes or when References are updated

### Default Version Behavior
- When a user navigates to a repository URL without specifying a version, TBv3 must:
  1. Redirect to the **latest released version** if one exists
  2. Redirect to the **latest non-HEAD version** if there are no released versions
  3. Show **HEAD** only if no other versions exist
- Never default to HEAD if released versions exist

### Repository Header Attributes
Always visible in the header (if non-null):
- Full name / Title
- Canonical URL
- Repository Type (badge)
- Description (condensed with expand on click)
- Default and supported locales
- Access Level (badge)

Accessible via "View all attributes" dialog (not in header):
- Checksums
- Validation Schema
- External ID
- Contact
- ID auto-assignment configuration
- All FHIR-specific attributes

### Canonical URL
- Must be unique across the OCL instance if registering to OCL's Global Registry
- Must be unique within the local URL registry
- Validated against the Canonical URL Registry on create/edit
- Used in all FHIR resource identifiers

### Custom Validation Schema
- Activating a validation schema applies content constraints that are enforced at concept/mapping create/edit
- Changing the schema on an existing repo does not retroactively validate existing content
- Supported schemas: `"OpenMRS"`, `"FHIR_CodeSystem"`, `WHO SMART Guidelines`, others TBD

---

## API Patterns

| Operation | Endpoint |
|---|---|
| Get source | `GET /:owner/sources/:source/` |
| Get collection | `GET /:owner/collections/:collection/` |
| Search repos | `GET /repos/?q=&source_type=&collection_type=&owner=` |
| Create source | `POST /:owner/sources/` |
| Create collection | `POST /:owner/collections/` |
| Update repo | `PUT /:owner/sources/:source/` or `PUT /:owner/collections/:collection/` |
| List repo versions | `GET /:owner/sources/:source/versions/` |

---

## UI Display Rules

**Repository list view (org or user repos page):**
- Show: repo ID, full name, repo type badge, description (truncated), concept/mapping/version count, last updated
- Default sort: ID ascending

**Repository header (detail page):** (To Do: Check this against new Version design)
- See "Repository Header Attributes" above
- Always show: current version context, version selector dropdown
- Version dropdown: show version ID, release status, release date, visibility icon — do NOT wrap
- Version dropdown: show HEAD last, highlighted differently
- Release date, not created date, is the most relevant date in the dropdown

**Repository tabs (Sources):**
- Concepts | Mappings | Versions | About | Settings (owners/admins only)

**Repository tabs (Collections):**
- Concepts | Mappings | References | Versions + Expansions | About | Settings (owners/admins only)
