# MS Entra ID: Zero Trust

TryHackMe room covering **Zero Trust principles, Microsoft Entra MFA, least-privilege administration, Security Defaults, and Conditional Access**.

I also recreated the MFA exercise in my own Microsoft Entra tenant. This produced an additional real-world finding when the administrative role specified in the training did not provide the required permissions in the current Entra environment.

---

## Zero Trust Fundamentals

Zero Trust removes implicit trust and evaluates access based on identity, context, risk, and the resource being accessed.

The three core principles are:

```text
Verify explicitly
        +
Use least privilege
        +
Assume breach
```

Zero Trust applies across the wider environment:

```text
Identity → Endpoints → Data → Applications → Infrastructure → Network
```

For identity, this means that successful authentication alone should not automatically result in unrestricted access.

---

## Zero Trust Maturity

The room introduced the progression from traditional security toward a more mature Zero Trust architecture.

```text
Traditional
Passwords / permanent access / limited visibility
        ↓
Initial
MFA / basic access controls / access reviews
        ↓
Advanced
Conditional Access / phishing-resistant MFA / adaptive access
        ↓
Optimal
Passwordless / continuous validation / JIT & JEA
```

The main takeaway was that **Zero Trust is not a single product or configuration**. It is an architecture that becomes stronger as identity, authentication, governance, device security, monitoring, and access controls mature.

---

# Hands-on MFA Lab

## Scenario

The practical section focused on enabling **per-user MFA** for test identities while applying least privilege to the administrative account.

I used my existing restricted account:

```text
THM-Lab-UserAdmin
```

TryHackMe instructed the lab administrator to assign:

**Authentication Administrator**

I assigned the role and continued the exercise using the restricted account.

---

## Finding: Training Role Did Not Provide the Required Access

After assigning **Authentication Administrator**, I attempted to access the MFA administration area required by the room.

Microsoft Entra returned:

```text
401
Insufficient privileges to complete the operation
```

This differed from the behaviour expected by the TryHackMe exercise.

Instead of escalating the account directly to **Global Administrator**, I reviewed the current Microsoft Entra role permissions.

The key distinction was that managing authentication methods and managing authentication policy are separate administrative responsibilities.

```text
Authentication Administrator
        ↓
Authentication-method management
        ↓
Did NOT provide required MFA administration access
```

The additional role required in my current tenant was:

**Authentication Policy Administrator**

After assigning this role, the required MFA administration page became accessible and the exercise worked as expected.

```text
TryHackMe instruction
Authentication Administrator
        ↓
Role assigned
        ↓
401 / Insufficient privileges
        ↓
Investigated Entra RBAC
        ↓
Authentication Policy Administrator
        ↓
Required access available
```

This became one of the most valuable findings from the room because it demonstrated the importance of checking **actual current RBAC permissions** rather than assuming that a role name provides every related capability.

---

# Per-User MFA

With the correct permissions available, I reviewed the **per-user multifactor authentication** configuration.

The available settings included controls for:

- MFA verification
- App passwords
- Trusted IPs
- Remembered MFA sessions

I then enabled MFA for selected test identities.

The user state changed from:

```text
Disabled
   ↓
Enabled
```

The room covered three per-user MFA states:

```text
Disabled
→ Per-user MFA is not enabled

Enabled
→ MFA is enabled but user registration may still be required

Enforced
→ MFA registration has been completed and MFA is enforced
```

---

## Testing the User Experience

I signed in using one of the MFA-enabled test accounts.

Microsoft displayed:

> **Action Required**

The user was required to provide additional security information and register Microsoft Authenticator.

This demonstrated an important distinction:

```text
Administrator enables MFA
        ↓
User signs in
        ↓
Additional security information required
        ↓
User registers authentication method
        ↓
MFA authentication becomes available
```

Therefore:

```text
MFA Enabled
     ≠
MFA Registration Completed
```

Enabling MFA administratively and completing MFA registration are separate stages.

---

# Security Defaults

My Microsoft Entra tenant already had **Security Defaults enabled**.

Security Defaults provide a basic identity-security baseline with protections around areas such as:

- MFA registration
- Privileged authentication
- Additional authentication challenges
- Legacy authentication

This provides a relatively simple baseline, while **Conditional Access** provides more granular and adaptive access control.

---

# Conditional Access

The room introduced **Microsoft Entra Conditional Access** as an important Zero Trust policy engine.

Instead of making an access decision based only on a username and password, Conditional Access can evaluate multiple signals.

```text
User / Group
Location
Device
Application
Risk
        ↓
Conditional Access
        ↓
Access Decision
        ↓
Allow
Require additional controls
Block
```

Examples of possible requirements include:

- Require MFA
- Require stronger authentication
- Require a compliant device
- Require an Entra hybrid joined device
- Require an approved client application
- Require an app protection policy
- Require a password change

This allows access decisions to become **context-aware and adaptive**.

For example:

```text
Normal sign-in
→ Allow

Unusual or higher-risk sign-in
→ Require stronger verification

Unacceptable conditions
→ Block
```

---

## Conditional Access Administrator

The room also introduced the **Conditional Access Administrator** role.

This allows Conditional Access management without automatically granting the much broader Global Administrator role.

It follows the same least-privilege approach used throughout the lab:

```text
Identify administrative task
        ↓
Determine required permissions
        ↓
Assign the narrowest suitable role
```

However, Conditional Access Administrator is still a highly sensitive role.

An administrator who can change Conditional Access may potentially weaken important controls such as MFA, device requirements, or access restrictions.

Administrative IAM roles are therefore part of the organization's security attack surface.

---

## Conditional Access Scope in This Room

The Conditional Access section was primarily **conceptual/read-through**.

I did **not** create a Conditional Access policy as part of this specific Zero Trust room.

Hands-on Conditional Access configuration is covered later in the:

**MS Entra ID: Authentication**

room.

I keep this distinction in the repository so the project accurately separates what I configured myself from what I studied conceptually.

---

# Least-Privilege Cleanup

After completing the MFA exercise, I removed the temporary administrative role assignments.

The workflow was:

```text
Restricted account
        ↓
Assign required privilege
        ↓
Test access
        ↓
Discover permission limitation
        ↓
Investigate RBAC
        ↓
Assign correct role
        ↓
Perform MFA task
        ↓
Verify user behaviour
        ↓
Remove temporary privileges
```

This directly applied the Zero Trust principle of:

**Use least privilege**

rather than leaving unnecessary administrative permissions permanently assigned.

---

# Key Findings & Takeaways

### Zero Trust is broader than MFA

MFA is only one control.

A Zero Trust access decision can combine:

```text
Identity
+
Authentication
+
Device
+
Location
+
Application
+
Risk
+
Access Policy
```

---

### Entra administrative roles are task-specific

A role with **Authentication** in its name does not automatically provide access to every authentication-related setting.

Actual role permissions must be checked against the task being performed.

---

### Training material may differ from the current platform

TryHackMe instructed the use of **Authentication Administrator**, but this did not provide the required MFA administration access in my current Entra tenant.

Investigating the current permission model showed that **Authentication Policy Administrator** was also required for the exercise.

This turned the room into a useful real-world RBAC troubleshooting exercise.

---

### Authentication-method management and authentication-policy management are different

Microsoft Entra separates these administrative responsibilities.

```text
Manage user authentication methods
                ≠
Manage authentication policy / MFA configuration
```

This separation helps support least privilege and separation of duties.

---

### MFA enablement and MFA registration are separate

An administrator can enable MFA, but the user may still need to register an authentication method.

```text
MFA enabled
        ↓
Registration required
        ↓
Authentication method configured
```

---

### MFA user states matter

The distinction between:

```text
Disabled
Enabled
Enforced
```

is useful when troubleshooting per-user MFA behaviour.

---

### Conditional Access enables adaptive access

Conditional Access can use multiple signals rather than applying the exact same authentication requirement to every request.

This is a major part of implementing Zero Trust identity security.

---

### Privileged IAM roles are themselves high-value targets

Roles capable of modifying:

- MFA
- authentication policy
- Conditional Access
- identity settings

can potentially weaken security controls if compromised.

These accounts and roles therefore require strong protection and monitoring.

---

### Least privilege applies to administrators too

Administrative permissions should only exist when required.

```text
Assign
  ↓
Use
  ↓
Verify
  ↓
Remove
```

Temporary elevated access should not become permanent access.

---

### Zero Trust maturity is gradual

Organizations normally progress from:

```text
Passwords + implicit trust
        ↓
MFA + basic controls
        ↓
Conditional Access + contextual signals
        ↓
Phishing-resistant authentication
        ↓
Passwordless + continuous validation
```

Zero Trust is therefore an ongoing security strategy rather than a single configuration change.

---

# Skills Practiced

- Microsoft Entra ID
- Zero Trust principles
- Zero Trust maturity
- Microsoft Entra RBAC
- Least-privilege administration
- Authentication Administrator
- Authentication Policy Administrator
- Per-user MFA
- MFA administration
- MFA registration
- MFA user states
- Security Defaults
- Conditional Access concepts
- Conditional Access Administrator
- Adaptive access
- RBAC troubleshooting
- Permission-boundary testing
- User authentication testing
- Privileged-access cleanup

---

# References

- [NIST – Executive Order 14028: Improving the Nation's Cybersecurity](https://www.nist.gov/itl/executive-order-14028-improving-nations-cybersecurity)
- [OMB M-22-09 – Moving the U.S. Government Toward Zero Trust Cybersecurity Principles](https://www.whitehouse.gov/wp-content/uploads/2022/01/M-22-09.pdf)
- [CISA – Zero Trust Maturity Model](https://www.cisa.gov/resources-tools/resources/zero-trust-maturity-model)
