# Object: Concept
(To Do: Add hierarchy attributes i.e. parent_concept_URL)

## Schema

```
concept:
  id: string                    # Unique within its source; URL-safe; never changes after creation
  external_id: string?          # Identifier from an external system
  concept_class: string         # Category e.g. "Diagnosis", "Drug", "Lab Set" — from repo dropdown config
  datatype: string              # Value type e.g. "Coded", "Numeric", "Text", "N/A" — from repo dropdown config
  retired: boolean              # Default false; retired concepts are excluded from active expansions by default
  names: Name[]                 # At least one required
  descriptions: Description[]
  mappings: Mapping[]           # Inline on the concept object; also queryable separately
  extras: Record<string, any>   # Custom key-value attributes
  checksums:
    standard: string            # Hash of all core attributes
    smart: string               # Hash of clinically significant attributes only
  # System-managed:
  owner: string                 # Org or user mnemonic
  source: string                # Source mnemonic
  url: string                   # Canonical (unversioned) URL
  version_url: string           # URL to this specific resource version
  versions: ConceptVersion[]    # Full history
  created_on: datetime
  updated_on: datetime
  created_by: string
  updated_by: string
```

### Name Sub-Object
```
name:
  name: string                  # The text
  locale: string                # BCP-47 e.g. "en", "fr", "sw"
  locale_preferred: boolean     # Is this the preferred name for this locale?
  name_type: string             # e.g. "Fully Specified", "Short", "Index Term" — from repo config
  external_id: string?
```

### Description Sub-Object
```
description:
  description: string
  locale: string
  locale_preferred: boolean
  description_type: string      # From repo config
  external_id: string?
```

---

## Lifecycle / States

```
Active (default)
  └─ Retired  (soft delete; preserved in history, excluded from active expansions)
```

A concept is never hard-deleted if it has been included in any released version. Retire is the correct operation for removing content. Hard-deletes should be enabled for concepts in HEAD only.

---

## Business Rules

### Names (OCL platform rules)
- Every concept must have at least one name with a locale
- A concept may have multiple names in multiple locales

### Names (OpenMRS validation schema — applied when OpenMRS schema is enabled on the source)

Per-concept rules: (To Do: Verify these against oclapi2)
- Must not have more than one preferred name per locale
- All names (except short names) must be unique within the concept
- Must not have more than one short name per locale
- Short name must not be marked as locale preferred
- Only one fully specified name per locale
- At least one fully specified name (across all locales)
- Valid values for class, datatype, name type, and locale

Dictionary-wide rules (enforced across all concepts in the source):
- Fully specified names must be unique across all fully specified names and preferred names (synonyms marked as locale preferred) within a locale ⚠️ *Not yet implemented in OCL validation*
- Multiple concepts cannot share the same preferred name in the same locale
- All concepts should have a unique `external_id` (OpenMRS UUID) with length of 36 chars (per UUID specification)
- All concept names and descriptions should have a unique `external_id` (OpenMRS UUID) with length of 36 chars (per UUID specification) ⚠️ *Not yet implemented in OCL validation*

### Display Name Resolution
When OCL needs to show a single display name for a concept:
1. Find names matching the user's active locale (exact match) (Note: This is a Post-V3 requirements)
2. Among those, prefer `locale_preferred = true`
3. Among preferred, prefer `name_type = "Fully Specified"`
4. If no match in active locale, fall back to the repository's `default_locale` 
5. If still no match, return any available name

### Concept ID
- Assigned by the user or auto-assigned by the source's ID assignment configuration
- Must be unique within the source
- Cannot be changed after creation
- Used in reference URLs
- Future state: May use prefixes, suffixes, etc. or other dynamic rules for ID generation

### Custom Attributes (Extras)
- Arbitrary key-value pairs
- Values may be strings, numbers, booleans, or nested objects
- Displayed in the UI as a table in the concept detail view
- Keys and value types may be constrained by the repo's configuration

### Checksums
- Computed server-side automatically on create/update
- `standard` changes on any attribute change
- `smart` changes only on clinically significant changes (names, mappings, concept class, datatype)
- The API uses checksums to detect whether a concept has meaningfully changed across source versions (see version locking)

---

## API Patterns

| Operation | Endpoint |
|---|---|
| Get concept | `GET /:owner/sources/:source/concepts/:concept/` |
| Search concepts | `GET /:owner/sources/:source/concepts/?q=&concept_class=&datatype=&locale=` |
| Create concept | `POST /:owner/sources/:source/concepts/` |
| Update concept | `PUT /:owner/sources/:source/concepts/:concept/` |
| Retire concept | `DELETE /:owner/sources/:source/concepts/:concept/` (soft) |
| Concept history | `GET /:owner/sources/:source/concepts/:concept/versions/` |
| FHIR lookup | `GET /fhir/CodeSystem/$lookup?system=:canonicalUrl&code=:concept` |
| Cascade | `GET /:owner/sources/:source/concepts/:concept/?cascade=:options` |

---

## Validation Rules by Schema

### OpenMRS Dictionary Schema
See full rule set in [Business Rules → Names (OpenMRS validation schema)](#names-openMRS-validation-schema--applied-when-openMRS-schema-is-enabled-on-the-source) above. Additional rules:
- Concept class must be one of the configured OpenMRS values
- Datatype must be one of the configured OpenMRS values
- Concepts with datatype "Coded" must have at least one mapping with `map_type = "Q-AND-A"` or `map_type = "CONCEPT-SET"`

---

## UI Display Rules

### Concept Chip

When a concept needs to be represented as a compact inline element (e.g., in a mapping association list, in a cascade result, in an "Add to Collection" picker):

| Form | Content | Example |
|---|---|---|
| Short | `[ID] [Display name]` | `168887 Malaria` |
| With source context | `[ID] [Display name] · [repo chip]` | `168887 Malaria · □ CIEL` |

On hover, a tooltip shows:
- Concept class and datatype
- Source repo chip (owner · ID · type · version)
- Quick actions if authenticated (e.g., "Add to Collection")

Chips must not wrap across lines; truncate with ellipsis if space is constrained. See `04_surfaces/design-system.md` for chip styling rules.

### List and Detail Views

- In list views: show concept ID, display name, concept class, source chip
- In split view / detail view: show all names grouped by locale, descriptions, core properties, mappings, custom attributes, history
- Retired concepts: shown with a visual "Retired" indicator; excluded from search results by default (can be included with filter)
- Concepts from another source (referenced into a collection): show provenance chip indicating the originating source
