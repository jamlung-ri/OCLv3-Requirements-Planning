# Capability: Manage Concept Proposals

## Scope

Submitting concept proposals (for non-source-owners), and reviewing/managing those proposals (for source administrators). Proposals from TBv3 and from OCL Mapper are managed in the same interface. This capability covers both sides of the workflow.

---

## Submitting a Proposal (Terminology Implementer or any authenticated user)

### Entry Points

**New Concept Proposal:**
- Source detail page → "Propose New Concept" button (authenticated users; visible even to non-editors)
- Concept Proposals tab in a source → "New Proposal" button

**Edit Concept Proposal:**
- Concept detail view → action menu → "Propose Edit"

**Retire Concept Proposal:**
- Concept detail view → action menu → "Propose Retirement"

### New Concept Proposal Form

Identical to the concept creation form (`02_capabilities/author-concept.md`) with these differences:
- The form header reads "Propose New Concept to [Source Name]"
- A **Reason** field is added (textarea; optional but strongly encouraged — prompt text: "Describe why this concept is needed and how it will be used")
- A **Mappings** section is available but labeled "Supporting Mappings (informational)" — these are not created automatically; the admin decides during review
- Concept ID may be left blank ("Assign automatically" option) - appears only if source admins enable manual ID generation i.e. no ID auto-assignment
- Submit button: "Submit Proposal" (not "Save Concept")

### Edit Concept Proposal Form

- Shows the existing concept's values alongside the proposed changes (diff model)
- User edits only the fields they want to change; unchanged fields are clearly indicated
- Reason field required for edit proposals
- Submit button: "Submit Proposed Edit"

### Retire Concept Proposal Form

- Shows the concept to be retired with a summary of where it is used (collections, expansions, mappings)
- Reason field required
- Submit button: "Submit Retirement Request"

### Post-Submission State

- Success toast: "Your proposal has been submitted to [Source Name]. You can track its status in Concept Proposals."
- Link to the submitted proposal's detail page
- User can view all their submitted proposals at `/user/concept-proposals/`

---

## Tracking a Submitted Proposal (Proposer View)

### "My Proposals" Page
- Accessible from user profile → "My Concept Proposals" tab, or from the post-submission link
- List view: Proposal ID | Type | Target Source | Status | Submitted Date | Last Updated
- Filter by status, source, date range

### Proposal Detail (Proposer View)
- Status tracker (step indicator): Draft → Submitted → In Review → Approved / Rejected
- Proposed concept details
- Comment thread (proposer can add comments at any time)
- Withdraw button (available until Approved or Rejected)
(To Do: Add action for Approved proposals to be Added to Collection, Cloned, etc.)
---

## Managing Proposals (Source Administrator View) (To Do: Explore idea of public-facing Proposal view for non-admins to see/add to/endorse existing proposals)

### Entry Point
- Source repository → "Concept Proposals" tab
- Badge on the tab showing count of pending proposals (Submitted + In Review)

### Proposals List 

| Column | Notes |
|---|---|
| Proposal ID | |
| Type | New Concept / Edit / Retire |
| Submitter | Username chip |
| Submitted | Relative date |
| Status | Badge: Submitted / In Review / Approved / Rejected / Withdrawn|
| Origin | TBv3 or Mapper badge |

- Default filter: pending proposals only (Submitted + In Review)
- Filters: Status, Type, Origin, Submitter, Date range
- Sort: Submitted date descending (newest first)

### Proposal Detail (Admin Review View)

**For New Concept proposals:**
- Full proposed concept shown in a structured form layout (read-only)
- Supporting mappings shown separately
- Proposer's reason displayed prominently
- Comment thread (admin can add comments)
- Action buttons:
  - **Mark In Review** — signals to proposer that it is being looked at; no other effect
  - **Approve** — opens a confirmation dialog with the option to set the Concept ID (if blank); on confirm, creates the concept in the source HEAD; sends notification to proposer
  - **Reject** — opens a dialog requiring a reason; reason appears as a comment; sends notification to proposer

**For Edit Concept proposals:**
- Side-by-side diff: Current concept (left) | Proposed changes (right)
- Changed fields highlighted
- Admin can modify proposed values before approving (edits are tracked)
- Same action buttons as above

**For Retire Concept proposals:**
- Summary of impact: "This concept is referenced in [N] collections across OCL"
- Option to show a filtered list of affected collections
- Retire action with confirmation; sends notifications to collection owners

---

## Notifications

| Trigger | Recipient | Message |
|---|---|---|
| New proposal submitted | Source admin | "New concept proposal for [Source]: [type] by [user]" |
| Proposal approved | Proposer | "Your concept proposal was approved. [View concept]" |
| Proposal rejected | Proposer | "Your concept proposal was rejected. [View reason]" |
| Comment added | Other participant in thread | "New comment on concept proposal [ID]" |
| Proposal status changed to In Review | Proposer | "[Source admin] is reviewing your proposal" |

---

## Permissions

| Action | Required Permission |
|---|---|
| Submit New Concept proposal | Authenticated user; read access to target source |
| Submit Edit / Retire proposal | Authenticated user; read access to target source |
| View own proposals | Proposer only |
| View all proposals for a source | Source owner or editor |
| Mark In Review / Approve / Reject | Source owner or editor |
| Add comments | Proposer or source admin |
| Withdraw a proposal | Proposer only (before Approved/Rejected) |
