# User Roles

## Role Definitions

### Terminology Consumer (TC)
**Who:** Clinicians, researchers, system analysts, developers evaluating or integrating terminology.

**What they care about:**
- Finding the right concept or code quickly
- Understanding what a concept means and how it maps to other standards
- Knowing which collection or source to use for their system
- Browsing without needing to sign in

**Permissions:** Read-only on all public content. Cannot create, edit, or manage any repository content.

**Key workflows:** Discover Terminology, Compare and Evaluate Content

---

### Terminology Implementer (TI)
**Who:** Health IT implementers, digital health project leads, OpenMRS/DHIS2 implementers building concept dictionaries for their deployment.

**What they care about:**
- Building and maintaining a concept dictionary based on CIEL or another source
- Adding concepts from CIEL to their collection with appropriate cascade
- Keeping their dictionary current when CIEL releases new versions
- Not breaking their system when updating terminology

**Permissions:** Read all public content. Create/edit repositories they own. Manage references, versions, and expansions within those repositories. Can submit concept proposals to sources they do not own.

**Key workflows:** Build Concept Dictionary, Add to Existing Dictionary, Update Collection to New Source Version, Propose and Manage Concepts (submit only)

---

### Terminology Publisher (TP)
**Who:** Canonical source maintainers — e.g., CIEL team, LOINC, national health authorities, FHIR IG authors.

**What they care about:**
- Authoring and releasing high-quality, canonical concept sources
- Reviewing and managing concept proposals from the community
- Controlling versioning and publication of their source
- Configuring their repository to expose correct FHIR resources

**Permissions:** All TI permissions plus: create/edit concepts and mappings in owned sources, approve/reject concept proposals, configure repo type and validation schema, manage org members.

**Key workflows:** Author and Publish Source, Propose and Manage Concepts (admin side)

---

### OCL Administrator (Admin)
**Who:** Server operators, OCL platform team.

**What they care about:**
- Managing user accounts
- Server-level configuration (branding, external server connections)
- Approving new users if approval-required mode is enabled

**Permissions:** All TP permissions plus server-level administration.

**Key workflows:** Admin/Server Settings (not in scope for v3 launch)

---

## Permission Matrix

| Action | TC | TI (own repo) | TI (other repo) | TP (own repo) | TP (other repo) | Admin |
|---|---|---|---|---|---|---|
| View public content | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ |
| Follow / subscribe to repos | | ✔ | ✔ | ✔ | ✔ | ✔ |
| Create a repository | | ✔ | | ✔ | | ✔ |
| Edit repo attributes | | ✔ | | ✔ | | ✔ |
| Add/edit concepts and mappings | | | | ✔ | | ✔ |
| Add references to collection | | ✔ | | ✔ | | ✔ |
| Create/release repo versions | | ✔ | | ✔ | | ✔ |
| Build/manage expansions | | ✔ | | ✔ | | ✔ |
| Submit a concept proposal | | ✔ | ✔ | ✔ | ✔ | ✔ |
| Review/approve concept proposals | | | | ✔ | | ✔ |
| Configure repo type/FHIR/validation | | ✔ | | ✔ | | ✔ |
| Manage org members | | | | ✔ (org admin) | | ✔ |

---

## Authenticated vs. Anonymous Access

- **Anonymous users** can browse all public repositories, search, and view concept/mapping details
- **Anonymous users** cannot follow repos, add to collections, create proposals, or access any edit functionality
- **Anonymous landing page** must have a "wow factor" — a curated feed of highlighted content and an option to take a guided tour
- **Session timeout** must be configurable; government deployments may require 24-hour sessions
