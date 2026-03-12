# Surface: Concept Detail

## Overview

The Concept Detail view is the most frequently used and highest-value surface in TBv3. It appears in two contexts:
1. **Split View** — opens to the right of a list (search results, repo browser); list remains interactive
2. **Full Page** — navigated to directly via URL or "Open in full page" from split view

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
- Table styling: no internal vertical borders (consistent with Names styling)
- Do not show if no mappings

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
