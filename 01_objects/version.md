# Object: Version (Repository Version)

A Version is a named snapshot of a repository at a specific point in time. Once released, a version is immutable — its content cannot be changed.

## Schema

```
version:
  id: string                    # User-defined name, e.g. "v2024-01-15", "2.0", "current"
  description: string?          # Release notes or changelog
  released: boolean             # True once published
  retired: boolean              # True if deprecated
  external_id: string?

  # Computed
  repo_type: string             # "Source" | "Collection"
  url: string                   # Versioned URL of this version
  concepts_url: string?         # Sources only
  mappings_url: string?         # Sources only
  references_url: string?       # Collections only; references at time of release
  expansions_url: string?       # Collections only
  
  summary:
    active_concepts: integer
    active_mappings: integer
    active_references: integer?   # Collections only
    
  checksums:
    standard: string            # Hash of all content at this version
    smart: string               # Hash of clinically significant content only

  # System-managed
  created_on: datetime
  created_by: string
  updated_on: datetime
  updated_by: string
```

---

## HEAD vs. Released Versions

**HEAD:**
- Always exists; the live, mutable workspace
- Cannot be referenced from other repositories
- Does not have a release date
- Should not be shown first in any dropdown or list; shown last, visually distinct
- Direct URL: `/:owner/sources/:source/HEAD/`

**Released Versions:**
- Immutable: no concept, mapping, or reference changes after release
- Can be referenced from other repos (versioned references, expansion parameters)
- Have a release date (distinct from creation date)
- Have checksums computed over their full content

---

## Lifecycle / States

```
(no releases yet) → HEAD only
  
HEAD  →  [user creates version]  →  Draft Version
                                       ├─ [user releases]  →  Released Version
                                       └─ [remains unreleased]  →  Draft Version
                                       
Released Version  →  [user retires]  →  Retired Version
```

Draft Versions (created but not yet released) are visible to repo owners but not to the public.

---

## Business Rules

### Version Naming
- Must be unique within the repository
- Convention: date-based (`v2024-01-15`) or semantic (`v2.1.0`)
- No enforced format, but guidance should encourage consistent naming

### Creating a Version
- A version is a snapshot of HEAD at the moment of creation
- For collections: creating a version also triggers an **auto-expansion** to be computed
- For sources: creating a version captures all current concepts and mappings; HEAD continues as a new working copy

### Releasing a Version
- Setting `released: true` makes the version publicly accessible and immutable
- Triggers notifications to users/orgs subscribed to this repository (see `02_capabilities/manage-versions.md`)
- For collections: the released version's auto-expansion is finalized

### Checksums and Version Comparison
- Checksums allow quick determination of whether two versions are content-identical
- `standard` checksums differ if *anything* changed
- `smart` checksums differ only if clinically significant content changed
- The Comparison Tool and version locking workflows use smart checksums

### Version Retirement
- A released version can be retired (deprecated) but not deleted
- Retired versions are hidden from default version lists but accessible via direct URL
- Should not be used as expansion parameters in new collections

---

## API Patterns

| Operation | Endpoint |
|---|---|
| List versions | `GET /:owner/sources/:source/versions/` |
| Get specific version | `GET /:owner/sources/:source/:version/` |
| Create version | `POST /:owner/sources/:source/versions/` |
| Release version | `POST /:owner/sources/:source/:version/processing/` |
| Update version metadata | `PUT /:owner/sources/:source/:version/` |
| Retire version | `DELETE /:owner/sources/:source/:version/` (soft) |

---

## UI Display Rules

**Versions tab (within a repository):**
- Table columns: Version ID | Status (HEAD / Draft / Released / Retired) | Release Date | Concepts | Mappings | Description
- Released status shown as a green badge; HEAD as a distinct muted label; Draft as pending indicator
- Sort: most recently released first (released date descending); HEAD pinned at top
- For collections: additional Expansions column showing count of expansions per version

**Version selector dropdown (in repo header):**
- Compact list: Version ID, Status badge (icon only), Release Date
- Do NOT wrap version ID and date onto separate lines
- HEAD appears last with a visual distinction (e.g., italic or muted color)
- Display: release date (not created date) for released versions

**Version context chip (in repo breadcrumb area):**
- Always visible when viewing versioned content
- Format: `[source name] [version ID]`
- Clicking navigates back to the Versions tab
