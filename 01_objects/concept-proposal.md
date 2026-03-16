# Object: Concept Proposal 

To Do: 
* Ensure that source admin can edit concept proposals before approving
* Concept proposals should not be enabled by default - only if source admin enables it.

A Concept Proposal is a formal request to add or modify a concept in a source that the proposer does not own. It is a first-class resource tracked with status, comments, and audit trail. Proposals from TBv3 and from OCL Mapper are stored in the same database and managed by source administrators in TBv3.

"?" represents optional (non-required) attributes

## Schema

```
concept_proposal:
  id: string                    # System-assigned
  status: string                # "Draft" | "Submitted" | "In Review" | "Approved" | "Rejected" | "Withdrawn"
  proposal_type: string         # "New Concept" | "Edit Concept" | "Retire Concept"

  # Target
  target_source_url: string     # Source the proposal is for
  target_concept_url: string?   # Set for Edit/Retire proposals; null for New Concept

  # Proposed content
  proposed_concept:             # If concept templates are supported and enabled, proposer may select one to start with
    concept_class: string
    datatype: string
    names: Name[]
    descriptions: Description[]
    mappings: Mapping[]?
    extras: Record<string, any>?   

  reason: string?               # Proposer's justification

  # Discussion
  comments: Comment[]

  # System-managed
  submitted_by: string          # User who created the proposal
  submitted_on: datetime
  updated_on: datetime
  reviewed_by: string?          # Admin who took action
  reviewed_on: datetime?
  source: string                # Source mnemonic
  owner: string                 # Source owner
  origin: string                # "TBv3" | "Mapper" — which app it was submitted from
```

### Comment Sub-Object
```
comment:
  id: string
  text: string
  author: string
  created_on: datetime
  updated_on: datetime?
```

---

## Lifecycle / States

```
Draft  →  Submitted  →  In Review  →  Approved  →  [concept created/updated in source]
                                    └─ Rejected
                                    
Submitted / In Review  →  Withdrawn  (by proposer)
```

- Only the proposer can withdraw a proposal
- Only source admins can move a proposal to In Review, Approved, or Rejected
- An Approved proposal automatically creates or updates the concept in the target source (async)

---

## Business Rules

### Who Can Submit
- Any authenticated user can submit a proposal to any source they have read access to
- The proposer does not need edit access to the source

### New Concept Proposals
- The proposer specifies concept class, datatype, and at least one name
- The concept ID may be left blank; the source admin assigns it during review
- The proposer may include mappings (e.g., SAME-AS SNOMED) as supporting evidence — these are informational until approved

### Edit Concept Proposals
- The proposer specifies only the fields they want to change (diff/patch model)
- The existing concept is shown alongside the proposed changes in the review view

### Retire Concept Proposals
- The proposer provides a reason
- Admin can preview the impact before approving (e.g., how many collections reference this concept)

### Comments
- Both proposers and source admins can add comments at any status
- Comments are ordered oldest-first (chronological thread)
- Comment authors are shown with their OCL username and timestamp

### Origin Tracking
- The `origin` field distinguishes TBv3 proposals from Mapper proposals
- Both types appear in the same queue in TBv3
- Source admins manage all proposals regardless of origin

### Notifications
- Proposer is notified when their proposal status changes
- Source admin is notified when a new proposal is submitted

---

## API Patterns

| Operation | Endpoint |
|---|---|
| Submit proposal | `POST /:owner/sources/:source/concept-proposals/` |
| List proposals (admin) | `GET /:owner/sources/:source/concept-proposals/` |
| Get proposal | `GET /:owner/sources/:source/concept-proposals/:id/` |
| Update status | `PUT /:owner/sources/:source/concept-proposals/:id/` |
| Add comment | `POST /:owner/sources/:source/concept-proposals/:id/comments/` |
| List my proposals | `GET /user/concept-proposals/` |

---

## UI Display Rules

**For proposers (TI/TP submitting a proposal):**
- Accessed from a concept detail page ("Propose Edit") or from a source's "Propose a Concept" CTA
- A form showing the proposed concept fields alongside the existing values (if editing)
- Status tracker shown once submitted (Draft → Submitted → In Review → Approved/Rejected)
- Comment thread visible throughout

**For proposers (TI/TP reviewing their outstanding proposals) - To Do**
-

**For source administrators (TP managing their queue):**
- Proposals tab in the Source repository view (badge showing pending count)
- List view: proposal ID, type, submitter, submitted date, status, origin (TBv3 / Mapper)
- Detail view: side-by-side of current concept vs. proposed changes (for edits), full proposed concept (for new), comment thread
- Action buttons: "Mark In Review", "Approve", "Reject", "Add Comment"
- Filtering: by status, proposal type, submitter, origin, date range, concept class, datatype
