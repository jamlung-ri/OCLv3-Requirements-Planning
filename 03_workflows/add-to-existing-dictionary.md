# Workflow: Add to an Existing Concept Dictionary

## Purpose

A Terminology Implementer already has a concept dictionary collection in OCL and wants to add more concepts — for example, when a new data entry form is being digitized, or when a new program area requires additional terminology.

**This is the "adding to existing" variant of the Build Concept Dictionary workflow, per the SOW.**

---

## Roles

- **Primary actor**: Terminology Implementer (collection owner or editor)

---

## Entry Conditions

- User is authenticated
- Collection already exists with at least one reference and has a canonical source version established (e.g., locked to CIEL v2024-08-01)
- User knows which new concepts are needed (by searching CIEL) or has a list of concepts to add

---

## Steps

### 1. Identify Needed Concepts

Same as Build workflow steps 2/3. User browses or searches CIEL to find the concepts they need.

**Important:** Since the collection already has a canonical CIEL version, TBv3 must guide the user toward browsing the **same CIEL version** that the collection is locked to. This avoids version consistency conflicts.

- The collection context chip should be visible when browsing CIEL ("Adding to [collection name], locked to CIEL v2024-08-01")
- If the user is browsing a different CIEL version, a warning banner is shown: "You're browsing CIEL HEAD. Your collection uses v2024-08-01. Concepts you add will trigger a version consistency warning."

---

### 2. Add Concepts via "Add to Collection"

Same as Build workflow step 3. Version consistency warning applies if the user is browsing a version that differs from the canonical.

**Version consistency dialog (if triggered):**
- "This concept is from CIEL HEAD, but your collection uses CIEL v2024-08-01."
- Options:
  - "Add as unversioned (recommended) — will resolve to v2024-08-01"
  - "Add versioned to HEAD — not recommended; may cause inconsistency"
  - "Cancel"

---

### 3. Add via New Reference (for Intensional/Bulk Additions)

If the user needs to add all concepts of a specific class or matching a query:
1. Collection → References tab → "New Reference"
2. Select: Intensional reference
3. Configure the query (e.g., source = CIEL, concept_class = "Diagnosis", name contains "Malaria")
4. Preview results before saving
5. Save → references added to HEAD; auto-expansion updated

---

### 4. Review the Changes

1. Collection → Concepts tab — verify new concepts appear
2. Check References tab — verify reference expressions are correct (unversioned for new additions)
3. Check HEAD auto-expansion count has increased

---

### 5. Optionally: Create a New Version

If this is a significant update:
1. Versions + Expansions tab → "New Version"
2. Name the version (e.g., "v2024-Q2" or "2024-01-15-added-nutrition")
3. Release → downstream systems are notified

If this is a minor addition (pre-release iteration), the user may defer version creation.

---

## Differences from "Build New Dictionary"

| Aspect | Build New | Add to Existing |
|---|---|---|
| Canonical source version | Not yet established; set on first add | Already established; must respect |
| Version consistency warnings | Not applicable on first add | Active for every add |
| Collection structure | Created from scratch | Existing structure preserved |
| Version creation | Required to go live | Optional; may batch multiple additions into one version |
