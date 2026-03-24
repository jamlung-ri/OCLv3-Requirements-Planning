# Surface: Concept Detail (To Do: Review this against current v3 implentation and designs - Joe to provide documentation)

## Overview

The Concept Detail view is the most frequently used and highest-value surface in TBv3. It appears in two contexts:
1. **Split View** — opens to the right of a list (search results, repo browser); list remains interactive
2. **Full Page** — navigated to directly via URL or "Open in full page" from split view (To Do: Remove all Full Page options? We currently do Split View only.)

Both contexts show the same content; split view condenses secondary panels as space allows.

---

## Layout

```
┌─────────────────────────────────────────────────┐
│  HEADER: Concept ID chip | Display Name         │
│  Source chip | Concept Class | Datatype         │
│  [Edit] [Add Mapping] [Add to Collection] [⋮]   │
├─────────────────────────────────────────────────┤
│  NAMES panel                                     │
│  DESCRIPTIONS panel                              │
│  PROPERTIES panel  (concept class, datatype,    │
│                     external ID, extras)         │
│  ASSOCIATIONS panel  (mappings)                  │
│  HISTORY panel  (collapsible)                    │
└─────────────────────────────────────────────────┘
```

---

## Header

- **Concept ID**: displayed in a monospace-style chip; "click to copy" icon; full ID shown on hover if long
- **Display Name**: large text, primary display name per locale resolution algorithm
- **Source chip**: links to the source/collection the concept belongs to; shows owner/source mnemonic
- **Concept Class** and **Datatype**: shown as small badges or inline labels in the header
- **Version context**: if viewing a specific version (not HEAD), the version is shown as a chip
- **Retired badge**: prominent "Retired" badge shown if the concept is retired; entire header area has a muted style

### Action Buttons (contextual)
- **Edit**: owners/editors only; hidden or disabled for non-HEAD views
- **Add Mapping**: owners/editors only; opens inline mapping form
- **Add to Collection**: all authenticated users; opens Add to Collection dialog
- **Add to Comparison**: all users; adds concept to comparison queue
- **Propose Edit**: authenticated non-owners; opens proposal form
- **⋮ More**: retire, view history, open in full page, copy URL

---

## Names Panel

- Section header: "Names" + count badge (e.g., "Names (4)")
- Grouped by locale (e.g., "English (en)", "French (fr)")
- Within each locale: list of name rows
  - Each row: name text | name type (e.g., "Fully Specified") | preferred indicator (star or badge) | external ID (truncated with hover-to-expand)
- External ID: show full ID on hover; "(click to copy)" appended
- Do not show the section if there are no names (empty section should never be rendered)

---

## Descriptions Panel

- Section header: "Descriptions" + count
- Same grouping pattern as Names (by locale)
- Each row: description text | description type | preferred indicator | external ID
- Do not show if no descriptions

---

## Properties Panel

Section header: "Properties"

| Property | Display |
|---|---|
| Concept Class | Inline label |
| Datatype | Inline label |
| External ID | Full value with click-to-copy; truncated if long |
| Retired | Badge (shown only if true) |
| Custom Attributes | Each key-value on its own row; consistent table format with no internal vertical borders |

- Custom attributes (Extras) must always be shown in the same table format — never collapsed into a single "View all" dialog
- If there are no custom attributes, the custom attributes section of Properties is omitted

---

## Associations Panel (Mappings)

Section header: "Associations" + count (e.g., "Associations (12)")

- Grouped by map type (e.g., "SAME-AS", "Q-AND-A", "CONCEPT-SET")
- Within each group: sorted by sort_weight ascending, then alphabetically
- Each row:
  - Map type badge (colored by type)
  - Target concept: clickable Object Chip if internal OCL concept; text code + source name if external
  - Sort weight (shown only if non-null and relevant)
  - Three-dot context menu (owners/editors only; HEAD version only): "Open From Concept" | "Open To Concept" | "Compare Concepts" | "Retire Mapping"
- Table styling: no internal vertical borders (consistent with Names styling)

- Do not show if no mappings
- Map type group headers are sticky during scroll within long lists
- "Add Mapping" button appears at both the top and bottom of the panel (not only at the bottom)
- No max-height constraint on this panel; it expands to its natural height to avoid multiple nested scrollbars on the page

### Retire Mapping from Associations Panel

- Triggered via the three-dot context menu → "Retire Mapping" on a mapping row
- A text field for a retirement reason is shown (optional, but the confirmation step is always required)
- On confirm: the mapping is retired in place; the panel updates without a full page reload; scroll position and focus are preserved on the retired row (now showing a Retired badge)
- Mirrors the behavior of the standalone mapping details page

### Scroll and Focus Preservation

- After any create, edit, or retire action on a mapping in the Associations panel, the panel must return focus to the affected row and preserve scroll position
- The page must not scroll to the top after these operations

### Performance

- The Associations panel must load asynchronously and independently of other concept detail panels
- Skeleton loading must be shown while associations are fetching
- Target: associations for concepts with 400+ mappings must load in under 3 seconds (requires query optimization or API-level indexing)
- See also: ocl_issues#2131

---

## History Panel

- Section header: "History" (collapsible; collapsed by default)
- Shows a chronological list of resource versions:
  - Version ID | Updated date | Updated by | Change summary
- Clicking a history entry shows a diff: what changed in that version

---

## Split View Behavior

- Opens from list row click; does not navigate away
- Width: ~40–45% of available area; draggable divider
- Close button (X) in top right corner of the split view panel
- When split view is open: switching the active version context updates the detail panel (same concept in new version)
- When switching from Concepts tab to Mappings tab: the split view must close (do not show concept detail while on the Mappings list)
- Pressing Escape closes the split view

---

## Breadcrumbs

When in split view, a breadcrumb trail is shown at the top of the detail panel:
`[Org] / [Source] / [Version] / [Concept ID]`
- Each segment is a link
- Breadcrumbs must align correctly vertically with the rest of the panel content

---

## Accessibility

- All panels keyboard-navigable (Tab to focus, Enter to expand/collapse)
- "Click to copy" actions available via keyboard
- Concept ID and external IDs are selectable text (not image or CSS content)
- Retired badge has sufficient color contrast
