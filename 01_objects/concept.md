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

## Business Rules (To Do: Check these against OpenMRS validation schema rules to verify which are OCL-specific vs. OpenMRS-specific)

### Names
- Every concept must have at least one name
- Each locale must have at most one name with `locale_preferred = true`
- Each locale should have at most one name with `name_type = "Fully Specified"` (required by OpenMRS validation schema if used)
- A concept may have multiple names in multiple locales

### Display Name Resolution (To Do: Verify this)
When OCL needs to show a single display name for a concept:
1. Find names matching the user's active locale (exact match)
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

### OpenMRS Dictionary Schema (to do: Verify and fill this in.)
- Must have exactly one name with `name_type = "Fully Specified"` per locale present
- Concept class must be one of the configured OpenMRS values
- Datatype must be one of the configured OpenMRS values
- Concepts with datatype "Coded" must have at least one mapping with `map_type = "Q-AND-A"` or `map_type = "CONCEPT-SET"`

---

## UI Display Rules(To do: Add concept chip.)

- In list views: show concept ID, display name, concept class, source chip
- In split view / detail view: show all names grouped by locale, descriptions, core properties, mappings, custom attributes, history
- Retired concepts: shown with a visual "Retired" indicator; excluded from search results by default (can be included with filter)
- Concepts from another source (referenced into a collection): show provenance chip indicating the originating source
