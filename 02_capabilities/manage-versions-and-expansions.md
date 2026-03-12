# Capability: Manage Versions and Expansions

## Scope

Creating, releasing, and managing repository versions. Creating, configuring, and managing collection expansions. Dependency notifications. Locking controls (Rebuild, Create Similar Expansion). Applies to both Sources and Collections, with collection-specific behaviors called out.

---

## Versions + Expansions Tab

This tab is only fully functional for Collections (per SOW). For Sources, the Versions tab shows versions only.

### Tab Location
- Repository → "Versions + Expansions" tab (Collections)
- Repository → "Versions" tab (Sources)

### Versions + Expansions Tab Layout (Collections)

Two-panel layout within the tab:
- **Left panel**: List of all versions (including HEAD)
- **Right panel**: Expansions for the selected version

Selecting a version in the left panel updates the right panel to show that version's expansions.

See `04_surfaces/versions-expansions-tab.md` for the full layout specification.

---

## Creating a Version

### Entry Point
- Versions tab → "New Version" button (owners/admins only)
- Accessible from both Sources and Collections

### Interaction Flow
1. Dialog opens with:
   - **Version ID**: text input (e.g., "v2024-01-15"); validated for uniqueness within the repo
   - **Description**: textarea for release notes
   - **Released**: checkbox (default: unchecked — creates as Draft first)
2. On submit:
   - For Collections: also creates the **auto-expansion** (async); progress indicator shown while it computes
   - For Sources: creates a snapshot of HEAD; HEAD continues as a new working copy
3. Success: new version appears in the list; if collection, expansion shows as "Processing"

### Releasing a Draft Version
- Draft versions have a "Release" action button in the version list
- One-click; confirmation dialog: "Release this version? It will become publicly accessible and immutable."
- On release:
  - Version status badge updates to Released
  - For collections: auto-expansion is finalized
  - Notification sent to subscribers (see Dependency Notifications below)

---

## Expansions (Collections Only)

### Auto-Expansion
- Created automatically whenever a collection version is created
- Labeled "auto-expansion" in the UI
- Parameters: `active_only = true`, no pinned repo versions (uses canonical source versions)
- Users do not need to create an expansion manually unless they want different parameters
- Auto-expansion is the default expansion for the version

### Custom Expansions
- Owners can create additional expansions with custom parameters
- Use cases: "What does this collection look like pinned to CIEL 2023 instead of 2024?", "Include retired concepts for audit"

### Creating a Custom Expansion

**Entry Point:** Versions + Expansions tab → Select a version → "New Expansion" button

**Interaction Flow:**
1. Dialog opens with:
   - **Expansion ID**: text input
   - **Parameters**:
     - **Repo Versions**: per-source version override table (source name | version selector)
     - **Date**: date picker ("Use versions released as of this date")
     - **Active Only**: toggle (default: on)
     - **Include Drafts**: toggle (default: off)
   - **Set as Default**: checkbox
2. On submit: expansion created; shown as "Processing" while being evaluated
3. On complete: concept/mapping counts appear; browsable

### Setting Default Expansion
- Any expansion for a version can be set as default
- "Set as default" action available from expansion row action menu (⋮)
- Only one default per version at a time; setting a new default removes the flag from the current one

### Rebuild Expansion
- Available on any expansion via action menu (⋮) → "Rebuild"
- Re-evaluates the expansion with the same parameters (picks up any underlying source content changes)
- Shows as "Processing" while rebuilding; non-blocking (user can navigate away)
- On complete: updated counts shown; staleness indicator removed

### Create Similar Expansion
- Available on any expansion via action menu (⋮) → "Create Similar"
- Opens the "New Expansion" dialog pre-filled with the current expansion's parameters
- User can modify any parameter before saving
- Use case: "Try this expansion but pin CIEL to v2025 instead of v2024"

---

## Dependency Notifications

_Maps to SOW Tracker #62 (dependency change notification) and #63 (review and accept upstream updates)._

### Detection: How TBv3 Knows a Dependency Changed

A collection has a dependency on a source when it contains one or more references whose expressions resolve to concepts in that source. TBv3 tracks this relationship as a **dependency graph** maintained server-side.

When a source releases a new version, the platform:
1. Identifies all collections whose HEAD references resolve to that source
2. Compares the new source version's content against the version currently resolved by those collections
3. Generates a dependency change event for each affected collection

The comparison produces a change summary:
- **Added concepts**: concepts in the new version not present in the previous
- **Retired/removed concepts**: concepts that have become inactive or were deleted
- **Modified concepts**: concepts whose names, properties, or mappings changed
- **Affected references**: count of references in the collection that point to changed concepts

This event is what triggers the notification. No notification is generated if the new source version contains no changes that affect the collection's resolved references.

### What Triggers a Notification
- A source repository that a collection references has released a new version
- The released version contains at least one change affecting concepts referenced by the collection
- Notifications are sent to the **owner** of the collection (and optionally to org members with edit access)

### Notification Content
- Which source released a new version (source chip with name and org)
- The new version ID and release date
- A summary of what changed: "[N] added, [N] retired, [N] modified concepts affect your collection"
- A direct link: "Review updates →" (launches the Update Collection workflow — see `03_workflows/update-collection-source-version.md`)

### Where Notifications Appear

**In-app notification bell (header)**
- Badge count on unread notifications
- Clicking opens the Notification Center

**Notification Center**
- List of all notifications for the user, newest first
- Each item shows: source chip, message, timestamp, "Review updates →" CTA
- Actions: Mark as read, Dismiss (hides from list but does not start the workflow)
- Notification persists as unread until the user either dismisses it or completes the update workflow

**Collection header banner**
- When a user navigates to a collection that has an unreviewed dependency update, a persistent yellow banner appears at the top of the collection page:
  > ⚠️ **[Source Name]** released a new version. [N] of your references may be affected. [Review updates →]
- Banner is dismissible per session; reappears on next visit until the update workflow is completed or dismissed permanently by the owner

**Email notification** (if enabled in user settings)
- Subject: "[Source Name] released a new version — review updates to [Collection Name]"
- Body: same content as in-app notification plus a direct link

### Notification List Item Format
```
[Source chip: CIEL / OpenConceptLab]
"CIEL released v2024-11-01 on Nov 1, 2024"
"23 concepts in your collection are affected (4 retired, 19 modified)"
[Review updates →]                          2 hours ago
```

### Staleness Indicator on Expansions
- Expansions that were built before the dependency change occurred are marked **stale**
- Stale indicator: a yellow dot + "Stale" badge next to the expansion in the Versions + Expansions tab
- Tooltip: "This expansion was built before [Source] released [versionID]. Rebuild to pick up changes."
- Rebuilding the expansion clears the stale indicator

---

## Version Locking Controls

### Context
Version locking (see `05_decisions/adrs.md` ADR-003 for full rationale) ensures a collection consistently resolves references to the same source version. The locking controls in this capability are the user-facing mechanisms.

### "Lock to Current Version" Action
- Available in the collection's References tab when unversioned references exist and a canonical source version is identified
- Applies the "Lock to Repo Version" transform to all unversioned references for the selected source
- Confirmation dialog showing: current canonical version, number of references that will be locked, preview of transformed expressions

### Canonical Source Version Indicator
- Shown in the collection header and Versions + Expansions tab
- Format: "CIEL: v2024-08-01" (or multiple entries if the collection references multiple sources)
- Clicking the indicator opens a summary of all referenced sources and their locked versions

---

## Permissions

| Action | Required Permission |
|---|---|
| View versions and expansions | Read access to repository |
| Create a version | Edit/owner access to repository |
| Release a version | Owner access to repository |
| Create custom expansion | Edit/owner access to collection |
| Rebuild expansion | Edit/owner access to collection |
| Set default expansion | Owner access to collection |
| Delete expansion | Owner access to collection |
| Dismiss dependency notification | Owner access to collection |
