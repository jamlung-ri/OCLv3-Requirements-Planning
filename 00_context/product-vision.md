# Product Vision

## What OCL Is

Open Concept Lab (OCL) is a cloud-based, open-source platform for publishing, managing, and consuming standardized medical terminology. It enables health information systems to share a common clinical language by aligning concepts, codes, and mappings across standards: FHIR, OpenMRS, CIEL, LOINC, SNOMED, and others.

## What TBv3 Is

TermBrowser v3 (TBv3) is the next-generation React SPA for OCL, replacing the TBv2 interface. It is not a monolith — it is a set of composable UI capabilities organized around a shared application shell. The goal is to make OCL accessible to a wider, less-technical audience while giving power users full control.

## Anchoring Outcome (SOW)

The primary funded outcome is:

> Streamlined access to CIEL (TBv3), management of CIEL-based dictionaries (using collections) in TBv3, and basic support for the primary OpenMRS/CIEL community dictionary management OCL Mapper use cases with CIEL.

This means: a Terminology Implementer working with OpenMRS and CIEL must be able to **build, manage, and update a concept dictionary** entirely within TBv3 — without needing to use the API directly or rely on workarounds from TBv2.

## Success Criteria

1. An implementer can browse CIEL (or LOINC) and add concepts to a new or existing collection without leaving the UI
2. References can be previewed before saving, and their results can be traced back to the defining reference
3. A collection can be versioned and expanded with intuitive controls
4. When CIEL releases a new version, the implementer is notified and can review, compare, and accept updates through a structured workflow
5. A source administrator can receive, review, and act on concept proposals from community members
6. The comparison tool allows any two concepts, versions, or expansions to be placed side by side without losing navigational context

## What TBv3 Is Not (v3 Scope Boundary)

- Multi-server / cross-server terminology management
- Bulk import (handled separately)
- OCL Mapper (separate product; TBv3 integrates with it via Concept Proposals)
- Full FHIR Implementation Guide authoring
- Highly customized org/repo visual configuration
