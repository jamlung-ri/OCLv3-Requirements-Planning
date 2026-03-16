# Capability: Compare Resources

## Scope

Side-by-side comparison of any two OCL resources: concepts, mappings, repository versions, or expansions. Comparison is non-destructive and non-navigating — it uses a bottom drawer that overlays the current view without replacing it.

---

## Comparison Drawer

### Behavior
- The Comparison Drawer slides up from the bottom of the screen
- The background view (search results, repository browser, etc.) remains fully interactive
- The drawer occupies the bottom ~50% of the screen by default; resizable by dragging the top edge
- The drawer can be minimized to a slim "comparison bar" at the very bottom (showing what's queued)
- Closing the drawer clears the comparison state entirely

### Comparison Bar (Minimized State)
- Shown at the bottom of the screen whenever there is at least one item queued for comparison
- Shows: chips for each queued item (up to 2) + "Compare" button + "Clear" button
- If only one item is queued: "Compare" button is disabled; shows "Add one more item to compare"
- The bar does not obstruct the main content area (content above it is scrollable)

---

## Adding Items to Compare

### Entry Points (Concepts and Mappings)
- Concept detail (split view or full page) → "Add to Comparison" button
- Concept row in search results → action menu → "Add to Comparison"
- Concept row in repo browser → action menu → "Add to Comparison"
- Mapping detail → "Add to Comparison"

### Entry Points (Repository Versions and Expansions)
- Repository → Versions tab → version row → "Compare" action button or checkbox
- Repository → Versions + Expansions tab → expansion row → "Compare" action

### Queue Rules
- Maximum of 2 items in the comparison queue at once
- Items in the queue must be of compatible types: both concepts, both mappings, or both versions/expansions
- Attempting to add a concept when a version is already queued: show inline message "Replace the current comparison with this concept?" with Yes/No
- When 2 items are queued, clicking "Add to Comparison" replaces the oldest item

### Version Selector (for Concepts/Mappings in Comparison)
- When comparing two concepts or mappings, a version selector dropdown appears above each column
- Allows the user to compare the same concept/mapping at two different versions (e.g., CIEL v2023 vs. CIEL v2024)
- Default: current version context

---

## Comparison Table Layout

### Concept Comparison

| Section | Left Column | Right Column |
|---|---|---|
| **Header** | Concept chip (name + ID + source) | Concept chip |
| **Core Properties** | Concept Class, Datatype | Same |
| **Names** | All names, grouped by locale | Same |
| **Descriptions** | All descriptions | Same |
| **Mappings** | All associations | Same |
| **Custom Attributes** | All extras | Same |

- **Highlighting**: Fields that differ between left and right are highlighted (yellow background or colored border)
- **Same** fields are shown without highlight
- Missing fields (present in one but not the other) shown as "—" in the absent column

### Version / Expansion Comparison

| Section | Left Column | Right Column |
|---|---|---|
| **Header** | Version/expansion chip | Version/expansion chip |
| **Summary** | Concept count, mapping count | Same |
| **Parameters** (expansions) | Repo versions, filters | Same |
| **Content diff** | Concepts only in this version | Concepts only in the other version |

- Summary statistics: "X concepts in common, Y only in left, Z only in right"  
- Content diff is browsable (paginated list within the drawer)

---

## Integrated Actions (Within Comparison)

The comparison drawer is not read-only. Contextually appropriate actions are available without closing the drawer:

| Context | Available Action |
|---|---|
| Concept comparison | "Add [left/right concept] to collection" |
| Concept comparison | "Open in new tab" |
| Concept comparison | "Propose edit" (if authenticated) |
| Version comparison | "Set as default expansion" (if authorized) |

---

## Link Sharing

- The comparison state is reflected in the URL as query parameters: `?compare=[url1]&compare=[url2]`
- Sharing the URL opens TBv3 with the drawer pre-loaded showing those two items, with repository in focus behind the drawer if available
- If either item is not accessible to the viewer (e.g., private repo), a graceful error is shown in that column

---

## Exiting Comparison

- Closing the drawer via X button: clears comparison state, URL params removed
- Navigating away: the drawer persists (comparison bar shown minimized) until explicitly closed
- Pressing Escape: minimizes to comparison bar (does not clear the queue)

---

## Constraints and Non-Goals (v3)

- Cross-server comparison (comparing resources from two different OCL instances) is out of scope for v3
- Comparison of more than 2 items is out of scope for v3
- AI-powered "smart summary" of differences is a post-v3 feature

---

## Accessibility

- Comparison drawer is keyboard-navigable
- "Add to Comparison" button is discoverable without hover (not icon-only)
- Screen reader announces when an item is added to the comparison queue
