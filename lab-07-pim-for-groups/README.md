Lab 07 — PIM for Groups (Just-in-Time Group Membership)
Objective

Extend Privileged Identity Management beyond Entra ID roles to group membership. Configure a security group so that users are never permanently members — instead, they hold an eligible assignment and activate it on demand for a limited time window, with MFA verification, justification, and manager approval required.

Zero Trust principle applied: Least privilege access + Assume breach — privileged group membership is never permanent. Access is time-bound, justified, approved, and fully auditable. When the activation window closes, the user is automatically removed from the group.

Environment
Component	Value
Tenant	DidacIAMLab.onmicrosoft.com
License	Microsoft Entra ID P2 (trial)
Admin account	DidacSanchez@DidacIAMLab.onmicrosoft.com
Test user	testpim@DidacIAMLab.onmicrosoft.com
Portal	entra.microsoft.com
What Was Built
Component	Name	Purpose
Privileged group	SG-Privileged-App-Admins	Role-assignable security group managed by PIM
Eligible assignment	Member (testpim)	Just-in-time membership — not permanently active
PIM role settings	Member settings	Controls activation duration, MFA, justification, and approval
Approver	DidacSanchez	Reviews and approves activation requests
Steps
1. Create role-assignable security group

Created SG-Privileged-App-Admins with Microsoft Entra roles can be assigned to the group: Yes.

This setting is mandatory — without it, PIM cannot manage the group's membership. It cannot be changed after group creation.

Show Image

2. Onboard group into PIM

Navigated to Identity Governance → Privileged Identity Management → Groups and selected SG-Privileged-App-Admins.

The group was automatically onboarded into PIM — the Eligible/Active/Expired assignments tabs became available immediately. The audit log registered an "Onboarded resource to PIM" event at this point.

Show Image

3. Create eligible assignment

Via Assignments → + Add assignments:

Setting	Value
Select role	Member
Select members	Test PIM User
Assignment type	Eligible
Duration	1 year (8/13/2026 → 8/13/2027)

Eligible means the user can request activation — they are not a group member until they do so and get approved.

Show Image

4. Configure PIM role settings

Edited the Member role settings to enforce governance controls on every activation:

Setting	Value
Activation maximum duration	2 hours
On activation, require	Azure MFA
Require justification on activation	Yes
Require approval to activate	Yes
Approver	DidacSanchez

Show Image

5. Activation request from testpim

Logged in as testpim in a private browser session. Navigated to PIM → My roles → Groups → SG-Privileged-App-Admins → Activate.

Duration: 2 hours
Reason: Need temporary admin access for application maintenance

Request entered pending approval state after MFA verification.

Show Image

6. Admin approval

Logged in as DidacSanchez → PIM → Groups → SG-Privileged-App-Admins → Approve requests.

Reviewed the pending request from Test PIM User and approved with justification: Approved for scheduled maintenance window.

Full request details visible in the approval panel: role, requestor, resource, start/end time, reason.

Show Image

7. Active time-bound membership verified in PIM

Navigated to SG-Privileged-App-Admins → Assignments → Active assignments.

Field	Value
Name	Test PIM User
State	Activated
Membership	Direct
Start time	8/13/2026, 6:21:30 PM
End time	8/13/2026, 8:21:29 PM

Membership will be automatically removed at 8:21 PM — no manual action required.

Show Image

8. Group membership verified

Navigated to Groups → SG-Privileged-App-Admins → Members.

Test PIM User appears as a direct member during the active window — confirming that PIM translates the approved activation into real group membership automatically.

Show Image

9. Audit trail

Navigated to SG-Privileged-App-Admins → Activity → Audit logs.

Full event chain recorded by PIM:

Time	Service	Activity	Status
6:15:10 PM	PIM	Onboarded resource to PIM	Success
6:15:11 PM	PIM	Add eligible member to role	Success
6:15:11 PM	Core Directory	Add eligible member to role	Success
6:16:47 PM	PIM	Update role setting in PIM	Success
6:19:55 PM	PIM	Add member to role (activation)	Success

Status reason on activation: Need temporary admin access for application maintenance — the justification is stored in the audit log, creating a complete paper trail.

Show Image

Design Decisions

Why a role-assignable group instead of a regular security group? PIM can only manage groups created with "Microsoft Entra roles can be assigned to the group" enabled. This is a hard requirement — it cannot be added retroactively. In production, role-assignable groups are used to grant access to sensitive resources or Entra roles indirectly, which is why PIM governance on them is critical.

Why require approval in addition to MFA? MFA proves the requestor is who they say they are. Approval proves a second person has reviewed and authorised the business reason. Both controls together address two different threat vectors: credential compromise (MFA) and insider abuse or mistakes (approval). In a real environment, the approver would be a team lead or security officer, not the same admin.

Why 2 hours instead of longer? The activation window should match the minimum time needed for the task. A 2-hour window limits the blast radius if the account is compromised during an active session — an attacker who gains access to testpim's session has at most 2 hours of group membership before it expires automatically.

Why PIM for groups instead of direct role assignment? Direct permanent group membership is a standing privilege — it exists 24/7 even when not needed. PIM for groups converts standing privileges into just-in-time access, which is the foundational control for least privilege at scale. This pattern is used in production to manage access to privileged app roles, SharePoint sites, and even Entra ID roles via group assignment.

Lessons Learned

"The resource is invalid" error is cosmetic. When first navigating to a PIM-managed group, an error banner appeared with a resource ID. This did not block any functionality — assignments, settings, and approvals all worked normally. This is a known display bug in the Entra admin center for PIM groups and does not indicate a configuration problem.

Role-assignable groups lock the Membership type to Assigned. When creating the group with "Entra roles can be assigned" set to Yes, the Membership type is automatically forced to Assigned and cannot be changed to Dynamic. This is by design — dynamic membership would conflict with PIM's control over who is in the group.

Audit logs capture both PIM and Core Directory events. The same eligible member addition was logged twice — once by the PIM service and once by Core Directory. This is expected: PIM orchestrates the operation, and Core Directory records the actual directory change. Both entries together provide a complete picture for forensic investigations.

Key Concepts (SC-300 Relevant)
Concept	Description
PIM for Groups	Extends PIM beyond Entra roles to manage group membership with JIT access controls
Eligible assignment	User can request activation but is not a member until approved
Active assignment	User is currently a group member; auto-removed when the window expires
Role-assignable group	Security group that can hold Entra role assignments; required for PIM management
Just-in-time (JIT) access	Access granted only when needed, for the minimum required time
Standing privilege	Permanent access that exists regardless of need — what PIM eliminates
Activation window	Maximum duration a user can be active in the group per request
Audit trail	Complete log of all PIM events: onboarding, assignment, settings changes, activations
