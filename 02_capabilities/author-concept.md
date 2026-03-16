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
| Sort Weight | Number | Optional; used for ordering Q-AND-A / CONCEPT-SET answers | 
(To Do: Remove Sort Weight from here, provide a better method for assigning sort weight in UI)

### Internal vs. External Target
- Toggle: "OCL concept" (default) | "External code"
- OCL concept: shows a Searchlite-style search for concepts; selecting auto-populates To Source
- External code: shows free-text fields for code + source name + optional canonical URL
(To Do: Document OCLv2's current Add Mapping behavior here, namely for populating "Source name" via search, canonical URL, etc.)

### After Save
- Mapping appears immediately in the concept's Associations panel
- No page reload required

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

### Interaction (To Do: Factor in the probable case that users will link these to an OCL source/collection instead of custom-managing each value)
- For each field: a list of current values with add/edit/delete controls
- Values can be reordered (drag or up/down arrows); order affects display in dropdowns
- Deleting a value does not remove it from existing concepts; it only removes it as a future option in UI forms
- If a source has no dropdown config, freeform text entry is allowed in all concept fields
- (To Do: Add OCL's default sources for each field)

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
