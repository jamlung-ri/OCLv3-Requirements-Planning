# Capability: Author Concept

## Scope

Creating new concepts and mappings in a source, editing existing concepts and mappings, and retiring them. Concept Proposals are covered in `02_capabilities/manage-concept-proposals.md`.

Only Source owners and editors can use this capability directly. Terminology Implementers can interact with sources they do not own via Concept Proposals.

---

## Creating a Concept

### Entry Point
- Source → Concepts tab → "New Concept" button (authenticated owners/editors only)
- Concept Proposal approval flow (admin approves → concept created via this same form pre-filled)

### Form Fields

**Required:**
| Field | Input Type | Notes |
|---|---|---|
| Concept Class | Dropdown | Populated from source's `dropdown_config.concept_class`; user cannot type freeform if config is set |
| Datatype | Dropdown | From `dropdown_config.datatype` |
| Names | Repeating group | At least one name required; see Name sub-form below |

**Optional:** 
| Field | Input Type | Notes |
|---|---|---|
| Concept ID | Text | Auto-assigned if source is configured for auto-ID; otherwise user-defined |
| External ID | Text | |
| Descriptions | Repeating group | |
| Retired | Checkbox | Default: unchecked |
| Custom Attributes (Extras) | Key-value editor | Add arbitrary key-value pairs |

### Name Sub-Form (within Concepts form)
- Each name row: Name (text) | Locale (dropdown from `dropdown_config.locale`) | Name Type (dropdown from `dropdown_config.name_type`) | Preferred (checkbox, one per locale)
- "Add another name" button to add rows
- Warning shown if multiple names in the same locale are both marked "Preferred"
- Warning shown if no name in any locale has `name_type = "Fully Specified"` and the source uses OpenMRS validation

### Concept Templates
- If the source has concept templates configured (pre-filled field combinations), a template selector appears at the top of the form
- Selecting a template pre-fills Concept Class, Datatype, and any locale-specific defaults
- User can override any pre-filled value

### Inline Validation
- Real-time validation as the user types/selects:
  - Required fields: red border + message on blur
  - Unique ID check: debounced, shows "ID already exists in this source" inline
  - OpenMRS schema check (if enabled): shows warnings for schema violations inline before save
- Save button is disabled while any blocking validation errors are present
- Non-blocking warnings (e.g., "no Fully Specified name") allow saving with a confirmation step

### After Save
- Concept is added to HEAD version
- User is shown the concept detail view for the new concept
- "Add mapping" CTA is prominently available since many concepts need mappings right after creation

---

## Adding Mappings to a Concept

### Entry Points
- Concept detail view → Associations panel → "Add Mapping" button
- Concept edit form → Mappings tab (for bulk addition during concept creation or editing)

### Mapping Form Fields

| Field | Input Type | Notes |
|---|---|---|
| Map Type | Dropdown | From `dropdown_config.map_type`; required |
| To Concept | Search + select OR text | If "to" concept is in OCL: search and pick; if external: type the code and source |
| To Source | Text or dropdown | If internal OCL concept is selected, auto-populated; otherwise user types |
Sort weight is **not** a field in the per-mapping add/edit form. Instead, ordering of Q-AND-A and CONCEPT-SET answers is managed via **drag-and-drop reordering** within the Associations panel. The `sort_weight` value is set automatically based on the user's drag order. This pattern is already implemented in OCLv2 — see [ocl_issues#1682](https://github.com/OpenConceptLab/ocl_issues/issues/1682) for reference.

### Internal vs. External Target

**OCLv2 behavior (verified):** The Add Mapping form uses a **search-then-fallback** model — no explicit internal/external toggle:

1. User selects a map type (e.g., Q-AND-A) from the Relationship dropdown; the inline row renders with Code and Name fields
2. User types a code into the Code field — the system searches the source for a matching concept
3. **If a match is found:** Name and Source auto-populate from the matched concept
4. **If no match is found:** the system transitions from "select existing" to "add custom" mode — Name and Source fields unlock for free-text entry; user manually types the name and source identifier

**v3 design intent:** Replace the implicit search-then-fallback with an explicit toggle:
- **OCL concept** (default): Searchlite-style search; selecting auto-populates To Source (with optional Name override)
- **External code**: free-text fields for code + source name + optional canonical URL

### After Save
- Mapping appears immediately in the concept's Associations panel
- No page reload required; scroll position and focus move to the new mapping row

---

## Retiring a Mapping

### Entry Points
- Concept detail view → Associations panel → three-dot context menu on a mapping row → "Retire Mapping"
- Mapping detail page → action menu → "Retire Mapping"

### Retire Flow
1. User selects "Retire Mapping" from the context menu
2. A text field appears for an optional retirement reason
3. User confirms; the mapping is retired in the HEAD version
4. The Associations panel updates in place — the mapping row shows a Retired badge
5. Scroll position and focus are preserved on the retired row (page does not scroll to top)

### Un-retiring a Mapping
- Un-retire is available via the same context menu on a retired mapping row ("Un-retire Mapping")
- No reason field required for un-retire; confirmation required

---

## Editing a Concept

### Entry Points
- Concept detail view → "Edit" button (authenticated owners/editors only)
- Cannot edit concepts in a released version (immutable); edit button is hidden or disabled for non-HEAD views

### Edit Form
- Same form as Create, pre-filled with existing values
- All fields editable except: Concept ID (immutable after creation) and Source (concepts cannot be moved)
- Changes are tracked: a new resource version is created on save; full history preserved

### Retiring a Concept
- Available via: Concept detail → action menu → "Retire Concept"
- Confirmation required: "Retiring this concept will exclude it from active expansions. It can be un-retired later. Continue?"
- Retired concepts remain in HEAD and released versions; they are simply flagged as inactive

---

## Configure Repository Dropdown Options

Dropdown configuration (concept class, datatype, name types, map types) is managed in Repository Settings.

### Entry Point
- Repository → Settings tab → "Configure Dropdowns" (owners only)

### Interaction
- For each field: a list of current values with add/edit/delete controls
- Values can be reordered (drag or up/down arrows); order affects display in dropdowns
- Deleting a value does not remove it from existing concepts; it only removes it as a future option in UI forms
- If a source has no dropdown config, freeform text entry is allowed in all concept fields

> **Post-v3:** Allowing users to link dropdown values to an existing OCL source or collection (rather than managing values manually) and specifying OCL's default sources for each field are deferred to post-v3. See `tbv3-knowledge-base.md`.

---

## Permissions

| Action | Required Permission |
|---|---|
| View concept create/edit form | Source owner or editor |
| Create concept | Source owner or editor |
| Edit concept | Source owner or editor; HEAD version only |
| Retire concept | Source owner or editor; HEAD version only |
| Add/edit/retire mapping | Source owner or editor; HEAD version only |
| Configure dropdowns | Source owner |
