# Surface: Design System

Core visual and interaction conventions for TBv3. All surfaces must follow these patterns.

---

## Layout Shell

```
┌────────────────────────────────────────────────────────────┐
│  [Logo]  [Global Search Bar]          [User/Auth] [⚙]      │  ← App Header (fixed)
├──────────┬─────────────────────────────────────────────────┤
│          │                                                   │
│  Left    │  Main Content Area                               │
│  Nav     │                                                   │
│  Panel   │                                                   │
│  (120px) │                                                   │
│          │                                                   │
└──────────┴─────────────────────────────────────────────────┘
                                                   ▲
                               Comparison Bar (fixed bottom, only when active)
To Do: Change Global Search Bar to searchlite?
To Do: Note that Left Nav panel collapses
To Do: Add collapsible right nav options 
```

### Left Navigation Panel
- Contains: logo/brand area (hamburger/icon transforms; does NOT use the label "Quick Links"), quick links to user's repos, recent activity, settings
- Icon for Organizations: not the Parthenon — something more intuitive (e.g., building/office icon)
- Icon for Repositories: not a folder — something specific to OCL's concept (e.g., stacked layers or database-style icon)

### App Header
- Fixed at top; always visible
- Contains: Left nav toggle, global search bar, notification bell (authenticated), user avatar menu

---

## Object Chips

Object chips are compact, clickable representations of OCL objects. They appear inline in text, in headers, and in other contexts where a resource needs to be identified as a single recognizable unit. See [OpenConceptLab/ocl_issues#1776](https://github.com/OpenConceptLab/ocl_issues/issues/1776) for the original design spec.

### Repository Chips

Repository chips are the primary chip component. They represent a specific repo (source or collection) as a "single object" — ID, type, owner, and optionally version.

**Two sizes:**

| Size | Content | Example |
|---|---|---|
| Small | `[repo icon] [ID] [Type]` | `□ LOINC Code System` |
| Regular | `[org icon] [Org] · [repo icon] [ID] [Type]` | `🏛 Regenstrief · □ LOINC Code System` |
| Regular + version | `[org icon] [Org] · [repo icon] [ID] [Type] · [version]` | `🏛 Regenstrief · □ LOINC Code System · latest (v2.77)` |

**Content rules:**
- Repo ID and repo type must **always** be shown together; neither appears alone
- Repo ID is visually emphasized (bold); repo type is secondary weight; org is de-emphasized
- Owner is usually shown; may be omitted when already clear from context (e.g., already shown in a table column)
- Version is usually shown; omit entirely if no version is specified (do not say "no version" — just omit)
- `latest (v2.77)` = reference resolved to latest, currently at v2.77; `latest` = unresolved reference to latest; `v2.77` = pinned to a specific version

**Repo type icons:** Source, Collection, ValueSet, Code System, Concept Map, Dictionary each have distinct icons.

**Tooltip (on hover):**
- Header line: compact chip (org · ID · type, no version)
- Full repository name (bold, large)
- Canonical URL (when available)
- Resolved version

**Truncation:**
- No text wrapping (per Material design guidelines); truncate with ellipsis when space is constrained
- Tooltip always shows the full content
- Truncation priority: Repo ID first, then Owner, then Version, then Repo Type

**Where used:**
- Repository page header
- Dashboard event cards
- Inline in text when referencing a specific repo (e.g., expansion results, reference detail panel)
- **Not used in data tables** — repository information in tables is shown as plain text (e.g., `CIEL · Source v2024-10-24`)

### Concept Chips

Compact inline representation of a concept.

| Form | Content | Example |
|---|---|---|
| Short | `[ID] [Display name]` | `168887 Malaria` |
| With source | `[ID] [Display name] · [repo chip]` | `168887 Malaria · □ CIEL` |

Tooltip shows: concept class, datatype, source repo chip, quick actions if authenticated (e.g., "Add to Collection").

### General Chip Rules
- Chips are clickable (navigate to the resource's detail page)
- Chips have a subtle background color differentiated by resource type
- Chips must not break across lines; use text-overflow: ellipsis if constrained
- Chip text must have sufficient contrast ratio (WCAG AA minimum)
- Owner and version may be hidden when contextually redundant — but Repo ID and Repo Type are never omitted from a repository chip

---

## Split View

- Opens to the right of a list
- Default width: ~40–45% of available area
- Divider: draggable to resize; double-click resets to default
- Close button: "X" in top right of split view
- The list behind the split view remains interactive (scroll, click, search)
- Only one split view open at a time
- Keyboard: Escape closes the split view

---

## Comparison Drawer

- Slides up from bottom of screen
- Default height: ~50% of viewport
- Resize: draggable top edge
- Minimize: a slim "Comparison Bar" at the very bottom (showing queued items)
- The background remains fully interactive when the drawer is open
- Background content scrolls independently of the drawer

---

## Forms

### Layout
- Labels above inputs (not inline/floating, except for filter dropdowns where space is tight)
- Required fields: asterisk (*) in the label
- Help text: below the input, in smaller muted text
- Consistent field widths within a form section (align to a grid)

### Validation Feedback
- Errors: red border on the field + error message below the input
- Warnings: amber border + warning message
- Success (e.g., "ID available"): green checkmark inline
- Validation fires on blur (not on keystroke for most fields); debounced async checks (ID uniqueness, canonical URL) fire ~300ms after last keystroke

### Dropdowns
- Source dropdowns (concept class, map type, etc.) from configured values — not freeform if configured
- Freeform entry if no dropdown config is set
- Searchable dropdown for fields with >10 options
- Dropdown for locale selection includes a "Search for locale" type-ahead

### Repeating Groups (Names, Descriptions)
- "Add another [item]" button below the last row
- Rows have a "remove" icon (trash/X) on the right
- Rows can be reordered if order matters (drag handle on left)

---

## Tables

- Default cell borders: horizontal only (no internal vertical borders)
- Header row: bold, no background color differentiation needed (or subtle)
- Selected row: background highlight (no checkbox required for single selection)
- Multi-select: checkbox column appears on hover/focus for the row
- Sticky header on scroll for long tables
- Empty state: shown within the table area (not as a separate page state)

---

## Date and Time Format

- Dates: `9/1/2022` (not `09/01/2022`)
- Times: `9:15 AM` (not `09:15 AM`)
- Relative dates: use human-friendly relative format for recent events: "2 hours ago", "yesterday", "last week"
- Relative dates switch to absolute format after 30 days

---

## Empty States

Every empty state must:
1. Explain clearly why it is empty
2. Not imply the owner is negligent (e.g., "No public repositories" is a bad empty state for an org page)
3. Provide a relevant CTA for users who have permission to fix it

Do not show empty section panels — if a section has no content, omit the section entirely (e.g., don't show a "Descriptions" panel header with nothing in it).

---

## Loading States

- Skeleton screens for initial content load (preferred over spinners for large content areas)
- Inline spinner for async operations (expansion processing, proposal approval)
- Progress toasts for long-running operations: "Computing expansion... (may take a few minutes)"
- Non-blocking: user should be able to navigate away; a notification fires on completion

---

## Notifications

- Bell icon in header; badge count of unread
- Notification center: accessible from header; shows list of all notifications
- Notification item: icon + message + relative timestamp + CTA link
- Marking as read: clicking a notification; also a "Mark all read" action
- Notification types: source version released, concept proposal status change, comment added

---

## Icons

Avoid misleading default icons:
- Repository ≠ folder icon (use a more specific icon representing a concept store)
- Organization ≠ Parthenon (use a building/office icon)
- Confirm icon choices with a non-technical user test before finalizing

---

## Responsive Behavior

- Minimum supported width: 1024px (desktop-first)
- At reduced resolution:
  - Version chip and version dropdown must not wrap; condense to ellipsis menu if needed
  - Pager component at page bottom must not be clipped
  - Repo header summary must not become unreadable
  - Split view: below 1280px, split view takes full width (overlay mode) rather than side-by-side

---

## Accessibility Minimums

- WCAG 2.1 AA for all interactive elements
- All actions available without a mouse (keyboard navigation)
- Screen reader labels on all icon-only buttons
- Focus states visible and high-contrast
- No color-only information (always pair color with icon or text)
