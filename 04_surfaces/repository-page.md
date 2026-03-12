# Surface: Repository Page

## Scope

Layout and navigation specification for the **Repository Detail Page** — the primary page for viewing and managing a single repository (Source or Collection). This page is the container for all repository-level tabs and the persistent header.

---

## Page Entry Points

- Search results → repository chip or row
- Global navigation → Org page → repository card
- Direct URL: `/{org}/{repo-type}/{repo-id}/`
- Notification link → specific tab within the repo
- Comparison drawer link share → repo + tab + resource pre-selected

---

## URL Structure

```
/{org}/{repo-type}/{repo-id}/                          → Concepts tab (default)
/{org}/{repo-type}/{repo-id}/concepts/                 → Concepts tab
/{org}/{repo-type}/{repo-id}/mappings/                 → Mappings tab
/{org}/{repo-type}/{repo-id}/references/               → References tab (Collections only)
/{org}/{repo-type}/{repo-id}/versions/                 → Versions + Expansions tab
/{org}/{repo-type}/{repo-id}/about/                    → About tab
/{org}/{repo-type}/{repo-id}/proposals/                → Proposals tab (if feature enabled)
/{org}/{repo-type}/{repo-id}/settings/                 → Settings (owners only)
```

Where `{repo-type}` is `sources` or `collections`.

Version context is applied via query parameter or path segment:
```
/{org}/{repo-type}/{repo-id}/concepts/?version=v2024-11-01
/{org}/{repo-type}/{repo-id}/concepts/?version=HEAD
```

Default: most recently released version if one exists; HEAD otherwise.

---

## Page Structure

```
┌──────────────────────────────────────────────────────────────────────┐
│  SHELL HEADER (global nav, search, user menu)                        │
├──────────────────────────────────────────────────────────────────────┤
│  REPO HEADER                                                         │
│  [Source|Collection chip]  Org / Repository Name                     │
│  Short description                                                   │
│  [Version selector ▾]  [Owner]  [Canonical URL]  [Star] [Follow]    │
│  [Dependency notification banner — conditional]                      │
├──────────────────────────────────────────────────────────────────────┤
│  TAB BAR                                                             │
│  Concepts  Mappings  [References]  Versions+Expansions  About        │
│  [Proposals]  [Settings ⚙]                                           │
├──────────────────────────────────────────────────────────────────────┤
│  TAB CONTENT AREA                                                    │
│  (fills remaining viewport; scrollable; split view occupies right    │
│   half when open)                                                    │
├──────────────────────────────────────────────────────────────────────┤
│  COMPARISON DRAWER (bottom, conditional — see design-system.md)      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Repository Header

### Always-Present Elements

| Element | Notes |
|---|---|
| Type chip | "Source" (green) or "Collection" (purple) |
| Breadcrumb | Org name → Repo name; org name is a link |
| Repository name | H1; not a link |
| Short description | Single line; truncated at 120 chars with expand |
| Version selector | Dropdown showing all released versions + HEAD; changes version context across all tabs |
| Owner | User/org avatar + name; links to org page |
| Canonical URL | Displayed as a copyable chip if set; links to external resource |
| Star / Follow | For authenticated users |
| Action button | "Edit" (owners/editors); "Fork" (authenticated non-owners) |

### Conditional Elements

**Dependency notification banner** (yellow, dismissible)
- Shown when: collection has an unreviewed dependency update from an upstream source release
- Content: "⚠ [Source Name] released [versionID]. [N] references may be affected. [Review updates →]"
- Dismiss: hides for the session; reappears on next visit until update workflow is completed
- See `02_capabilities/manage-versions-and-expansions.md` for full notification logic

**Validation error banner** (red, persistent)
- Shown when: collection has one or more Error-level validation violations
- Content: "❌ This collection has [N] validation errors. [View errors →]"
- Not dismissible until errors are resolved or suppressed

**HEAD editing indicator** (blue chip)
- Shown when version context = HEAD
- Content: "Viewing HEAD — changes here affect the next version"

---

## Tab Bar

### Tab Definitions

| Tab | Available For | Default? | Notes |
|---|---|---|---|
| **Concepts** | Source, Collection | ✅ Yes | Lists concepts in current version context |
| **Mappings** | Source, Collection | No | Lists mappings in current version context |
| **References** | Collection only | No | Lists references; primary authoring surface for collections |
| **Versions + Expansions** | Source (Versions only), Collection | No | Full spec: `04_surfaces/versions-expansions-tab.md` |
| **About** | Source, Collection | No | Metadata, description, linked resources, FHIR metadata |
| **Proposals** | Source, Collection (if feature enabled) | No | Concept proposal queue; admin and proposer views |
| **Settings ⚙** | Source, Collection | No | Owner/editor only; gear icon; not shown to read-only users |

### Tab Visibility Rules
- **References** tab: only shown for Collections (never Sources)
- **Proposals** tab: shown when the Concept Proposals feature is enabled for the org; visibility to non-owners is controlled by org settings
- **Settings** tab: only shown to users with owner or editor permission on the repo

### Active Tab Indicator
- Underline on active tab label
- Active tab label is bold
- URL updates to reflect the active tab (see URL structure above)

---

## Concepts Tab

### Layout
```
[Search/filter bar]                          [+ Add Concept]  [⋮ More]
─────────────────────────────────────────────────────────────────────
[Concept row]  Name  Class  Datatype  Locale  Updated  [Actions ▾]
[Concept row]
...
[Pagination / Load more]
```

### Filter Bar
- Text search (searches names and synonyms in the current locale)
- Class filter (multi-select dropdown)
- Datatype filter (multi-select dropdown)
- Locale filter (multi-select dropdown)
- Status filter (Active / Retired / All)
- "More filters" expander: custom attributes, date range, source filter (for collections)

### Concept Row
- Clicking a row opens the Concept Detail panel (split view) — see `04_surfaces/concept-detail.md`
- Row actions: View (opens split view), Edit (opens edit form), Add to Comparison (adds to Comparison Drawer queue)
- Retired concepts: greyed out text + "Retired" badge

### Empty State
```
No concepts found.
[Add first concept]  [Adjust filters]
```

---

## Mappings Tab

### Layout
Same pattern as Concepts tab.

### Columns
- From Concept | Map Type | To Concept / To Source Code | Sort Weight | Status | Actions

### Filter Bar
- Map Type filter (multi-select)
- From Concept search
- To Concept / Code search
- Status filter (Active / Retired)

### Mapping Row
- Clicking a row opens a mapping detail side panel (lighter than concept detail; no full split view)
- Row actions: View, Edit, Add to Comparison

---

## References Tab (Collections Only)

### Layout
```
[Search/filter bar]  [Validation status: ❌ N errors  ⚠ N warnings]
[+ Add References]  [Transform ▾]  [Validate]  [⋮ More]
─────────────────────────────────────────────────────────────────────
[Reference row]  Expression | Resolved Concept | Type | Status | [⋮]
[Reference row]
...
```

### Reference Row
- Expression: the reference expression (e.g., `/orgs/CIEL/sources/CIEL/concepts/116128/`)
- Resolved Concept: display name of the resolved concept in the current version context; "Unresolved" if broken
- Type: Extensional / Intensional
- Status: validation severity badge (❌ / ⚠ / ✅ / —)
- Actions menu: View concept, Edit reference, Transform, Remove

### Validation Summary Bar
- Shows counts of Error and Warning violations for the full reference set
- Links to the Validation Report Panel (see `02_capabilities/validate-content.md`)
- Updates in real-time as background validation runs

### Add References Entry Points
- "+ Add References" button: opens the Add References drawer (from outside collection context; full search + preview flow)
- "+ New Reference" quick action (in action menu): opens the New Reference dialog (from inside collection context; manual expression entry)
- Both documented in `02_capabilities/manage-references.md`

---

## About Tab

### Sections
- **Description**: full markdown description (rendered); edit inline for owners
- **Details**: Created, Updated, Canonical URL, Supported Locales, Default Locale, Custom Validation Schema, FHIR resource type (if applicable)
- **External Links**: up to 5 linked resources with labels
- **Statistics**: Concept count, Mapping count, Reference count (collections), Version count, Subscriber count

---

## Proposals Tab

Full spec in `02_capabilities/manage-concept-proposals.md`.

### Proposer View (authenticated, non-admin)
- List of proposals they have submitted, with status
- "+ New Proposal" button

### Admin View (repo owner/admin)
- Full queue of all proposals with pending count
- Filter by status (Pending / Under Review / Approved / Rejected)
- Bulk actions: Assign reviewer, Reject selected

---

## Settings Tab (Owners Only)

Not fully specced in this document. Key sections:

| Section | Content |
|---|---|
| General | Name, description, canonical URL, external ID, default locale, supported locales |
| Access | Visibility (public/private), member access list, permission levels |
| Validation | Schema rules, validation templates — see `02_capabilities/validate-content.md` |
| Versions | Default version behavior, auto-release settings |
| Danger Zone | Archive, transfer ownership, delete |

---

## Split View Behavior

When a concept or mapping is selected (clicked) anywhere on this page:
- Right half of the tab content area becomes the split-view panel
- Left half shows the list (narrowed)
- Split view panel shows the Concept Detail surface (`04_surfaces/concept-detail.md`)
- Closing the split view (X button or Esc) returns to full-width list
- Split view state is reflected in the URL (concept ID appended as query param)

---

## Responsive Behavior

| Viewport | Behavior |
|---|---|
| ≥ 1200px | Two-column split view available; full tab bar |
| 900–1199px | Split view stacks (concept detail below list); tab bar scrolls horizontally if needed |
| < 900px | No split view; concept detail navigates to a new full-page view; tabs collapse to dropdown menu |

---

## Relationship to Other Files

- `04_surfaces/concept-detail.md` — split-view panel content for concepts
- `04_surfaces/versions-expansions-tab.md` — full spec for the Versions + Expansions tab
- `04_surfaces/design-system.md` — shell layout, comparison drawer, chips, action menus, forms
- `02_capabilities/manage-references.md` — add/edit/transform references (References tab)
- `02_capabilities/manage-versions-and-expansions.md` — create/release versions, expansions, dependency notifications
- `02_capabilities/validate-content.md` — validation badges, validation report panel (References tab)
- `02_capabilities/manage-concept-proposals.md` — Proposals tab content
- `02_capabilities/configure-repository.md` — Settings tab content
