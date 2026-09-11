# MS Entra ID: External ID

Hands-on Microsoft Entra ID lab based on the TryHackMe **MS Entra ID: External ID** room.

After completing the room, I recreated the main exercises in my own Microsoft Entra tenant to practice **B2B guest collaboration, external identity hardening, invitation management, and guest access controls**.

---

## Lab Scenario

The objective was to allow external collaboration while reducing the risk created by guest identities.

The lab focused on:

- restricting guest directory visibility
- limiting who can invite external users
- disabling unnecessary authentication options
- restricting application registration for standard users
- testing an existing B2B guest invitation
- completing and verifying the invitation lifecycle

```text
External user
      ↓
Controlled invitation
      ↓
Restricted guest identity
      ↓
Only explicitly assigned access
```

---

# Hands-on Work

## 1. Harden External Collaboration Settings

I changed the guest access configuration to the most restrictive option:

> **Guest user access is restricted to properties and memberships of their own directory objects**

I also restricted guest invitations to:

> **Only users assigned to specific admin roles can invite guest users**

![External collaboration settings hardened](./External%20collaboration%20settings%20hardened.png)

This reduces unnecessary directory visibility and prevents normal users from freely creating external identities.

---

## 2. Disable Email One-Time Passcode

I reviewed the available external identity providers and disabled:

> **Email one-time passcode for guests**

![Email OTP disabled](./Email%20OTP%20disabled.png)

For this lab, I wanted guest authentication to use approved identity providers rather than adding another authentication path.

In a production environment, this decision would depend on business requirements and the organization's security policy.

---

## 3. Restrict Standard User Permissions

In **User settings**, I confirmed restrictive guest access and disabled application registration for normal users.

```text
Users can register applications: No
```

![User settings hardened](./User%20settings%20hardened.png)

This limits the ability of standard users to introduce new application identities and permission relationships into the tenant.

---

# B2B Guest Invitation Test

## 4. Review the Existing Guest

A guest account from an earlier lab already existed in the tenant.

![Guest user](./Guest%20User.png)

The account was enabled, but the B2B invitation had never been completed.

![Guest pending acceptance](./Guest%20pending%20acceptance.png)

```text
Guest object exists
        ↓
Invitation still pending
```

This showed that an existing Guest object does not necessarily mean that onboarding has been completed.

---

## 5. Resend the Invitation

Because the original invitation had not been received, I used **Resend invitation** instead of creating a duplicate guest.

![Resend guest invitation](./Resend%20guest%20invitation.png)

The new invitation was successfully delivered but was found in the recipient's **Spam** folder.

This was a useful troubleshooting finding: pending guest onboarding may be caused by email delivery rather than an Entra configuration issue.

---

## 6. Accept the Guest Invitation

I opened the invitation using the external account and reviewed the consent screen.

![Guest consent screen](./Guest%20consent%20screen.png)

After accepting, the guest successfully entered the tenant and could access the Microsoft **My Apps** portal.

![Guest My Apps portal](./Guest%20My%20Apps%20portal.png)

No applications had been assigned, so the guest had no application access.

```text
Authentication successful
        ↓
No assigned applications
        ↓
No unnecessary access
```

---

## 7. Verify the Invitation State

After completing the redemption process, I returned to Microsoft Entra and verified that the B2B invitation state had changed to:

> **Accepted**

![B2B invitation accepted](./B2B%20invitation%20accepted.png)

The complete lifecycle was therefore:

```text
Guest created
      ↓
Invitation pending
      ↓
Invitation resent
      ↓
Guest accepts
      ↓
Authentication succeeds
      ↓
B2B state = Accepted
```

---

# Security Observations

The lab produced several useful practical observations:

| Observation | Result |
|---|---|
| Guest object can exist before invitation redemption | Confirmed |
| Invitation delivery can fail or be filtered to Spam | Observed |
| Guest can authenticate without receiving application access | Confirmed |
| Guest directory visibility can be restricted | Configured |
| Normal users can be prevented from inviting guests | Configured |
| Standard user application registration can be disabled | Configured |

A particularly useful IAM lesson was the difference between:

```text
Authentication
= The guest proved who they are

Authorization
= The guest receives access only to assigned resources
```

The guest could authenticate successfully but still had **no applications available**.

---

# Least-Privilege Administration

Guest invitation is a security-sensitive capability.

For future guest onboarding, the preferred role is:

**Guest Inviter**

rather than using a broad administrative role such as Global Administrator.

```text
Need to invite guests
        ↓
Guest Inviter
        ↓
Only required permission
```

This keeps external-user administration aligned with least privilege.

---

# Configuration Summary

| Setting / Test | Result |
|---|---|
| Guest access visibility | ✅ Most restrictive |
| Guest invitations | ✅ Restricted to approved admin roles |
| Guest self-service sign-up | ❌ Disabled |
| Email OTP for guests | ❌ Disabled |
| Standard users register applications | ❌ Disabled |
| Existing guest reviewed | ✅ |
| Invitation resent | ✅ |
| Guest consent completed | ✅ |
| Guest successfully authenticated | ✅ |
| B2B invitation state | ✅ Accepted |
| Applications assigned to guest | 0 |

---

# Offensive Security Context

Alongside the lab, I reviewed how overly permissive external identities can be used for:

- directory reconnaissance
- user and group enumeration
- persistence through guest identities
- expanding an initial foothold inside a tenant

Research reviewed:

- **Azure Threat Research Matrix — AZT104: Gather User Information**
- **Azure Threat Research Matrix — AZT502.3: Guest Account / External Identity Persistence**
- **AADInternals — Quest for Guest**

This helped connect the defensive configuration with how guest identities may be abused from an attacker's perspective.

---

# Skills Practiced

- Microsoft Entra External ID
- B2B guest collaboration
- Guest invitation lifecycle
- External collaboration settings
- Guest access restrictions
- Guest invitation restrictions
- Guest Inviter role
- Email OTP configuration
- Application registration restrictions
- Invitation troubleshooting
- Guest redemption
- Authentication vs authorization
- Least privilege
- External identity attack-surface reduction
