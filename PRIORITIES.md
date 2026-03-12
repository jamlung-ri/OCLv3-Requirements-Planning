# Requirements Library — Priorities & Next Steps
_Anchored to [CIEL Implementers] GitHub Milestones · March 2026_

---

## Milestone Overview

The five open [CIEL Implementers] milestones define the near-term showcase cadence. In due-date order:

| Milestone | Due | Open Issues | Focus |
|---|---|---|---|
| M42 · Collection & Reference Creation E2E | Apr 17, 2026 | 11 | Create collection, add references, lock, transform, validate |
| M43 · Team-based Mapping & Left Panel E2E | Apr 30, 2026 | 5 | Collaborative mapping, smart locking, team dashboard |
| M44 · Collection Maintenance (CIEL Release) | TBD | 3 | Update collection when CIEL releases new version |
| M45 · Concept Proposals E2E | May 29, 2026 | 2 | Propose → review → approve → concept created |
| M59 · Mapping Decision Memory | TBD | 3 | AI-assisted mapping memory and recall |

---

## Coverage Assessment

### ✅ Well-covered by the current library

| Milestone | Primary requirements files |
|---|---|
| M42 | `02_capabilities/manage-references.md` (add from outside, add from inside, preview, transforms, version locking), `02_capabilities/manage-versions-and-expansions.md`, `03_workflows/build-concept-dictionary.md`, `03_workflows/add-to-existing-dictionary.md` |
| M44 | `03_workflows/update-collection-source-version.md` (notification → comparison → accept → apply → new version), `02_capabilities/manage-versions-and-expansions.md` |
| M45 | `01_objects/concept-proposal.md`, `02_capabilities/manage-concept-proposals.md`, `03_workflows/manage-concept-proposals-workflow.md` |

### ⚠️ Partially covered — needs targeted expansion

**M42 / Schema validation (#2346):** The `manage-references.md` capability covers reference transforms and preview, but real-time validation against schemas (error/warning/info severity, inline warnings, validation rules per repo) is not yet specified. The placeholder `02_capabilities/validate-content.md` was identified as a gap and should be written for this milestone.

**M44 / Dependency change notifications:** `update-collection-source-version.md` covers the review-and-accept workflow (Tracker #63), but the upstream notification trigger (Tracker #62) — how TBv3 detects and surfaces that a source dependency has changed — needs a dedicated section or its own capability file.

### ❌ Not covered — outside current scope

**M43 · Team-based Mapping & Left Panel** is primarily an OCL Mapper feature (team roles, record locking, assignment, activity feed, discussion threads). The TBv3 requirements library does not cover Mapper internals. No action needed in this library unless TBv3 surfaces (e.g., left panel concept browsing) are affected.

**M59 · Mapping Decision Memory** is an AI/ML capability with no current home in the library. If TBv3 surfaces a decision memory UI (e.g., suggested mappings, decision audit trail), a new capability file will be needed at that time.

---

## Priorities for the Next Library Sprint

Listed by impact on upcoming showcases.

### 1 · Write `02_capabilities/validate-content.md` _(M42, due Apr 17)_
The highest-priority gap. Should cover:
- Validation rules: required properties, allowed concept classes/datatypes, mapping constraints, cardinality
- Validation triggers: on-blur in reference form, on Preview, continuous background for existing references
- Severity model: Error (blocks expansion/version) / Warning / Info
- Warning display locations: inline in form, preview panel, References tab, collection summary badge
- User actions: fix inline, ignore with confirmation, export validation report, filter by validation status
- Schema management: per-repo/per-collection rule configuration; OpenMRS and FHIR templates

### 2 · Expand `02_capabilities/manage-versions-and-expansions.md` — dependency notifications _(M44)_
Add a **Dependency Change Notifications** section (maps to Tracker #62) covering:
- What triggers a notification (linked source version advances past what the collection references)
- How TBv3 surfaces the notification (collection header banner, notification center, email)
- Notification payload: which source changed, old vs new version, number of affected references
- Entry point into the update-source-version workflow from the notification

### 3 · Add `04_surfaces/versions-expansions-tab.md` _(M42 + M44)_
The two-panel layout spec (Versions left, Expansions right) is referenced throughout capability and workflow files but never written. Should cover:
- Panel layout and responsive behavior
- Version list: columns, status badges, action buttons (Create Similar, Rebuild, Lock controls)
- Expansion list: columns, stale indicator, async rebuild status
- Empty states
- This tab is central to both the M42 and M44 showcases.

### 4 · Write `04_surfaces/repository-page.md` _(foundational for all milestones)_
All showcase workflows land users on repository pages. The tab structure (Concepts, Mappings, References, Versions & Expansions, About, Proposals) needs a spec that cross-references the existing capability files. Without this, developers have no single authoritative layout document.

### 5 · Review `03_workflows/manage-concept-proposals-workflow.md` against M45 milestone description _(M45, due May 29)_
The milestone's showcase story is detailed and specific:
1. Mapper searches, confirms no concept exists
2. Creates proposal with justification and suggested mappings
3. Maintainer notified, reviews, discusses, approves
4. Concept created in CIEL; mapper notified
5. Mapper maps to the new concept and saves to local collection

The workflow file should be validated against each step to confirm it is complete and that cross-system behavior (Mapper → TBv3 shared database, `origin` field) is accurately described.

---

## Suggested Work Order

```
Week 1 (before Apr 17 showcase)
  → validate-content.md (new)
  → manage-versions-and-expansions.md: add notification section

Week 2 (before Apr 30 showcase)
  → versions-expansions-tab.md (new surface spec)
  → repository-page.md (new surface spec)

Week 3–4 (before May 29 showcase)
  → Review and tighten manage-concept-proposals-workflow.md vs M45
  → Verify concept-proposal.md covers TBv3 + Mapper shared-database behavior
```

---

## What to Do With M43 and M59

- **M43** — No library action needed now. If TBv3's left panel browsing UX changes as a result of team-mapping work, update `browse-repository.md` and `04_surfaces/design-system.md` accordingly.
- **M59** — Monitor. If a Mapping Decision Memory UI surface is planned for TBv3, open a new `02_capabilities/mapping-decision-memory.md` file at that time.
