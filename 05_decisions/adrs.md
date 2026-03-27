# Architecture Decision Records

> **Note:** Jon to review all ADRs and add additional records for open architectural questions surfaced during requirements planning (e.g., Canonical URLs, Linked Sources). This is an ongoing document.

---

## ADR-001: Capability + Object + Workflow Structure

**Status:** Accepted

**Context:**  
Earlier requirements documentation used a "Mini-app" organizational structure that reflected development sprint planning rather than product requirements. This structure caused fragmentation of shared behaviors, inheritance of outdated organizational strategies, and difficulty using the docs as AI context for design/dev/testing.

**Decision:**  
Organize requirements into four layers:
- `01_objects/` — stable data schemas and business rules (what the system knows about)
- `02_capabilities/` — discrete user-facing functions with prescriptive UI interaction detail
- `03_workflows/` — end-to-end user journeys that sequence capabilities into meaningful tasks
- `04_surfaces/` — page-level and component-level UI specs

**Consequences:**
- A new developer working on any feature loads: relevant `01_objects/`, relevant `02_capabilities/`, relevant `03_workflows/`, and relevant `04_surfaces/` files
- Shared behaviors (search, forms, validation, chips) live in one place and are referenced, not repeated
- Workflows are directly traceable to SOW trackers

---

## ADR-002: Comparison Tool as an Independent Component

**Status:** Accepted

**Context:**
Users need to compare two concepts or two version expansions without losing their current navigation context. A modal blocks interaction. A new page loses context entirely. A side-panel would compete with the existing split view. Early designs proposed a fixed bottom drawer, but this constrains the comparison surface to one presentation mode.

**Decision:**
The comparison tool is an **independent component** that is not tied to any specific placement on the screen. It may be rendered as a modal, a bottom drawer, a side drawer, or a full page — depending on the context in which it is invoked and the screen space available. The component itself does not assume its container.

**Consequences:**
- The comparison surface can be invoked from search results, concept detail, version expansions, or any other context without dictating screen layout
- The component must be self-contained (no dependency on surrounding page state)
- Specific placement decisions (e.g., when to use a drawer vs. a modal) are left to the design system and individual surface specs
- Cross-server comparison remains out of scope
- The comparison bar / queue reminder pattern from the original bottom-drawer design remains a valid implementation option but is not required

---

## ADR-003: Version Locking Strategy for Collections
Note: Current UI exposes basic information, but it isn't made intelligent and intuitive for users.

**Status:** Accepted

**Context:**  
Collections using unversioned references to CIEL (e.g., `/orgs/CIEL/sources/CIEL/concepts/1234/`) resolve to different content depending on which version of CIEL is current at the time of expansion. This means a collection's expansion can silently change content when CIEL releases a new version, without the collection owner being aware. This creates clinical safety risks for implementations that assume terminology is stable.

**Decision:**  
Implement version locking with a visual indicator approach (Approach 2 from the July 2025 squad discussion):

1. When a collection first resolves an unversioned reference to a source, the resolved source version becomes the **canonical source version** for that collection
2. All expansions use this canonical source version by default
3. When the underlying source releases a new version:
   - Notify collection owners
   - Show visual indicator: concepts that are in both versions with identical smart checksums appear normal; concepts that have changed (different smart checksum) are visually flagged
   - The **Update Collection** workflow guides owners through reviewing and accepting changes
4. Locking controls: "Lock to Current Version" transform, "Rebuild" expansion, "Create Similar" expansion

**Consequences:**
- Collection owners have control over when their content changes
- The system prevents silent content drift
- The Update Collection workflow (see `03_workflows/update-collection-source-version.md`) becomes a critical path feature
- Resource-versioned references (the deprecated pattern) are never created by the UI; existing ones are migrated via the Transform capability

---

## ADR-004: Reference Modeling Best Practices Enforced by UI

**Status:** Accepted

**Context:**  
OCL supports three reference patterns: unversioned, repo-versioned, and resource-versioned. Resource-versioned references (pinning to a specific resource version) are deprecated — they create fragile, hard-to-maintain collections and are the root cause of many "version chaos" issues.

**Decision:**  
The TBv3 UI must never offer the user a way to create a new resource-versioned reference. The only reference patterns the UI creates are:
- Unversioned references (default; recommended for extensional references)
- Repo-versioned references (available when the user explicitly needs to pin to a source version)
- Intensional (query-based) references

Existing resource-versioned references can be viewed and transformed, but not created anew.

**Consequences:**
- Reduces future migration burden
- Users who need resource-versioned references for special cases must use the API directly
- The Transform capability (`02_capabilities/manage-references.md`) provides a migration path for existing deprecated references

---

## ADR-005: Default Repository Version on Navigation

**Status:** Accepted

**Context:**
When a user navigates to a repository URL without specifying a version, which version should TBv3 show? Showing HEAD is confusing for most users (who want to see stable, published content). Showing a non-current released version is also confusing.

**Decision:**
On first navigation to a repository (without a version in the URL):
1. Redirect to the **latest released version** if one exists
2. Redirect to the **latest non-HEAD version** (draft) if there are no released versions
3. Show **HEAD** only if no other versions exist

This must be implemented as a server-side redirect (or client-side routing redirect) so the version is always reflected in the URL.

**Consequences:**
- Users immediately see stable content, not in-progress drafts
- Deep links always include the version, making them stable and shareable
- Owners editing HEAD must explicitly switch to HEAD in the version dropdown
- The version dropdown must show HEAD at the bottom in a visually distinct style

**Future (post-v3):** An explicit "Edit" mode action on the repository page — distinct from navigating the version dropdown — that switches the user directly to HEAD. This would make the editing intent explicit and reduce confusion about how to access in-progress content.

---

## ADR-006: Concept Proposals as First-Class Resources

**Status:** Accepted

**Context:**  
Community members using OCL (particularly OpenMRS implementers) frequently need concepts that don't exist in CIEL. Currently, this is managed via ad-hoc email, GitHub issues, and the OCL Mapper — with no structured tracking in TBv3.

OCL Mapper also generates concept proposals but they are not surfaced in TBv3.

**Decision:**  
Implement Concept Proposals as a first-class API resource with CRUDS operations. Both TBv3 and OCL Mapper write to the same proposals database. Source administrators in TBv3 manage all proposals regardless of origin. The proposal object includes an `origin` field to distinguish source.

**Consequences:**
- Single source of truth for all concept proposals across OCL products
- TBv3 becomes the primary proposal management interface for source admins
- Mapper retains its submission UI but routes to the shared backend
- Notification system must handle proposal status changes across origins
