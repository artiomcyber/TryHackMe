# MS Entra ID: Zero Trust

TryHackMe room covering **Zero Trust identity security, Microsoft Entra MFA, least-privilege administration, Security Defaults, and Conditional Access**.

Alongside the room, I recreated the MFA administration exercise in my own Microsoft Entra tenant. The practical work also produced a useful real-world finding: the administrator role specified by the training did not provide the required permissions in my current Entra environment, so I investigated the RBAC model, identified the correct role, completed the MFA configuration, tested the user experience, and removed the temporary privileges afterward.

---

## Zero Trust Fundamentals

Zero Trust replaces implicit trust with continuous verification.

The three core principles are:

```text
Verify explicitly
        +
Use least privilege
        +
Assume breach
```

Zero Trust should be applied across the entire digital environment:

```text
Identity → Endpoints → Data → Applications → Infrastructure → Network
```

![Zero Trust](./Zero%20Trust.png)

For identity, this means that a successful username and password should not automatically result in unrestricted access. Identity, device, application, location, risk, and authentication strength can all contribute to the access decision.

---

## Zero Trust Maturity

The room introduced the progression from traditional identity security toward a mature Zero Trust architecture.

![High-Level Zero Trust Maturity Model Overview](./High-Level%20Zero%20Trust%20Maturity%20Model%20Overview.png)

A simplified identity progression is:

```text
Traditional
Passwords / permanent access / limited visibility
        ↓
Initial
MFA / basic controls / access reviews
        ↓
Advanced
Phishing-resistant MFA / Conditional Access / adaptive access
        ↓
Optimal
Passwordless / continuous validation / automated JIT & JEA
```

The key lesson was that **Zero Trust is not a single Microsoft product or configuration**. It is an architecture that matures across identity, devices, applications, data, networks, monitoring, governance, and automation.

The room also showed what a more mature Zero Trust environment looks like when multiple signals feed into real-time security policy enforcement.

![Optimal Zero Trust implementation](./optimal%20Zero%20Trust%20implementation.png)

---

# Security Defaults

Before working with individual MFA configuration, I reviewed the tenant's baseline protection.

**Security Defaults were already enabled.**

![Security Defaults enabled](./Security%20defaults%20enabled.png)

Security Defaults provide a basic identity-security baseline, including protections around:

- MFA registration
- MFA for privileged activity
- additional authentication challenges when required
- protection against legacy authentication

For environments requiring more granular and context-aware decisions, Microsoft Entra provides **Conditional Access**.

---

# Hands-on MFA Lab

## Scenario

The practical task was to enable **per-user MFA** for test identities while using a restricted administrative account rather than performing everything as Global Administrator.

I reused my existing lab identity:

```text
THM-Lab-UserAdmin
```

The TryHackMe exercise instructed the administrator to use the:

**Authentication Administrator**

role.

I assigned that role to the lab account.

![Authentication Administrator assigned](./AUTH%20ADmin%20assigned.png)

---

## Testing the Authentication Administrator Role

After switching to the restricted account, I confirmed that the account was operating with limited administrative privileges.

![Authentication Administrator profile](./Overview%20of%20auth%20amin%20profile.png)

The role allowed authentication-method administration for non-admin users, but other security information remained inaccessible.

When I attempted to access the administration area required for the MFA exercise, Microsoft Entra returned:

```text
401
Insufficient privileges to complete the operation
```

![Authentication Administrator access denied](./Auth%20admin%20no%20access.png)

This did **not** match the expected behaviour in the TryHackMe instructions.

---

# Finding: Authentication Administrator Was Not Sufficient

Instead of immediately assigning Global Administrator, I checked Microsoft's current Entra role permissions to determine which role actually controlled the required MFA settings.

The role comparison showed an important separation of responsibilities.

![Microsoft authentication administrator role comparison](./MS%20Auth%20admins%20describtions%20of%20roles.png)

The important difference was:

| Role | Manage user authentication methods | Manage per-user MFA | Manage MFA settings |
|---|---:|---:|---:|
| **Authentication Administrator** | Yes, for some users | No | No |
| **Privileged Authentication Administrator** | Yes, for all users | No | No |
| **Authentication Policy Administrator** | No | Yes | Yes |
| **User Administrator** | No | No | No |

This explained the failed access.

The TryHackMe exercise expected **Authentication Administrator**, but Microsoft's current permission model showed that the MFA task required:

**Authentication Policy Administrator**

I assigned the additional role.

![Authentication Policy Administrator assigned](./Auth%20policy%20admin%20assigned.png)

After this change, the required MFA administration interface became accessible and the lab worked correctly.

The troubleshooting path was:

```text
TryHackMe instruction
Authentication Administrator
        ↓
Role assigned
        ↓
Required MFA page denied
        ↓
401 / Insufficient privileges
        ↓
Review Microsoft Entra RBAC permissions
        ↓
Authentication Policy Administrator identified
        ↓
Role assigned
        ↓
MFA administration available
```

This became one of the most valuable findings from the room because it demonstrated the importance of checking **actual role permissions** instead of assuming that a role name provides every related capability.

---

# Reviewing MFA Service Settings

With the correct permissions available, I accessed the **per-user multifactor authentication** administration area.

I reviewed the existing MFA service configuration, including:

- app passwords
- trusted IPs
- verification options
- remembered MFA sessions

![MFA service settings](./Security%20settings%20assigned%20by%20global%20admin%20prev.png)

This also highlighted that Microsoft Entra still exposes some older per-user MFA controls alongside newer authentication-policy and Conditional Access functionality.

---

# Enabling Per-User MFA

I enabled MFA for selected test identities.

The user state changed from:

```text
Disabled
   ↓
Enabled
```

![Users enabled for MFA](./users%20enabled%20MFA.png)

The room covered three per-user MFA states:

```text
Disabled
→ Per-user MFA is not enabled

Enabled
→ MFA has been enabled, but registration may still be required

Enforced
→ MFA registration has been completed and MFA is enforced
```

Understanding these states is useful when troubleshooting why two users with apparently similar MFA configuration may experience different sign-in behaviour.

---

# Testing the User Experience

I then signed in with one of the MFA-enabled test accounts.

Instead of continuing directly into the account, Microsoft displayed:

> **Action Required**

The user was required to provide additional security information and configure Microsoft Authenticator.

![MFA registration required](./confirmation%20of%20MFA%20fron%20assigned%20users%20log%20in.png)

This demonstrated an important distinction:

```text
Administrator enables MFA
        ↓
User signs in
        ↓
Action Required
        ↓
User registers authentication method
        ↓
MFA can be used during authentication
```

Therefore:

```text
MFA Enabled
     ≠
MFA Registration Completed
```

Administrative enablement and end-user registration are separate stages.

---

# Conditional Access

The second major concept in the room was **Microsoft Entra Conditional Access**.

Conditional Access acts as a Zero Trust policy engine.

Instead of making an access decision based only on a username and password, it can evaluate multiple signals:

```text
User / Group
Location
Device
Application
Real-time risk
        ↓
Conditional Access
        ↓
Access Decision
        ↓
Allow
Require additional controls
Block
```

Possible grant requirements include:

- Require MFA
- Require authentication strength
- Require a compliant device
- Require an Entra hybrid joined device
- Require an approved client application
- Require an app protection policy
- Require password change

This makes authentication **adaptive** rather than applying the same requirement to every sign-in.

For example:

```text
Normal sign-in
→ Allow

Higher-risk sign-in
→ Require stronger verification

Unacceptable conditions
→ Block
```

---

## Conditional Access and Least Privilege

The room also introduced the **Conditional Access Administrator** role.

This role can manage Conditional Access policies without requiring the much broader Global Administrator role.

The same least-privilege principle applies:

```text
Identify the task
        ↓
Identify the required permission
        ↓
Assign the narrowest suitable role
```

However, Conditional Access Administrator is still a highly sensitive role.

An attacker with control over Conditional Access could potentially weaken controls such as:

- MFA requirements
- device requirements
- location restrictions
- access policies

Administrative IAM roles are therefore part of the organization's **attack surface** and require strong protection.

---

## Conditional Access Scope in This Room

The Conditional Access section was primarily **conceptual/read-through**.

I did **not** create a Conditional Access policy as part of this specific Zero Trust room.

Hands-on Conditional Access configuration is covered later in the **MS Entra ID: Authentication** room.


---

# Privilege Cleanup

After completing the MFA exercise and testing the user experience, I removed the temporary administrative role assignments from the lab account.

![Administrative role cleanup](./Admins%20roles%20cleanup.png)

The complete workflow was:

```text
Restricted lab account
        ↓
Assign Authentication Administrator
        ↓
Test permissions
        ↓
Discover access limitation
        ↓
Investigate Entra RBAC
        ↓
Assign Authentication Policy Administrator
        ↓
Access MFA configuration
        ↓
Enable MFA for test users
        ↓
Test user registration
        ↓
Verify expected behaviour
        ↓
Remove temporary administrative roles
```

This directly applied the Zero Trust principle of:

**Use least privilege**

rather than leaving unnecessary administrative privileges permanently assigned.

---

# Key Findings & Takeaways

### 1. Zero Trust is much broader than MFA

MFA is one control inside a larger access model.

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
Access policy
```

---

### 2. Microsoft Entra roles are highly task-specific

An administrative role containing the word **Authentication** does not automatically provide access to every authentication-related function.

The exact permissions must be checked against the task being performed.

---

### 3. The TryHackMe role did not match the current permission requirement

The room instructed the use of **Authentication Administrator**.

In my current Microsoft Entra tenant, that role did not provide access to the required per-user MFA administration area.

I confirmed the limitation through testing and then reviewed Microsoft's role permissions.

**Authentication Policy Administrator** provided the additional permissions required for the task.

This turned the exercise into a useful real-world RBAC troubleshooting scenario.

---

### 4. Authentication methods and authentication policy are separate responsibilities

Microsoft Entra separates:

```text
Managing user authentication methods
                ≠
Managing MFA / authentication policy
```

This supports both **least privilege** and **separation of duties**.

---

### 5. A permission failure is useful security information

The `401 / Insufficient privileges` response confirmed that the restricted administrator could not access configuration outside the assigned role.

The correct response was not to immediately grant Global Administrator.

Instead:

```text
Access denied
      ↓
Identify required permission
      ↓
Choose narrower role
```

---

### 6. MFA enablement and MFA registration are different stages

An administrator can enable MFA, but the user may still need to register a second authentication factor.

```text
MFA enabled
        ↓
Registration required
        ↓
Authenticator configured
```

---

### 7. Per-user MFA states matter

The distinction between:

```text
Disabled
Enabled
Enforced
```

is important when investigating MFA behaviour.

---

### 8. Security Defaults provide a baseline, not full adaptive access

Security Defaults provide useful tenant-wide protections.

Conditional Access provides more granular control by evaluating signals and applying different requirements depending on the situation.

---

### 9. Conditional Access is a major Zero Trust control

Conditional Access converts signals into policy decisions.

```text
Signals
   ↓
Policy evaluation
   ↓
Allow / Challenge / Block
```

This is significantly more flexible than relying on static authentication requirements.

---

### 10. Privileged IAM roles are high-value attack targets

Roles capable of modifying MFA, authentication policy, or Conditional Access can potentially weaken identity protections.

They should be:

- strongly authenticated
- monitored
- limited in number
- assigned only when required
- removed when no longer needed

---

### 11. Least privilege applies to administrators too

Administrative access should follow the same security principles as normal user access.

```text
Assign
  ↓
Use
  ↓
Verify
  ↓
Remove
```

The lab finished with the temporary roles removed.

---

### 12. Zero Trust maturity is gradual

Organizations generally progress from:

```text
Passwords + implicit trust
        ↓
MFA + basic access controls
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
- Zero Trust maturity model
- Microsoft Entra RBAC
- Least-privilege administration
- Authentication Administrator
- Authentication Policy Administrator
- RBAC permission troubleshooting
- Permission-boundary testing
- Per-user MFA
- MFA service settings
- MFA registration
- MFA user states
- Microsoft Authenticator onboarding
- Security Defaults
- Conditional Access concepts
- Adaptive access
- Separation of duties
- Privileged-role management
- Administrative-role cleanup

---

# References

- [NIST – Executive Order 14028: Improving the Nation's Cybersecurity](https://www.nist.gov/itl/executive-order-14028-improving-nations-cybersecurity)
- [OMB M-22-09 – Moving the U.S. Government Toward Zero Trust Cybersecurity Principles](https://www.whitehouse.gov/wp-content/uploads/2022/01/M-22-09.pdf)
- [CISA – Zero Trust Maturity Model](https://www.cisa.gov/resources-tools/resources/zero-trust-maturity-model)
