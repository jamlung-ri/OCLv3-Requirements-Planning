# Capability: Validate Content

## Scope

Real-time and on-demand validation of references, concepts, and mappings against schema rules defined at the repository or collection level. This capability surfaces schema violations inline during authoring and during the reference-addition workflow, preventing invalid content from being committed to expansions or versions.

Applies to: collection references, source concepts, source mappings.

---

## Validation Contexts

Validation can be triggered in three contexts:

| Context | Trigger | Who sees it |
|---|---|---|
| **Reference form** (Add References dialog) | On-blur per field; on "Preview" before finalizing | User adding references |
| **References tab** (existing references) | Continuous background scan; also triggered on manual "Validate" action | Collection owner/editor |
| **Concept / Mapping editor** | On-blur per field; on Save | Concept/mapping author |

---

## Severity Model

Three severity levels. Each has a distinct visual treatment and different blocking behavior.

| Severity | Color | Icon | Blocks expansion? | Blocks version creation? |
|---|---|---|---|---|
| **Error** | Red | ❌ | Yes | Yes |
| **Warning** | Yellow | ⚠️ | No | No (but shown in summary) |
| **Info** | Blue | ℹ️ | No | No |

A collection with any unresolved **Error**-level violations cannot be expanded or have a new version created until the errors are resolved or explicitly suppressed by an owner.

---

## Validation Rules (To Do: Consider configurability of these e.g. "Make your own validation schema" and changing severity of each.)

### Built-in Rule Categories (To Do: Validate these further)

**Required Properties**
- Concept is missing a required property defined by the schema (e.g., LOINC Component, LOINC Property, OpenMRS Datatype)
- Severity: Error (if marked required by schema); Warning (if recommended)

**Allowed Concept Classes**
- Concept's class is not in the set of allowed classes for this collection/source schema
- Example: A LOINC-typed source that only allows Test and Procedure classes
- Severity: Error

**Allowed Datatypes**
- Concept's datatype is not in the allowed set
- Severity: Error

**Mapping Constraints**
- Required mapping type is absent (e.g., schema requires a SAME-AS to an external terminology)
- Mapping target is in a deprecated or unavailable source
- Severity: Warning (for missing recommended mappings); Error (for structural violations)

**Cardinality Constraints**
- More than the allowed number of references to a given concept (e.g., max 1 per concept per collection)
- Severity: Error

**Reference Structure**
- Reference expression is malformed or points to a resource that cannot be resolved
- Severity: Error

### Schema Sources

Rules are defined at two levels:
- **Repository-level schema**: set in repository settings; applies to all collections/sources in the org that reference this repo
- **Collection-level schema**: overrides or extends the repository-level schema for a specific collection (To Do: Note that this is a brand new idea - interesting! Actually this whole page is interestingly new.)

Predefined schema templates:
- OpenMRS (concept classes, datatypes, required mappings)
- Custom (user-defined rules)

---

## Warning Display

### In the Reference Form (Add References Dialog)

- Inline warning appears below the affected field immediately on blur
- A summary count appears at the top of the Preview panel before finalizing: "3 errors, 1 warning — review before adding"
- The "Add References" confirm button is disabled if any Error-level violations exist
- Each warning shows:
  - Plain-language description of the issue
  - Which schema rule was violated
  - Suggested fix (where deterministic)
  - Link to schema documentation (if available)

### In the References Tab (Existing References)

- References with violations display a severity badge (❌ / ⚠️ / ℹ️) in the status column
- Clicking the badge opens an inline detail panel showing all violations for that reference
- A collection-level summary banner appears at the top of the tab when errors exist: "This collection has [N] validation errors. Expand or create a version to see full impact. [View all errors]"
- "View all errors" opens a filterable validation report panel (see below)

### In the Concept / Mapping Editor

- Field-level: red outline + error message below the field for Error; yellow outline for Warning
- Form-level: summary at top of Save button area: "Fix [N] error(s) before saving"
- Warnings do not block save but are shown persistently until resolved

---

## Validation Report Panel

Accessible from: References tab → "View all errors" banner link; also via a standalone "Validate" action in the collection action menu (⋮). (To Do: Change this so that it works in sources as well as collections. May be a misunderstanding here that misses how validation applies to sources and collections.)

**Panel contents:**
- Filter bar: severity (All / Error / Warning / Info), status (Unresolved / Ignored), reference type
- Table columns: Reference | Rule Violated | Severity | Suggested Fix | Status | Actions
- Actions per row: "Fix" (opens that reference in edit mode), "Ignore" (with confirmation and required reason), "View Rule" (opens schema rule detail)
- Bulk actions: "Ignore all warnings", "Export report (CSV)"

**Export format:** CSV with columns: reference_expression, severity, rule_id, rule_description, suggested_fix, status, ignored_by, ignored_at

---

## User Actions on Violations

| Action | Who can do it | Notes |
|---|---|---|
| Fix inline | Editor/owner | Opens reference or concept editor for that item |
| Ignore (Error) | Owner only | Requires a written reason; logged in audit trail; does not remove the error from counts for version/expansion blocking — owner must also confirm bypass at expansion time |
| Ignore (Warning / Info) | Editor/owner | Silences the warning for that item; can be un-ignored |
| Export report | Editor/owner | CSV of all current violations |
| Re-run validation | Editor/owner | Forces a fresh validation pass (clears cached results) |

---

## Schema Management

### Configuring Rules
- Location: Repository Settings → Validation tab; Collection Settings → Validation tab
- UI: rule list with toggles (enabled/disabled), severity selector per rule, value inputs where applicable
- "Apply template" button to load a predefined schema (OpenMRS, FHIR, Custom)
- "Preview impact" button: runs validation against current content and shows a summary count before saving changes

### Rule Versioning
- Schema rules are saved with a version stamp
- When a schema changes, existing violations are re-evaluated on next validation run
- Schema version is recorded in the collection's version metadata for audit purposes

---

## Performance Requirements

- Single-reference validation: < 100ms (client-side where possible, server-side for accuracy)
- Background scan of full collection: async; does not block UI; progress indicator in References tab header
- Batch validation (e.g., 500+ references): debounced; results streamed progressively
- Validation results cached per reference; invalidated only when the reference or schema changes

---

## Relationship to Other Capabilities

- **Manage References** (`manage-references.md`): the Add References dialog calls the validation engine before allowing finalization
- **Manage Versions and Expansions** (`manage-versions-and-expansions.md`): version creation and expansion are blocked when Error-level violations exist (unless owner bypasses)
- **Configure Repository** (`configure-repository.md`): schema rules are configured in repository/collection settings

---

## Open Questions

- Should Error-level violations block **draft** version creation, or only **release**? (Current assumption: only release)
- Can a collection owner fully bypass all Error violations to force an expansion? (Current assumption: yes, with an explicit acknowledgment dialog and audit log entry)
