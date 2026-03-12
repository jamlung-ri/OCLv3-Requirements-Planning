# Workflow: Propose and Manage Concepts

## Purpose

Covers the full lifecycle of a concept proposal — from a community member identifying a gap in CIEL (or another source) and submitting a proposal, through the source administrator reviewing and acting on it.

**This is the Concept Proposals MVP per the SOW [Tracker: 27].**

---

## Roles

- **Primary actor (submitter)**: Terminology Implementer, or any authenticated user
- **Primary actor (reviewer)**: Terminology Publisher / Source Administrator
- **Supporting actor**: OCL Mapper (proposals submitted via Mapper appear in the same queue)

---

## Entry Conditions

- Proposer: authenticated; has read access to the target source
- Reviewer: authenticated; is an owner or editor of the target source

---

## Part 1: Submitting a Proposal (Proposer)

### Scenario: A concept is missing from CIEL

1. User is building a dictionary and cannot find the concept they need in CIEL
2. From search results: "No results for [term]" → "Propose this concept to CIEL" CTA
   - Or: from CIEL source page → "Propose New Concept" button
3. Proposal form opens (see `02_capabilities/manage-concept-proposals.md` for form spec)
4. User fills in:
   - Concept Class, Datatype (from CIEL's configured dropdowns)
   - At least one name (Fully Specified in their locale)
   - Optional: additional names, descriptions, supporting mappings (e.g., SAME-AS SNOMED)
   - Optional: reason/justification
5. User submits → proposal status: **Submitted**
6. User receives confirmation + link to their proposal
7. Notification sent to CIEL admins

### Scenario: An existing concept needs correction

1. User finds a concept in CIEL with an incorrect name, wrong concept class, or missing mapping
2. Concept detail → action menu → "Propose Edit"
3. Form shows current values; user edits only the fields to change
4. User provides a reason (required for edits)
5. Submit → same flow as above

### Scenario: Proposing from OCL Mapper

- User is in OCL Mapper and cannot find a matching CIEL concept for their mapping row
- Mapper UI allows submitting a "Propose to CIEL" action
- Proposal appears in CIEL's proposal queue in TBv3 with `origin = "Mapper"`
- Proposer can track it from either Mapper or TBv3

---

## Part 2: Managing the Queue (Source Administrator)

### Entry Point
- Source → "Concept Proposals" tab (badge showing pending count)
- Notification: "N new concept proposals for [Source]"

### Review Flow

1. Open proposal queue; filter by status = Submitted or In Review
2. Click a proposal to open detail view
3. For **New Concept**:
   - Review proposed concept in full
   - Check for potential duplicates (search box shown within the review panel)
   - If clarification needed: add a comment; status stays In Review
   - If ready: Approve or Reject
4. For **Edit Concept**:
   - Side-by-side diff: current vs. proposed
   - Admin may modify the proposed values before approving
   - Approve or Reject
5. For **Retire Concept**:
   - See impact summary (N collections reference this)
   - Approve (concept retired in HEAD) or Reject

### On Approval
- Concept is created or updated in source HEAD automatically
- Proposer notified: "Your concept proposal was approved"
- Proposal status: Approved; link to new/updated concept shown

### On Rejection
- Admin must enter a rejection reason (appears as a comment)
- Proposer notified: "Your concept proposal was rejected. Reason: [text]"
- Proposal status: Rejected; proposer can view the reason

---

## Part 3: Cross-Origin Management

- Proposals from OCL Mapper are shown with a "Mapper" origin badge
- Admins can respond to and manage them identically to TBv3 proposals
- The comment thread is visible in both TBv3 and (eventually) Mapper
- Approved Mapper proposals: the concept becomes available in CIEL for Mapper users immediately (on next CIEL HEAD refresh)

---

## Post-Completion State

- **Approved:** Concept exists in source HEAD; will be included in the next source release
- **Rejected:** Proposal closed; proposer informed; no changes to source
- **Withdrawn:** No changes; proposal closed by proposer

---

## Error and Edge Cases

| Situation | Behavior |
|---|---|
| Proposed concept ID already exists | Admin prompted to assign a different ID during approval |
| Proposer has been banned or lost access | Proposal remains in queue; admin handles it |
| Duplicate proposals for same concept | System detects potential duplicates based on name similarity; warns admin; admin can merge or reject one |
| Large volume of proposals | Queue supports sorting, filtering, and bulk status updates (mark multiple as "In Review") |
