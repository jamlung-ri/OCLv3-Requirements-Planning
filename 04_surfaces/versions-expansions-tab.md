# Surface: Versions + Expansions Tab (To Do: Review this against current v3 implentation and designs - Joe to provide documentation)

## Scope

Full layout and interaction specification for the **Versions + Expansions** tab on a Collection repository page. This tab is the primary surface for managing version history, expansion state, staleness, and locking.

For Sources, a simplified **Versions** tab exists (left panel only, no right panel). Source-specific differences are noted inline.

---

## Tab Entry Point

- Repository page → tab bar → **"Versions + Expansions"** (Collections)
- Repository page → tab bar → **"Versions"** (Sources)
- Also reachable via direct link from dependency change notifications (opens to the relevant version row pre-selected)

---

## Two-Panel Layout

```
┌─────────────────────────────────────────────────────────────────────┐
│  Versions + Expansions                              [+ New Version]  │
├──────────────────────────┬──────────────────────────────────────────┤
│  VERSIONS                │  EXPANSIONS  for: v2024-11-01            │
│  ──────────────────────  │  ───────────────────────────────────────  │
│  HEAD            Active  │  auto-expansion    Released  [Rebuild ▾] │
│  v2024-11-01  ● Released │  custom-loinc      Released  [Rebuild ▾] │
│  v2024-08-01    Released │                                          │
│  v2023-12-01    Released │  [+ New Expansion]                       │
│  v2023-06-01  ⚠ Stale    │                                          │
└──────────────────────────┴──────────────────────────────────────────┘
```

- Selecting a row in the left panel populates the right panel for that version
- Default selection on tab open: the most recently released version (not HEAD)
- If no released versions exist: HEAD is selected

---

## Left Panel: Version List

### Column Layout

| Column | Width | Notes |
|---|---|---|
| Version ID | flex | Clickable; sets context for right panel |
| Status badge | fixed 90px | Released / Draft / HEAD |
| Stale indicator | fixed 30px | Yellow ⚠ dot if expansions are stale |
| Actions (⋮) | fixed 32px | Context menu |

### Version Row States

**HEAD row** (always present, always first)
- Label: "HEAD"
- Status badge: "Active" (green) or "Editing" (blue) — no Released badge
- Cannot be deleted or released directly; must be versioned via "New Version"
- No expansion panel (HEAD does not have expansions)

**Released version row**
- Status badge: "Released" (blue)
- Stale indicator shown if any expansion for this version is stale
- Row is selectable to view expansions

**Draft version row**
- Status badge: "Draft" (grey)
- Action menu includes "Release" in addition to standard actions
- Draft versions cannot be browsed publicly (read-only for non-owners)

### Version Row Action Menu (⋮)

Available actions vary by version state and user permission:

| Action | Available When | Permission |
|---|---|---|
| View | Always | Read |
| Edit description | Always | Owner/editor |
| Release | Draft only | Owner |
| Create Similar Expansion | Released only | Owner/editor |
| Copy version URL | Always | Read |
| Delete | Draft only, no expansions | Owner |

### Stale Indicator

- Shown on the version row when at least one of its expansions was built before the most recent dependency change for any referenced source
- Tooltip on hover: "One or more expansions for this version are stale. Select this version and rebuild."
- Clicking the stale indicator selects the version and scrolls the right panel to the stale expansion

### "New Version" Button

- Top-right of the tab header
- Visible to owners/editors; hidden for read-only users
- Opens the Create Version dialog (spec in `02_capabilities/manage-versions-and-expansions.md`)

### Empty State (No Versions)

```
No versions yet.
Create a version to snapshot this collection's current state.
[Create First Version]
```

---

## Right Panel: Expansions for Selected Version

### Panel Header

```
EXPANSIONS  for: [version ID chip]    [+ New Expansion]
```

- Version chip is non-interactive (display only)
- "+ New Expansion" is visible to owners/editors; hidden for read-only users

### Expansion Row Layout

| Column | Width | Notes |
|---|---|---|
| Expansion ID | flex | Clickable; navigates to browsable expansion |
| Default badge | fixed 60px | "Default" pill shown if this is the default expansion |
| Status | fixed 90px | Released / Processing / Failed |
| Stale badge | fixed 60px | "Stale" pill (yellow) if expansion is stale |
| Concept count | fixed 80px | e.g., "12,043" |
| Mapping count | fixed 80px | e.g., "8,421" |
| Last built | fixed 120px | Relative timestamp, e.g., "3 days ago" |
| Actions (⋮) | fixed 32px | Context menu |

### Expansion Row States

**Processing**
- Status: "Processing" (animated spinner)
- Counts: shown as "—"
- Actions: "Cancel" only (owner)
- Non-interactive until complete

**Released / Complete**
- Full row data shown
- All actions available per permission

**Stale**
- Yellow "Stale" pill in stale column
- Tooltip: "Built before [Source] released [versionID] on [date]. Rebuild to pick up changes."
- Row is still browsable; stale content is not blocked from use

**Failed**
- Status: "Failed" (red)
- Actions: "Retry" + "View Errors"
- "View Errors" opens an error detail panel below the row

### Expansion Row Action Menu (⋮)

| Action | Available When | Permission |
|---|---|---|
| Browse expansion | Released / Stale | Read |
| Set as default | Released | Owner/editor |
| Rebuild | Any | Owner/editor |
| Create Similar | Released | Owner/editor |
| Copy expansion URL | Released | Read |
| Delete | Not default | Owner |

### "Stale" Indicator Interaction

- Stale badge is a clickable chip
- Clicking opens a tooltip/popover: "Source: CIEL, New version: v2024-11-01, Released: Nov 1, 2024. [Rebuild now] [Dismiss]"
- "Rebuild now" triggers an immediate async rebuild
- "Dismiss" removes the stale flag for this expansion only (without rebuilding); requires owner permission; logged in audit trail

### Empty State (No Expansions for Version)

Shown when a version exists but has no expansions yet (rare; normally auto-expansion is created with the version):

```
No expansions for this version.
[Create Expansion]
```

### Right Panel When HEAD is Selected

HEAD does not have expansions. The right panel shows:

```
HEAD does not have expansions.
Expansions are created when you create a version.
[Create Version]
```

---

## Responsive Behavior

- Below 900px viewport width: panels stack vertically (versions on top, expansions below)
- Stacked layout: selecting a version scrolls the page down to the expansions panel
- Minimum panel width (two-panel): 280px each

---

## Dependency Notification Entry Point

When the user arrives via a dependency change notification link:
- The version referenced by the notification is pre-selected in the left panel
- The stale expansion is scrolled into view in the right panel
- A highlighted banner above the right panel: "Review changes from [Source] [versionID] before rebuilding. [Compare changes →]" (links to the comparison drawer workflow)

---

## Relationship to Other Files

- `02_capabilities/manage-versions-and-expansions.md` — business rules and interaction flows for all actions on this tab
- `03_workflows/update-collection-source-version.md` — the end-to-end workflow that this tab is a key surface in
- `04_surfaces/repository-page.md` — parent page structure and tab bar layout
