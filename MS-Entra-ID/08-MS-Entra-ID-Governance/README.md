# Microsoft Entra ID Governance — Privileged Identity Management (PIM)

This project documents my study of **Microsoft Entra Privileged Identity Management (PIM)** through the TryHackMe Microsoft Entra ID Governance lab.

The room focused on protecting privileged access through **Just-in-Time (JIT) administration, eligible role assignments, time-bound access, role activation, MFA, approval workflows, justification, ticket information, notifications, access reviews, and auditing**.

The main security principle behind PIM is simple:

> Privileged access should exist only when it is needed, for only as long as it is needed.

---

## Lab Scenario and Goal

The lab simulated an enterprise environment where administrators and technical users occasionally require elevated Microsoft Entra permissions.

Instead of assigning powerful directory roles permanently, the organization uses **Privileged Identity Management** to control:

- who is allowed to request privileged access,
- which role they may request,
- when they may activate it,
- how long the privilege lasts,
- which security controls must be satisfied,
- whether another administrator must approve the request,
- and how the privileged activity is audited.

The room covered the complete privileged-access lifecycle:

**Assign → Activate → Approve → Audit**

The objective was to understand how Microsoft Entra PIM reduces standing privilege and supports **Least Privilege, Zero Trust, separation of duties, and privileged-access governance**.

---

# Why Privileged Identity Management Matters

Permanent administrative privileges create unnecessary risk.

If an attacker compromises an account that permanently holds a powerful role such as:

- Global Administrator
- Privileged Role Administrator
- Application Administrator
- User Administrator

the attacker immediately inherits those permissions.

PIM changes this model.

Instead of:

    User account
        ↓
    Permanent Administrator
        ↓
    Privilege always available

PIM allows:

    Normal user account
        ↓
    Eligible for privileged role
        ↓
    Activation requested
        ↓
    MFA / Reason / Ticket / Approval
        ↓
    Temporary privileged access
        ↓
    Administrative task performed
        ↓
    Role deactivated or expires

This reduces the amount of time privileged permissions are available for abuse.

---

# Core PIM Concepts

## Eligible Assignment

An **eligible assignment** means the user is allowed to activate a role when required.

The user does not continuously possess the role.

Example:

    User: HR Administrator
    Eligible role: User Administrator

The HR administrator normally remains a standard user.

When privileged work is required, the role must first be activated.

---

## Active Assignment

An **active assignment** means the user currently possesses the privileged role.

Unlike an eligible assignment, no additional activation is required before the privileged permissions can be used.

From a Least Privilege perspective, eligible assignments are generally preferable where operationally possible.

---

# Permanent vs Time-Bound Assignments

PIM can also control how long role assignments exist.

| Assignment Type | Description |
|---|---|
| Permanent Eligible | User can activate the role indefinitely |
| Time-Bound Eligible | Eligibility expires on a defined date |
| Permanent Active | User continuously possesses the role |
| Time-Bound Active | Active assignment expires on a defined date |

A mature privileged-access model should minimize permanent assignments.

For example:

    Eligibility duration: 6 months
    Maximum activation duration: 4 hours

This means the employee may be authorized to perform the function for six months, but each individual privileged session remains short.

---
# The Four A's of PIM

The room introduced four important PIM operations:

**Assign → Activate → Approve → Audit**

![Four A's of PIM](./Four%20A%27s%20of%20PIM.png)

This model represents the core privileged-access lifecycle in Microsoft Entra PIM:

- **Assign** — make a user eligible for a privileged role.
- **Activate** — temporarily elevate the eligible role when required.
- **Approve** — review and authorize sensitive activation requests.
- **Audit** — maintain visibility into privileged assignments and activations.

---

---

# PIM Assignment Scenario

One scenario in the room involved assigning privileged role eligibility to a user.

The administrator navigates through:

    Identity Governance
        ↓
    Privileged Identity Management
        ↓
    Microsoft Entra roles
        ↓
    Roles / Assignments

Instead of directly assigning standing administrative privilege, the user is configured as:

    Assignment type: Eligible

This creates an important IAM distinction:

> A user can be authorized to request privilege without permanently possessing that privilege.

---

# Just-in-Time Privileged Access

After receiving an eligible assignment, the user can activate the role when privileged work is actually required.

The user navigates to:

    Identity Governance
        ↓
    Privileged Identity Management
        ↓
    My roles
        ↓
    Microsoft Entra roles

The role appears under **Eligible assignments**.

The user can then request activation.

This implements **Just-in-Time privileged access**.

Instead of administrative privilege existing continuously, privilege exists only during the required administrative task.

---

# Activation Duration

PIM allows administrators to configure the maximum amount of time a role may remain activated.

For example:

    Eligible for role: 6 months

    Individual activation:
    Maximum duration: 4 hours

This substantially reduces exposure.

Even if an administrator forgets to manually remove the privilege, the activation automatically expires.

A stronger operational practice is to manually deactivate the role immediately after completing the required task.

---

# Business Justification

PIM can require users to explain why privileged access is required.

Example:

    Reason:
    Need to perform user onboarding tasks for new employees.

This adds business context to the technical privilege elevation.

Instead of an audit record showing only:

    User activated User Administrator

the organization can understand:

- who requested the access,
- which privilege was requested,
- when it was requested,
- and why it was required.

This is valuable for:

- IAM governance
- incident investigations
- compliance
- internal auditing
- privileged-access reviews

---

# Service Ticket Information

PIM can also require service-ticket information during privileged-role activation.

An activation request may require:

    Ticket system
    Ticket number
    Business justification

Example:

    Ticket system: ServiceNow
    Ticket number: IAM-1042
    Reason: Register approved enterprise application

This helps link privileged operations with an organization's formal change-management or service-management process.

An important security detail is that PIM records this information for governance purposes, but the ticket information itself is not automatically validated against the external ticketing platform.

---

# MFA During Privileged Role Activation

PIM can require **Multifactor Authentication** before privileged elevation.

This creates another security boundary between normal account access and administrative access.

For example:

    Attacker steals password
            ↓
    Signs in successfully
            ↓
    Attempts privileged role activation
            ↓
    MFA required
            ↓
    Attacker cannot satisfy MFA
            ↓
    Privileged access denied

This supports an important Zero Trust principle:

> Authentication to the account does not automatically authorize privileged access.

Administrative elevation should require stronger assurance than normal account access.

---

# Approval-Based Privilege Elevation

Sensitive roles can require administrator approval before activation.

The workflow becomes:

    Eligible user
         ↓
    Requests privileged role
         ↓
    MFA
         ↓
    Business justification
         ↓
    Ticket information
         ↓
    Approval request
         ↓
      Approver
       /    \
      /      \
    Approve   Deny
      |
      ↓
    Temporary privileged role

This introduces **separation of duties** and reduces the possibility of uncontrolled privilege escalation.

---

# Role-Specific Security Policies

An important feature of PIM is that different privileged roles can have different activation requirements.

For example:

| Control | User Administrator | Application Administrator |
|---|---:|---:|
| MFA | Optional depending on policy | Required |
| Justification | Required | Required |
| Ticket information | Optional | Required |
| Approval | Optional | Required |
| Maximum activation duration | Limited | Limited |

This allows organizations to apply stronger security controls to higher-risk privileged roles.

---

# Application Developer Security Scenario

The TryHackMe lab included a scenario where the **Application Developer** privileged role had to meet stricter security requirements.

The policy required:

- Maximum activation duration of **4 hours**
- MFA during activation
- Business justification
- Service-ticket information
- Administrator approval
- No permanent eligible assignment
- No permanent active assignment
- Assignments expiring after a defined period
- Notifications for administrators, assignees, requestors, and approvers

This demonstrates how PIM can convert privileged access from a permanent entitlement into a controlled security workflow.

---

# Restricting Permanent Privileged Assignments

PIM allows organizations to prevent privileged assignments from remaining indefinitely.

Example policy:

    Allow permanent eligible assignment: No

    Eligible assignments expire after:
    6 months

and:

    Allow permanent active assignment: No

    Active assignments expire after:
    6 months

This prevents temporary operational requirements from becoming permanent privileged access.

It also forces organizations to periodically reconsider whether users still require the privileged role.

---

# Approval Workflow

When approval is required, privileged activation requests can be reviewed by designated approvers.

The administrator navigates to:

    Identity Governance
        ↓
    Privileged Identity Management
        ↓
    Approve requests

The approver can review details including:

- Requested role
- Requesting user
- Request time
- Start time
- End time
- Business justification
- Ticket system
- Ticket number

The request can then be approved or denied.

This creates a formal checkpoint before sensitive privilege is granted.

---

# PIM Notifications

PIM supports notifications for privileged-access events.

Recipients can include:

- Administrators
- Assignees
- Requestors
- Approvers

Notifications may be generated when:

- a user receives an eligible assignment,
- a user receives an active assignment,
- an eligible user activates a role,
- an approval request is created,
- or privileged role activity occurs.

These notifications provide visibility into administrative privilege changes.

---

# Role Deactivation

A user does not need to remain privileged until the configured activation period expires.

Once the administrative task is completed, the role can be manually deactivated.

Preferred workflow:

    Activate role
        ↓
    Perform privileged task
        ↓
    Deactivate role immediately

Less desirable workflow:

    Activate role
        ↓
    Perform privileged task
        ↓
    Leave privilege active
        ↓
    Wait for automatic expiration

The first approach reduces unnecessary privileged exposure.

---

# Access Reviews

Privileged access should not only be controlled when it is activated.

Organizations must also periodically verify whether users still require the ability to activate those roles.

Microsoft Entra Access Reviews can support this governance process.

Example lifecycle:

    Privilege assigned
        ↓
    Privilege activated
        ↓
    Privilege used
        ↓
    Activity audited
        ↓
    Access review performed
        ↓
    Is access still required?
       /            \
     Yes             No
      |               |
    Retain          Remove

This helps prevent privilege accumulation over time.

---

# PIM Audit History

PIM provides audit history for privileged role operations.

The lab reviewed events including:

- Eligible assignment creation
- Active assignment creation
- Role activation
- Role approval
- Permanent assignments
- Time-bound assignments
- Role-setting changes

Audit history helps answer questions such as:

- Who received the privilege?
- Who requested the activation?
- Who approved it?
- Which role was involved?
- When was the role activated?
- How long did the role remain active?
- Why was the access required?
- Were any PIM security settings modified?

This provides valuable evidence for investigations and compliance.

---

# Monitoring Changes to PIM Security Settings

One of the most important security lessons from the room was that **PIM configuration itself is a high-value security target**.

An attacker who gains sufficient administrative permissions may attempt to weaken PIM before performing additional malicious actions.

Examples include:

    Disable MFA requirement

    Disable approval requirement

    Increase maximum activation duration

    Enable permanent privileged assignments

    Disable justification requirement

Therefore, IAM and security teams should monitor events such as:

    Update role setting in PIM

Changes to privileged-access controls should be treated as sensitive administrative events.

---

# Privileged Identity Attack Surface

PIM reduces the Identity Attack Surface by minimizing standing privilege.

Without PIM:

    Compromised admin
        ↓
    Immediate privileged access
        ↓
    Directory modification
        ↓
    Persistence
        ↓
    Lateral movement

With PIM:

    Compromised normal account
        ↓
    Eligible privilege exists
        ↓
    Activation still required
        ↓
    MFA / Approval / Justification
        ↓
    Attacker may be blocked

PIM therefore introduces several control points between account compromise and administrative privilege.

---

# Security Findings

## 1. Eligibility is safer than permanent privilege

A compromised account with an eligible role does not automatically possess the privileged permissions.

The attacker still needs to complete the activation requirements.

---

## 2. MFA should protect privilege elevation

Administrative privilege represents a higher security boundary than ordinary sign-in.

Requiring MFA during activation provides another barrier even if the password has already been compromised.

---

## 3. Approval creates separation of duties

Sensitive administrative roles should not always be self-approved.

Delegated approval reduces uncontrolled privilege escalation.

---

## 4. Privileged access should be short-lived

Users may need elevated permissions for only a small part of their working day.

Privilege should therefore remain active for hours rather than permanently.

---

## 5. Eligibility itself should expire

Even the ability to request privilege should not automatically remain forever.

Time-bound eligibility forces organizations to periodically reconsider whether privileged access is still required.

---

## 6. Business justification improves governance

A technical role activation becomes much easier to investigate when the organization knows why the privilege was requested.

---

## 7. Ticket information connects IAM with change management

Service-ticket details help connect privileged access with approved operational work.

---

## 8. PIM settings must also be protected

An attacker may attempt to weaken PIM controls before escalating privileges.

Changes to:

- MFA requirements
- approval requirements
- activation duration
- permanent assignment settings
- justification requirements

should therefore be monitored.

---

# PIM Security Model

The complete privileged-access model covered in the room can be summarized as:

    ┌─────────────────────────────────────────┐
    │      Microsoft Entra PIM                │
    └─────────────────────────────────────────┘
                       |
                       v
                    ASSIGN
                       |
                Eligible Role
                       |
                       v
                   ACTIVATE
                       |
        ┌──────────────┼──────────────┐
        |              |              |
       MFA        Justification      Ticket
        |              |              |
        └──────────────┼──────────────┘
                       |
                       v
                    APPROVE
                       |
                       v
             Temporary Privilege
                       |
                       v
              Administrative Task
                       |
                       v
            Deactivate / Auto-expire
                       |
                       v
                     AUDIT
                       |
                       v
                Access Review

---

# Zero Trust Connection

PIM directly supports several Zero Trust principles.

## Verify Explicitly

Privileged access can require:

- MFA
- approval
- justification
- ticket information

before elevation is allowed.

## Use Least Privilege

Users receive only the role they require and only when they require it.

## Assume Breach

Even if a standard user account is compromised, privileged access still requires additional controls.

This reduces the blast radius of account compromise.

---

# Traditional Privileged Access vs PIM

| Traditional Model | PIM Model |
|---|---|
| Permanent administrator | Eligible administrator |
| Privilege always available | Privilege activated when required |
| Long exposure window | Short activation window |
| Limited business context | Justification and ticket information |
| User may self-elevate | Approval can be required |
| Normal login may be enough | MFA can be required again |
| Privileges accumulate | Access can expire and be reviewed |
| Limited visibility | Detailed audit history |

---

# Key Takeaways

Microsoft Entra Privileged Identity Management changes privileged administration from:

    Administrator account
        =
    Permanent administrator

into:

    Normal account
        +
    Eligible privilege
        +
    Business requirement
        +
    Strong authentication
        +
    Approval
        +
    Limited duration
        +
    Audit evidence

PIM is therefore not simply about deciding **who can become an administrator**.

It controls:

    WHO

    can obtain

    WHICH PRIVILEGE

    for WHAT REASON

    after WHICH SECURITY CONTROLS

    for HOW LONG

    with WHOSE APPROVAL

    and with WHAT AUDIT EVIDENCE

This is the foundation of modern **Privileged Access Management and Identity Governance**.

---

# Skills Developed

This TryHackMe room strengthened my understanding of:

- Microsoft Entra Privileged Identity Management
- Microsoft Entra Identity Governance
- Privileged Access Management
- Privileged Role Management
- Just-in-Time access
- Just-Enough-Access concepts
- Microsoft Entra administrative roles
- Eligible role assignments
- Active role assignments
- Permanent vs time-bound assignments
- Role activation
- Role deactivation
- Activation duration controls
- Least Privilege
- Zero Trust
- Multifactor Authentication
- Approval workflows
- Separation of Duties
- Business justification
- Service-ticket information
- Privileged access notifications
- Access Reviews
- PIM audit history
- Privileged-role policy configuration
- Identity Attack Surface reduction
- Privilege escalation prevention
- Privileged access monitoring
- IAM governance
- Administrative access lifecycle management

