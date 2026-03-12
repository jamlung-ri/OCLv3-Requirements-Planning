# Capability: Configure Repository

## Scope

Creating repositories and configuring their attributes: type, FHIR settings, validation schema, dropdown vocabulary, locale configuration, About page, and access controls.

---

## Creating a Repository

### Entry Points
- User or Org profile → "New Repository" button (authenticated users only)
- Dashboard → "New Repository" shortcut

### Create Repository Form

**Step 1: Basic Information**

| Field | Required | Notes |
|---|---|---|
| Owner | Yes | Dropdown: user or orgs they are a member of |
| Repository Type | Yes | Radio: Source / Collection |
| ID (Mnemonic) | Yes | URL-safe; unique within owner; no spaces; validated live; cannot be changed after creation |
| Full Name | Yes | Display name; can be changed |
| Description | No | Short description (shown in repo header and search results) |
| Access Level | Yes | Public (View) / Private (None) / Public Edit (rarely used) |
| Default Locale | Yes | Dropdown (BCP-47); default: "en" |
| Supported Locales | No | Multi-select; defines which locales are expected in content |

**Step 2: Type-Specific Configuration**

For Sources:
| Field | Required | Notes |
|---|---|---|
| Source Type | No | Dropdown: Dictionary, Interface Terminology, Indicator Registry, Externally Defined, etc. |
| Custom Validation Schema | No | Dropdown: None, OpenMRS, FHIR CodeSystem |
| Canonical URL | No (recommended) | Validated against Canonical URL Registry on submit |
| Hierarchy Meaning | No | For CodeSystems: is-a, part-of, grouped-by, classified-with |
| Auto-ID Assignment | No | Concept ID auto-assignment: None, Sequential, UUID; if Sequential: starting number |

For Collections:
| Field | Required | Notes |
|---|---|---|
| Collection Type | No | Dropdown: Dictionary, Value Set, Concept Map, etc. |
| Canonical URL | No (recommended) | |

---

## Editing Repository Settings

### Entry Point
- Repository → Settings tab (owners only)

### Settings Tab Sections

**General:**
- All fields from the Create form (except ID, which is immutable)
- Full Name, Description, Access Level, Default Locale, Supported Locales, Canonical URL, Website, External ID, Contact

**Validation Schema:**
- Dropdown: None / OpenMRS / FHIR CodeSystem
- Warning shown when changing schema on a repo with existing content: "Changing the schema does not retroactively validate existing concepts. New and edited concepts will be validated against the new schema."

**Dropdown Configuration:**
- Per-field lists for: Concept Class, Datatype, Name Type, Description Type, Map Type, Locale
- Each list: current values shown with add/edit/delete/reorder controls
- "Add value" inline input at bottom of each list
- Values can be drag-reordered; order controls display order in forms
- Deleting a value: shows count of concepts using it ("15 concepts use this value"); value is removed from the dropdown but preserved on existing concepts

**About Page:**
- Markdown editor for the long-form repository description (shown in the About tab)
- Live preview toggle

**Advanced:**
- Auto-ID assignment configuration (Sources only)
- Retire / Archive repository (soft delete)

---

## Canonical URL Registry

### Purpose
Allows users to test how OCL resolves a canonical URL within a specific namespace (org or user workspace).

### Entry Point
- User profile → "Canonical URL Registry" (or Settings area)
- URL: `/user/[username]/url-registry/` (changes when owner selection changes; do not embed in URL path)

### Interface
- **Registry Owner** dropdown: searchable, shows all users and public organizations across OCL (uses Searchlite-style search)
- **Canonical URL** text input: enter the URL to test
- **Resolve** button (also triggered by Enter key in the URL field)
- Results: show the resolved resource (repo, source, collection) with a link to it
- If not resolvable: show a clear "not found" message with suggestions

### UI Notes
- Registry Owner and Canonical URL inputs should be visually balanced (same width)
- Add a close/exit button in the top right
- Page title: "Canonical URL Registry - OCL" (not "url-registry - OCL")
- Add a "Learn more" link that opens OCL documentation in a new tab (use an external link icon)

---

## Permissions

| Action | Required Permission |
|---|---|
| Create repository | Authenticated user |
| Edit repository settings | Repository owner |
| Configure dropdowns | Repository owner |
| Edit About page | Repository owner |
| Change validation schema | Repository owner |
| Retire/archive repository | Repository owner |
| Test canonical URL | Authenticated user |
