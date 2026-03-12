# Workflow: Update Collection to Latest Source Version

## Purpose

A Terminology Implementer has an existing concept dictionary (collection) built on CIEL. CIEL has released a new version. The TI wants to review what changed, decide what to accept, and update their dictionary to use the new CIEL version — without breaking their system or silently including unwanted changes.

**This is the primary version locking / collection update use case in the SOW.**

---

## SOW Coverage

This workflow directly implements:
- Notifications of changes of dependencies [Tracker: 62]
- Workflow for reviewing and accepting updates, possibly including Before/after comparison [Tracker: 63]
- Intuitive controls for collection version locking (Rebuild, Create similar expansion) [Tracker: 1]
- Linked source: Workflow to resolve references to HEAD when appropriate (during updates) and transition to released repos upon version creation/release [Tracker: 60]

---

## Roles

- **Primary actor**: Terminology Implementer (collection owner)
- **Supporting actors**: System (notifications, version resolution, expansion computation)

---

## Entry Conditions

- User owns or edits a collection that references CIEL (or another source)
- CIEL has released a new version since the collection's canonical source version was last set

---

## Trigger

**Notification:** When CIEL releases a new version, TBv3 sends an in-app notification (and optionally email) to owners of collections that reference CIEL.

Notification content:
- "[CIEL] released a new version: v2025-01-15"
- "Your collection [name] references CIEL. Review what changed and update your collection."
- [Review Updates →] CTA button

---

## Steps

### 1. Review the Notification

1. User sees notification bell badge in the app header
2. Opens notification center
3. Sees the CIEL new version notification
4. Clicks "Review Updates →" — navigates to the collection's update review flow

---

### 2. Version Comparison (What Changed in CIEL?)

**Entry point:** Notification CTA, or Collection → Versions + Expansions tab → "Review Source Updates" banner

**The comparison view shows:**
- **Left**: Collection's current canonical CIEL version (e.g., v2024-08-01)
- **Right**: New CIEL version (e.g., v2025-01-15)

**Summary statistics:**
- Concepts in current version: [N]
- Concepts in new version: [N]
- Concepts added since current version: [+N] — shown as a browsable list
- Concepts retired since current version: [-N] — shown as a browsable list
- Concepts with changed names or mappings: [~N] — shown as a browsable list

**Browsable diff list:**
- Each row: Concept chip (name + ID) | Change type (Added / Retired / Modified)
- "Added" concepts: checkboxes (user can deselect concepts they do NOT want to accept)
- "Retired" concepts: warnings (user decides how to handle — remove from collection or retain as-is)
- "Modified" concepts: side-by-side diff of what changed (names, mappings, class, datatype)

**Key design principle:** The user must feel in control. They are not blindly accepting everything — they are reviewing and making informed decisions.

---

### 3. Select Changes to Accept

User decisions per change type:

**For Added concepts:**
- Default: accept all (checkboxes checked)
- User may uncheck individual concepts to exclude them
- Bulk: "Deselect all" / "Reselect all"

**For Retired concepts:**
- Three options per concept (or bulk):
  - **Remove from my collection** — deletes the reference; this concept will not appear in next version
  - **Keep in my collection** — retains the reference; the concept will appear as "Retired" in the expansion
  - **Review later** — flag for follow-up; treated as "Keep" for now but added to a to-do list

**For Modified concepts:**
- **Accept change** — when the collection is updated, the new version of the concept will be used
- **Keep current version** — lock this specific reference to the old CIEL version (creates a version-pinned reference)

---

### 4. Apply Changes to HEAD

1. User reviews their selections and clicks "Apply to My Collection"
2. Confirmation dialog:
   - Summary: "Adding [N] concepts, removing [N] references, keeping [N] as retired, locking [N] to current version"
   - "Apply changes" | "Cancel"
3. System applies:
   - New references added to collection HEAD (for accepted "Added" concepts)
   - References removed from HEAD (for "Remove" decisions on retired concepts)
   - References to modified concepts updated to be unversioned (will resolve to new CIEL version)
   - References locked to old version where user chose "Keep current version"
4. Auto-expansion of HEAD is re-queued
5. Collection's canonical source version is updated to the new CIEL version

---

### 5. Review and Verify HEAD

1. User is taken to the collection's Concepts tab
2. Browse or search to spot-check the updated content
3. Check the expansion (Versions + Expansions tab → HEAD → auto-expansion) once it finishes computing
4. Optionally: use "Before/after comparison" — compare the current HEAD expansion against the previous released version's expansion

---

### 6. Create a New Collection Version

When satisfied:
1. Versions + Expansions tab → "New Version"
2. Enter version ID (e.g., "v2025-01-15")
3. On version creation:
   - All unversioned CIEL references transition to the new CIEL released version (v2025-01-15) in the expansion parameters
   - New auto-expansion computed
4. Release the version

---

## Post-Completion State

- Collection has a new released version based on the updated CIEL version
- Old version is still accessible and unchanged
- Notification is dismissed
- Any "Review later" items remain in a to-do list for the user

---

## Error and Edge Cases

| Situation | Behavior |
|---|---|
| CIEL new version has no changes relevant to this collection | Notification is shown with "No content in your collection changed in the new CIEL version. You may still update your canonical version." |
| User dismisses notification without acting | Notification persists in notification center; banner remains on collection page |
| HEAD collection has uncommitted changes | Warn user before applying update; offer to show what changed in HEAD first |
| Expansion computation fails after applying changes | Error shown; Rebuild button available; does not roll back the reference changes |
| Multiple source updates pending | Each source is shown as a separate update flow; the collection may batch all into one new version at the end |
