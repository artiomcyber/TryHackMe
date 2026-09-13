# MS Entra ID: Authentication

TryHackMe room covering **Microsoft Entra authentication methods, passwordless authentication, authentication strengths, FIDO2/passkeys, Conditional Access, Password Protection, and Self-Service Password Reset (SSPR)**.

The main hands-on exercise was recreated in my own Microsoft Entra tenant. I built a **Conditional Access policy requiring Passwordless MFA for the Teams Administrator role**, resolved a Security Defaults conflict, assigned a test administrator, and verified the passwordless sign-in experience by creating a passkey.

---

## Authentication Methods

Microsoft Entra supports authentication methods with different levels of security and usability.

![Authentication methods comparison](./Password%20ratings.png)

The general progression is:

```text
Password
   ↓
Password + MFA
   ↓
Passwordless
   ↓
Phishing-resistant authentication
```

Methods covered in the room included:

- Microsoft Authenticator
- Windows Hello for Business
- Passkeys / FIDO2 security keys
- Certificate-based authentication
- Temporary Access Pass
- Software and hardware OATH tokens
- SMS and voice

Passwordless authentication removes the reusable password from the sign-in process and replaces it with stronger authentication based on a trusted authenticator, cryptographic key, biometric, or local PIN.

![Security and convenience comparison](./Password%20auth%20conv%20vs%20security.png)

A key distinction is:

```text
Passwordless
does not automatically mean
Phishing-resistant
```

For sensitive access, organizations can require specific phishing-resistant methods instead of accepting every passwordless option.

---

## Authentication Strengths

**Authentication Strength** is a Conditional Access control that defines which authentication methods are acceptable for a resource.

Instead of only requiring:

```text
MFA
```

an administrator can require:

```text
Passwordless MFA
```

or:

```text
Phishing-resistant MFA
```

Microsoft Entra provides built-in strengths for:

| Authentication Strength | Purpose |
|---|---|
| Multifactor authentication | Broad set of accepted MFA combinations |
| Passwordless MFA | Authentication without a traditional password |
| Phishing-resistant MFA | Stronger protection for sensitive access |

This allows authentication requirements to match the sensitivity of the resource.

For example:

```text
Standard business application
→ MFA

Privileged or sensitive resource
→ Phishing-resistant MFA
```

Organizations can also create custom authentication strengths containing only approved authentication methods.

---

## FIDO2 and Passkeys

FIDO2/passkeys use public-key cryptography rather than a reusable password.

![FIDO2 authentication flow](./FIDO2%20Authentication%20Flow.png)

A simplified model is:

```text
Registration

Private key
→ remains protected on the authenticator

Public key
→ registered with Microsoft Entra
```

During authentication:

```text
Entra sends challenge
        ↓
Authenticator signs challenge
        ↓
Entra verifies signature
        ↓
Authentication succeeds
```

The private key is not sent to Microsoft Entra, making passkeys much more resistant to traditional credential phishing.

---

# Hands-on Lab: Passwordless Conditional Access

## Scenario

The practical task was to create a Conditional Access policy requiring **Passwordless MFA** for users holding the:

**Teams Administrator**

directory role when accessing:

**Office 365**

The policy was named:

```text
Passwordless Test
```

---

## 1. Assign Conditional Access Administrator

I reused my restricted lab administrator and temporarily assigned:

**Conditional Access Administrator**

instead of performing the task with Global Administrator.

![Conditional Access Administrator assigned](./Assigning%20role%20to%20user.png)

This followed the least-privilege approach used throughout the Entra labs.

---

## 2. Target the Teams Administrator Role

The Conditional Access policy was configured to target the **Teams Administrator directory role**.

![Teams Administrator targeted](./Setting%20up%20CA.png)

This means the policy follows role membership rather than depending on a manually maintained user list.

```text
User receives Teams Administrator
        ↓
User enters policy scope
        ↓
Passwordless requirement applies
```

---

## 3. Target Office 365

The target resource was configured as:

```text
Office 365
```

![Office 365 target resource](./Setting%20up%20CA%202.png)

The policy therefore became:

```text
WHO?
Teams Administrator

WHAT?
Office 365

REQUIREMENT?
Passwordless MFA
```

---

## 4. Require Passwordless MFA

Under the grant controls, I selected:

```text
Grant access
+
Require authentication strength
+
Passwordless MFA
```

![Passwordless MFA grant control](./CA%20Grant.png)

An important finding was that Microsoft Entra does not allow the generic:

```text
Require multifactor authentication
```

and:

```text
Require authentication strength
```

controls to be used together in this configuration.

Authentication Strength already defines the accepted authentication requirement, so the more specific control is used instead.

---

## Finding: Security Defaults Blocked Conditional Access

When I attempted to enable the new policy, Microsoft Entra blocked the action because **Security Defaults were still enabled**.

![Security Defaults conflict](./CA%20error%20of%20sec%20def.png)

The tenant returned:

```text
Security defaults must be disabled
to enable Conditional Access policy
```

This demonstrated an important relationship between Microsoft's basic tenant security baseline and custom Conditional Access.

For the lab, I selected the option indicating that the organization was moving to Conditional Access.

![Disable Security Defaults](./Dissabled%20SD.png)

Microsoft then confirmed that Security Defaults had been successfully disabled.

![Security Defaults disabled](./SD%20disabled.png)

After that, the custom Conditional Access policy could be activated successfully.

![Conditional Access policy created](./CA%20created.png)

### Production consideration

Security Defaults should not simply be disabled without replacement controls.

In a production environment, equivalent or stronger Conditional Access protections should be designed and tested before removing the default baseline.

---

## 5. Assign Teams Administrator to a Test User

A dedicated test identity was assigned the:

**Teams Administrator**

role.

![Teams Administrator assigned](./User%20assigned%20Teams%20admin.png)

Because the policy targeted the directory role, the user automatically became subject to the Passwordless MFA requirement.

This showed the difference between two important Entra controls:

```text
RBAC
→ What can this identity administer?

Conditional Access
→ Under what conditions can this identity access resources?
```

---

## 6. Verify the Passwordless User Experience

When the Teams Administrator account signed in, Microsoft required the user to configure passwordless authentication.

![Passwordless setup required](./Teams%20admin%20log%20in%20require%20passwordless%20set%20up.png)

The user was offered passkey-based sign-in using options such as:

- face
- fingerprint
- PIN
- passkey

After registration, Microsoft confirmed that the passkey had been created successfully.

![Passkey created](./Passwordless%20set%20up%20created.png)

The final workflow was:

```text
Conditional Access policy
        ↓
Teams Administrator targeted
        ↓
Office 365 targeted
        ↓
Passwordless MFA required
        ↓
Test user receives Teams Administrator
        ↓
User signs in
        ↓
Passwordless setup required
        ↓
Passkey created
```

This combined several Entra concepts in one practical exercise:

```text
RBAC
+
Conditional Access
+
Authentication Strength
+
Passwordless Authentication
+
Passkeys
```

---

# Password Protection

The room also covered **Microsoft Entra Password Protection** for identities that still rely on passwords.

Important protections include:

- Smart Lockout
- Microsoft's global banned password list
- Custom banned password lists
- Password protection for supported hybrid Active Directory environments

The global banned password list blocks commonly weak passwords based on Microsoft's security intelligence.

Organizations can add their own custom terms such as:

```text
Company names
Product names
Office locations
Brand names
Internal terminology
```

This helps prevent predictable passwords based on information that may be publicly known.

---

## Password Spray Attacks

The room also covered **password spraying**.

Unlike traditional brute force:

```text
Many passwords
→ One account
```

password spraying typically uses:

```text
One or a few weak passwords
→ Many accounts
```

For example:

```text
Password123 → User A
Password123 → User B
Password123 → User C
Password123 → User D
```

This technique attempts to avoid repeatedly failing against one account and triggering lockout thresholds.

Password Protection, Smart Lockout, MFA, and passwordless authentication all help reduce the effectiveness of these attacks.

---

# Self-Service Password Reset — SSPR

The final topic was **Self-Service Password Reset**.

SSPR allows users to recover access without requiring the help desk to manually reset their password.

A simplified process is:

```text
User starts SSPR
        ↓
Identity is verified
        ↓
Approved recovery method used
        ↓
Password reset
```

The relationship between MFA and SSPR registration was also covered.

![MFA and SSPR registration flow](./azure%20mfa%20sspr.png)

Organizations can require one or more registered methods depending on their recovery policy.

SSPR can also be introduced gradually:

```text
Selected users/groups
        ↓
Pilot
        ↓
Organization-wide rollout
```

---

# Key Findings & Takeaways

### Authentication Strength is more precise than generic MFA

```text
Require MFA
→ broad MFA requirement

Require Authentication Strength
→ specific approved authentication methods
```

This is especially useful for privileged and sensitive resources.

### Passwordless and phishing-resistant authentication are not identical

Passwordless removes the traditional password, while phishing-resistant authentication additionally requires methods designed to resist credential interception and fake sign-in pages.

### Conditional Access can target directory roles

Targeting **Teams Administrator** means the security policy automatically follows users who receive that role.

This is more scalable than creating separate policies for individual administrators.

### Security Defaults and custom Conditional Access require a deliberate transition

In my tenant, Security Defaults prevented activation of the custom Conditional Access policy.

The lab therefore demonstrated that moving from Security Defaults to Conditional Access is an architectural change, not simply another policy switch.

### RBAC and Conditional Access solve different problems

```text
RBAC
→ permissions

Conditional Access
→ access conditions
```

Both are important for protecting privileged identities.

### Passkeys remove reusable credentials

The user's private key remains protected on the authenticator, reducing the value of stolen passwords and traditional phishing pages.

### Password Protection remains important during migration

Organizations rarely become fully passwordless immediately. Smart Lockout and banned-password protection remain relevant while password-based accounts still exist.

### SSPR combines security with operational efficiency

Users can recover their own accounts while administrators still control which authentication methods are acceptable for identity verification.

---

# Skills Practiced

- Microsoft Entra ID
- Authentication Methods
- Authentication Strengths
- Passwordless MFA
- Phishing-resistant authentication
- FIDO2 / Passkeys
- Public-key authentication
- Conditional Access
- Conditional Access Administrator
- Directory-role targeting
- Office 365 access policies
- Security Defaults
- Entra RBAC
- Password Protection
- Smart Lockout
- Global and custom banned passwords
- Password spray concepts
- Self-Service Password Reset
- Least-privilege administration
