# Workflow: Clean Existing References and Expansions

## Overview

Many of the benefits of collection expansions — particularly around staying current with upstream sources — only materialize when references are **dynamic** (i.e., unversioned or cascade-based). Dynamic references bring in newly added descendants automatically when the collection is re-expanded. This workflow describes how to migrate a collection's existing references toward a dynamic, cascade-based model, and then produce a clean expansion and release.

---

## When to Use This Workflow

- A collection was built using individual extensional references (specific concept or resource version URLs) and the owner wants to take advantage of cascading and dynamic resolution going forward
- Mapping references have accumulated that are now redundant because cascading handles them
- References were added via "Transform to Extensional" and need to be converted back to cascaded dynamic references
- Preparing a collection for a regular update cadence where re-expanding HEAD is the primary mechanism for picking up new upstream content

---

## Step 1: Convert Version-Pinned Concept References to Unversioned

**Goal:** Remove references that point to specific concept resource versions (deprecated pattern), replacing them with unversioned references.

- Back-end change may be needed to make resource-version-pinned references versionless by default (i.e., strip the `:resourceVersion` segment from stored expressions) — **open question, under discussion**
- In the UI, use the **Transform to Unversioned** action on resource-versioned references (see `02_capabilities/manage-references.md` → "Reference Transforms")
- Bulk transform is available: select all resource-versioned references → "Transform selected" → "Transform to Unversioned"

**Result:** References resolve to the concept's current (HEAD or canonical source version) content rather than a frozen historical version.

---

## Step 2: Add Cascade to Concept References (Optional, Run-Time Choice)

**Goal:** Ensure that mappings and related descendants are brought in automatically via cascade rather than requiring individual references.

- For each concept reference (or in bulk), set the **Cascade** option to "Source Mappings" — this adds all mappings where the concept is the "from concept" in its own source
- This is an **optional, per-reference run-time setting**, not a collection-level default
- **Pros and cons to consider before enabling:**
  - Pro: Mappings stay in sync with upstream; no need to maintain individual mapping references
  - Pro: New mappings added to the source are picked up on re-expansion without any reference changes
  - Con: The collection may grow in ways that are harder to audit if cascade scope is broad
  - Con: Some implementers may want explicit control over which mappings are included
- This setting can also be applied when adding new references via the "Add to Collection" dialog

---

## Step 3: Remove Individual Mapping References Handled by Cascading

**Goal:** Clean up redundant mapping references that are now covered by cascade.

- After enabling cascade (Step 2), the expansion will include mappings that are also referenced individually
- Identify duplicate mapping references: in the References tab, filter by type = Mapping and cross-reference against the cascade output
- Remove the now-redundant individual mapping references via the reference row action menu → "Remove reference"
- Bulk removal is available if many references qualify
- Confirm each removal; the expansion count will update after re-expansion

---

## Step 4: Convert "Transform to Extensional" References Back to Dynamic

**Goal:** Reverse any references that were previously "transformed to extensional" (i.e., flattened into individual resource URLs) and restore them as cascaded dynamic references.

- Extensional references created via a cascade transform are a snapshot in time; they do not pick up new content when re-expanded
- For each such reference group, remove the extensional references and re-add them as a single cascaded (intensional or dynamic unversioned) reference
- Use the **New Reference** flow (see `02_capabilities/manage-references.md` → "Creating References from Inside") to create the replacement dynamic reference with the appropriate cascade option
- Preview the new reference before saving to confirm it resolves to the expected content

---

## Step 5: Create a New Expansion of the Collection's HEAD Version

**Goal:** Evaluate the cleaned-up references against the current HEAD to produce an up-to-date snapshot.

- Navigate to the collection → Versions + Expansions tab → select HEAD → "New Expansion"
- Parameters: leave repo versions at default (resolves against canonical source versions), Active Only = on
- Optionally set "Set as Default" at creation time, or do so after reviewing (Step 6)
- The expansion will process asynchronously; status shows as "Processing" until complete

See `02_capabilities/manage-versions-and-expansions.md` → "Creating a Custom Expansion" for the full interaction flow.

---

## Step 6: Verify the New Expansion Looks Correct

**Goal:** Confirm the expansion's concept and mapping counts are as expected and no unintended content was added or lost.

- Once processing is complete, review the expansion's concept and mapping counts
- Browse the expansion to spot-check key concepts
- Use the "Trace back" feature on any concept to confirm it was brought in by the expected reference (see `02_capabilities/manage-references.md` → "Reference Detail")
- Compare against the previous expansion if available (see `02_capabilities/compare-resources.md`)

---

## Step 7: Set the New Expansion as the Default

**Goal:** Make the clean expansion the canonical resolved content for the collection's HEAD version.

- In the Versions + Expansions tab, find the new expansion row → action menu (⋮) → "Set as default"
- The previous default expansion is unset automatically
- The collection's browsable concept list now reflects the new expansion

---

## Step 8: Create a New Version of the Collection

**Goal:** Publish a stable, immutable snapshot that ties the cleaned references to specific source versions at a point in time.

- Versions + Expansions tab → "New Version" button
- Provide a version ID and description (e.g., release notes summarizing the reference cleanup)
- On creation, an auto-expansion is generated for the new version; this expansion resolves the references against the canonical source versions at the time of release
- The released version is immutable — it represents a single known version of each referenced source

See `02_capabilities/manage-versions-and-expansions.md` → "Creating a Version" for details.

---

## Ongoing: Keeping the Collection Current

Once this workflow has been completed once, the collection is in a healthy dynamic state. Future updates follow a simplified cycle:

1. **Create a new HEAD expansion** — re-evaluates all dynamic/cascade references against the latest upstream source content
2. **Verify it looks correct** — review counts and spot-check for unexpected changes
3. **Set the new expansion as default** (if it replaces a previous HEAD expansion)
4. **Create a new collection version** — releases the snapshot tied to the current source versions

No reference changes are needed unless the underlying sources change structure or the collection's scope changes.

---

## Related

- `02_capabilities/manage-references.md` — Reference transforms, cascade options, remove references
- `02_capabilities/manage-versions-and-expansions.md` — Creating expansions, setting default, creating versions
- `03_workflows/update-collection-source-version.md` — For updating to a new upstream source release
