# Capability: Manage References


## Scope

This capability covers all ways a user can define what content belongs in a collection: adding references from outside the collection context (e.g., from search results or concept detail), creating references from inside the collection, previewing reference results without saving, tracing a concept back to the reference that brought it in, transforming existing references, and removing references.

---

## Adding References from Outside the Collection (Add to Collection)

This is the most common path for Terminology Implementers building a dictionary. The user encounters a concept elsewhere in TBv3 and wants to add it to a collection they own.

### Entry Points
- Concept Detail (split view or full page) → "Add to Collection" CTA
- Concept row in Search Results → action button or right-click menu
- Concept row while browsing a repository → action button or right-click menu
- Hierarchy view node → "Add to Collection" action

### Interaction Flow
1. User clicks "Add to Collection"
2. A dialog/drawer opens:
   - **Collection selector**: searchable dropdown of collections the user has edit access to; shows collection name, owner, and current version status (HEAD only — you can only add to HEAD)
   - **Cascade option**: dropdown with options:
     - None — add the concept only
     - Source Mappings — add the concept plus all mappings where it is the "from concept" in its own source
     - [Custom cascade options if configured for the collection]
   - **Preview** (optional, default collapsed): "Preview results" expands to show what will be added (see Preview section below)
3. User confirms → POST to `/[owner]/collections/[collection]/references/` with expression built from the concept URL
4. Success state: inline confirmation with count of concepts/mappings added and a chip linking to the collection
5. Error state: show errors inline (e.g., "3 concepts already exist in this collection")

### Version Consistency Warning
- If the concept being added comes from a **different version of a source** than the collection's current canonical source version:
  - Show a warning in the dialog before confirming: "This concept is from [source] v[X], but this collection currently uses [source] v[Y]. Do you want to add it with a version-pinned reference (to v[X]) or an unversioned reference (which will resolve to v[Y])?"
  - Two options: "Add versioned reference" | "Add unversioned reference (recommended)"
  - If the collection has no canonical version yet, no warning is needed

---

## Creating References from Inside the Collection (New Reference)

Used when the user is in the collection context and wants to define a reference explicitly, including intensional (query-based) references.

### Entry Point
- Collection → References tab → "New Reference" button (owners/admins only)

### Interaction Flow
1. "New Reference" dialog/drawer opens
2. **Reference Type** selector: Extensional | Intensional
3. For **Extensional**:
   - **Expression input**: text field pre-populated with `/:owner/sources/` prefix
   - **Search and select** toggle: instead of typing, the user can search OCL for concepts and the expression is built automatically
   - **Include/Exclude** toggle (default: Include)
   - **Cascade option** dropdown
4. For **Intensional**:
   - **Source selector**: pick the source to query
   - **Filter builder**: add filter rows (Property | Operator | Value)
   - **Preview** button: evaluate the query and show results before saving
5. **Preview** always available before saving (see Preview section below)
6. User saves → reference is added to HEAD version; auto-expansion is queued for update

### Validation
- If the expression does not resolve to any resources: show a warning ("This reference resolves to 0 results. Are you sure you want to save it?")
- If resources already exist in the collection: show a warning ("X concepts from this reference are already in this collection.")
- Errors (e.g., bad URL format, source not found): shown inline, must be resolved before saving

---

## Reference Preview (Without Persisting)

Users can evaluate what a reference would resolve to before committing it. This is non-destructive and does not add anything to the collection.

### Trigger
- "Preview" button within the Add to Collection dialog
- "Preview results" toggle in the New Reference form
- "Preview" action on any existing reference in the References list

### Behavior
- Calls `GET /:owner/collections/:collection/references/?expression=:expr&preview=true`
- Shows a paginated list of concepts/mappings that would be included
- Grouped by: ✅ New (not yet in collection) | ⚠️ Already exists (would be duplicate) | ❌ Error (not resolvable)
- Summary counts shown at the top: "X new concepts, Y already in collection, Z errors"
- Individual rows are not selectable; this is a read-only preview
- For large result sets (>100 resources), preview shows the first 100 with "and X more" indicator; full count is shown 

---

## Reference Presentation in the References Tab

### List View
- Table columns: Expression | Type (Extensional/Intensional/Cascade/Exclude) | Resolved Count | Version Pinned? | Status
- Resolved Count: number of concepts/mappings this reference evaluates to in the current expansion
- Version Pinned: badge shown if the reference is locked to a specific source version; warning badge if that version differs from the canonical source version
- Status: Active | Warning | Error

### Reference Detail (Row Click → Drawer or Split View)
- Full expression (not truncated)
- Reference type and cascade settings
- Full list of resolved resources (paginated)
- "Trace back": any concept in the expansion can link to the reference(s) that brought it in

---

## Reference Transforms (HEAD Only)

Transforms change a reference's expression type without changing the content it resolves to. Only available on the HEAD version. Note that this is distinct from Cascade's Transform option.

### Available Transforms (per SOW)

> **Post-v3:** A transform that lets a user update the pinned version of a repo-versioned reference (e.g., from `v2023` to `v2024`) without changing the reference pattern is deferred to post-v3. See `tbv3-knowledge-base.md`.
1. **Transform to Unversioned** — Changes a resource-versioned reference to an unversioned reference
   - `/:owner/sources/:source/concepts/:id/:resourceVersion/` → `/:owner/sources/:source/concepts/:id/`
   - Use when: cleaning up deprecated resource-versioned references
2. **Transform to Repo-Versioned** — Changes a resource-versioned reference to a repo-versioned reference
   - `/:owner/sources/:source/concepts/:id/:resourceVersion/` → `/:owner/sources/:source/:repoVersion/concepts/:id/`
   - Use when: the user wants to pin to a source version but use the modern pattern
3. **Lock to Repo Version** — Changes an unversioned reference to a repo-versioned reference
   - `/:owner/sources/:source/concepts/:id/` → `/:owner/sources/:source/:repoVersion/concepts/:id/`
   - Use when: the user wants to freeze the reference to the current canonical source version

### UI Interaction
- Transform actions available from the reference row action menu (⋮) in the References tab
- Transform 1 and 2 only available on references that are currently resource-versioned (deprecated pattern)
- Transform 3 only available on unversioned references
- Before applying: show preview of the new expression and what it will resolve to, alongside the old expression (before/after comparison in the preview dialog) and what it resolved to
- After applying: confirmation toast; reference row updates immediately; re-expansion queued

### Bulk Transform
- Checkbox selection in References list
- "Transform selected" dropdown with the same options
- Bulk operations only apply to references where the transform is valid; invalid ones are skipped with a report

---

## Removing References

- Available from reference row action menu (⋮) → "Remove reference"
- Confirmation required: "This will remove [X] concepts from the collection expansion. Continue?"
- Only possible on HEAD version (released versions are immutable)
- Removal queues a re-expansion of HEAD

---

## Linked Source: Resolve to HEAD During Updates

> **Open question:** Whether HEAD-resolution during updates applies only to sources the user owns, or also to sources they do not own (e.g., CIEL), is still unresolved and requires a dedicated design decision. This is one of multiple workflow design areas that deserve dedicated time in the near term.

When a collection is being updated (typically after a new CIEL version is released):
- Unversioned references that previously resolved to a released version now need to be evaluated against HEAD of the source (to pick up new content) and then locked again when a new collection version is created
- This is part of the Update Collection to Latest Source Version workflow (see `03_workflows/`)
- The UI presents this as a guided flow, not a raw reference management task (see workflow doc for details)

---

## Permissions

| Action | Required Permission |
|---|---|
| Preview a reference | Read access to collection and source |
| Add reference (Add to Collection) | Edit access to the collection |
| Create new reference (References tab) | Edit access to the collection |
| Transform reference | Edit access to the collection |
| Remove reference | Edit access to the collection |
