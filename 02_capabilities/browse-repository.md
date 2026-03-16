# Capability: Browse Repository

## Scope

Navigating repository content once you are inside a repository page — browsing concept and mapping lists, split view detail, hierarchy view, and version context management.

---

## Version Context

At all times when browsing a repository, a **version context** is active. This context is:
- Shown as a version chip in the repository header area
- Persisted in the URL (`/:owner/sources/:source/:version/concepts/`)
- Remembered across tab switches within the same repository

### Version Selector Dropdown
- Displayed in the repo header, always visible
- Shows: Version ID | Status badge (icon only) | Release Date
- HEAD shown last, in muted/italic style
- Versions sorted: most recently released first; HEAD pinned at bottom
- Selecting a version updates the URL and reloads content for that version
- **Default on first load**: redirect to latest released version; if no released versions, use latest non-HEAD version; if no versions at all, use HEAD
- Version ID and date must not wrap to separate lines in the dropdown

---

## Concepts Tab

### List View
- Table columns: Name | ID | Concept Class | Datatype | Repository (if collection context) | Updated
- Default sort: ID ascending (when browsing, no query) or Score descending (when a search query is active)
- Clicking a row opens the Split View
- Multiple selection: checkbox column (shown on hover/keyboard focus); used for bulk actions (Add to Collection, Compare)

### Split View
- Opens to the right of the list; list remains interactive
- Width: ~40–45% of the available area (configurable by dragging the divider)
- Contains the full concept detail (see `04_surfaces/concept-detail.md`)
- Split view closes when: switching to a different tab, pressing Escape, or clicking the X button
- When split view is open, switching the version context updates the detail panel to show the same concept in the new version (if it exists)

### Concept Detail (In Split View)
- See `04_surfaces/concept-detail.md` for full spec

### Hierarchy View
- Available for sources configured with `hierarchy_meaning` (e.g., is-a trees)
- Toggle between List and Hierarchy view via toolbar button
- Hierarchy is a collapsible tree; nodes expand on click to load children (lazy loading)
- Each node: concept ID + display name + concept class badge
- Selecting a node opens split view
- "Add to Collection" action available on each node

---

## Mappings Tab

### List View
- This view is a full redesign from TBv2 (current view is unreadable)
- Table columns: ID | From Concept | Map Type | Target Concept
  - "From Concept": concept ID + name chip, linking to the from-concept
  - "Target Concept": target code + source name (external) or concept chip (internal)
- Default sort: From Concept ID ascending
- Filtering: by Map Type, From Source, To Source, Retired status (default: exclude retired)
- Row click: opens Split View with mapping detail

---

## References Tab (Collections Only)

See `02_capabilities/manage-references.md` for full reference management spec.

### Browse Behavior
- References tab shows the reference definitions (not the resolved expansion content)
- To see the expanded content, use the Concepts or Mappings tabs (which show expansion results) # To Do: Consider consolidated Concepts/Mappings tab that displays them concurrently. Concepts/mappings views may be separately enabled?
- Or use the Versions + Expansions tab to browse a specific expansion

---

## Versions + Expansions Tab (Collections Only)

See `02_capabilities/manage-versions-and-expansions.md` for full spec.

---

## About Tab

- Long-form repository description in markdown format
- Rendered as formatted HTML (not raw markdown)
- Editable by repo owners via Settings
- If no About text is set: show a contextually appropriate empty state (not "no public repositories")

---

## Settings Tab (Owners/Admins Only)

Entry point for repository configuration. See `02_capabilities/configure-repository.md`.

Not shown in the tab bar for users who do not have edit access.

---

## Repository Header

Always visible while browsing:
- Full name / Title
- Canonical URL (if set)
- Repository Type badge
- Description (condensed with "expand" on click)
- Default locale + supported locales
- Access Level badge
- Version selector dropdown
- Action buttons: Follow | Compare | (owner: Edit | Settings)

"View all attributes" link/button:
- Only shown if additional attributes exist beyond what is in the header
- Opens a dialog listing all additional attributes (checksums, validation schema, external ID, contact, etc.)

---

## Summary / About Panel (Sidebar)
- Shows: Created date+by, Last Updated date+by, Total concepts, Total mappings
- Label: "Summary" (not "Statistics")
- Date format: "Created 9/1/2022 by [username]" (not "Created On: 09/01/2022")

---

## Empty States

| Situation | Message |
|---|---|
| No concepts in source | "No concepts yet. [Add Concept] to get started." (owners only see CTA) |
| No concepts matching filters | "No concepts match your current filters. [Clear filters]" | # "Include retired" as a CTA if applicable 
| No released versions | "No released versions. [Create Version] to publish your content." (owners) |
| No references defined | "This collection has no references. [Add reference] to define what content to include." (owners) |
