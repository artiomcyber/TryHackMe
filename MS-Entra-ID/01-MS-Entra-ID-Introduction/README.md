# MS Entra ID: Introduction

Hands-on Microsoft Entra ID lab based on the TryHackMe **MS Entra ID: Introduction** room.

This room introduced the fundamentals of **Identity and Access Management (IAM)** and basic Microsoft Entra administration.

Because the TryHackMe cloud environment was temporarily unavailable, I recreated the practical exercises in my own Microsoft Entra lab tenant using fictional identities and a restricted **User Administrator** account.

---

## Room Overview

Topics covered:

- Identity and Identity Providers (IdP)
- Authentication (AuthN)
- Authorization (AuthZ)
- Identity types
- Microsoft Entra ID
- Microsoft Entra admin center
- User and group administration
- Identity lifecycle
- Administrative permissions
- Least privilege

A simple distinction reinforced throughout the lab:

```text
Authentication = Who are you?
Authorization  = What are you allowed to do?
```

Successfully signing in does not mean an identity is authorized to perform every action.

---

## Lab Setup

To recreate the TryHackMe environment, I created a temporary administrative account with only **User Administrator** role - THM-Lab-UserAdmin.

The account was intentionally **NOT** assigned Global Administrator privileges.

This allowed me to perform normal identity-management operations while also observing real Microsoft Entra permission boundaries.

The temporary account completed normal first-sign-in onboarding:

```text
Temporary password
        ↓
Forced password change
        ↓
Microsoft Authenticator registration
        ↓
Restricted lab administrator ready
```

---

# Hands-on Tasks

## 1. Create a Security Group

Created an assigned **Security Group** to represent the fictional LV-426 team.

```text
Group type: Security
Membership type: Assigned
```

![Security Group created](./Security%20Group%20created.png)

The exercise reinforced the basic group-based access model:

```text
Users
  ↓
Security Group
  ↓
Access / Resources / Policies
```

---

## 2. Create Two Users

Created two fictional internal Microsoft Entra identities:

- **Major Alex**
- **Sergeant Bill**

![Users created](./Users%20created.png)

This provided practical experience with basic cloud identity provisioning.

---

## 3. Add Users to the Group

Added both users as direct members of the Security Group.

![Group members](./Group%20members.png)

Managing access through groups is more scalable than assigning permissions separately to every user.

---

## 4. Delete and Restore a User

As part of the TryHackMe scenario, Sergeant Bill was deleted.

Microsoft Entra placed the account into the recoverable **Deleted users** state rather than immediately removing it permanently.

Sergeant Bill was then successfully restored.

![User restored](./User%20restored.png)

The exercise demonstrated the user lifecycle:

```text
Active
  ↓
Delete
  ↓
Soft-deleted
  ↓
Restore
  ↓
Active
```

---

## 5. Delegate Group Ownership

Major Alex was successfully assigned as owner of the Security Group.

![Group owner added](./Group%20owner%20added.png)

This demonstrated delegated administration:

```text
Group Owner
     ≠
Global Administrator
```

A user can manage a specific group without receiving tenant-wide administrative privileges.

---

## 6. Test Group Owner Removal

The next task required removing Major Alex as group owner.

Because Major Alex was the group's only owner, the operation was rejected and Microsoft Entra reported that the group must retain at least one owner.

![Owner removal blocked](./Owner%20removal%20blocked.png)

This was a useful real-world result.

Administrative permission alone does not guarantee that every operation is valid:

```text
Requested action
        ↓
Permission check
        ↓
Object / service rules
        ↓
Allowed or blocked
```

---

## 7. Test Directory Role Assignment

While still signed in as **User Administrator**, I attempted to assign a Microsoft Entra directory role to Sergeant Bill.

The role-assignment capability was unavailable.

![Directory role assignment restricted](./Directory%20role%20assignment%20restricted.png)

This demonstrated least privilege and separation of duties.

```text
User Administrator

Create users             ✅
Manage users             ✅
Manage groups            ✅
Reset passwords          ✅

Grant privileged roles   ❌
```

A normal user administrator should not be able to freely elevate another identity to a privileged administrative role.

---

## 8. Reset a User Password

Sergeant Bill's password was successfully reset.

![Password reset successful](./Password%20reset%20successful.png)

This showed an important permission distinction:

```text
Reset user password
        ✅

Grant privileged role
        ❌
```

Both affect an identity, but Microsoft Entra protects higher-risk administrative operations with stronger role requirements.

---

## 9. Test Access to Sign-in Logs

I attempted to review Sergeant Bill's sign-in activity while operating as User Administrator.

Access was denied with an authorization error.

![Sign-in logs access denied](./Sign-in%20logs%20access%20denied.png)

This demonstrated separation between identity administration and security monitoring:

```text
Identity Administration
        ↓
Manage users/groups
        ✅

Security Monitoring
        ↓
Investigate sign-in telemetry
        ❌
```

---

## 10. Disable a User

Major Alex's account was disabled as part of the simulated security scenario.

![Account disabled](./Account%20disabled.png)

Disabling and deleting an account are different actions:

```text
Disable
→ Identity remains
→ Sign-in blocked

Delete
→ Identity enters deleted-user lifecycle
```

Disabling can therefore be useful during investigations, temporary suspensions, or suspected credential compromise.

---

## 11. Clean Up the Lab

After completing the exercises, the temporary objects were removed.

- Security Group deleted
- Major Alex deleted
- Sergeant Bill deleted
- temporary exercise objects verified as removed

![Group cleanup](./Group%20cleanup.png)

![User cleanup](./User%20cleanup.png)

Cleanup is an important part of lab administration:

```text
Create
  ↓
Test
  ↓
Document
  ↓
Clean up
  ↓
Verify
```

Temporary accounts, groups, and permissions should not remain after testing is finished.

---

# Permission Testing Summary as USER ADMIN

| Action | Result |
|---|---|
| Create users | ✅ Allowed |
| Create Security Group | ✅ Allowed |
| Add group members | ✅ Allowed |
| Delete normal user | ✅ Allowed |
| Restore deleted user | ✅ Allowed |
| Add group owner | ✅ Allowed |
| Remove sole group owner | ⚠️ Blocked by group constraint |
| Reset user password | ✅ Allowed |
| Disable user account | ✅ Allowed |
| Assign privileged directory role | ❌ Not allowed |
| View sign-in logs | ❌ Not allowed |

The restricted administrator was able to perform normal identity-management tasks without receiving unrestricted tenant privileges.

---

# Key Takeaways

### Authentication is not Authorization

```text
Authentication
= proving who you are

Authorization
= determining what you can do
```

The lab administrator could authenticate successfully while still being denied specific operations.

### Least Privilege

The **User Administrator** role provided enough access for normal user and group administration without giving unrestricted control over the tenant.

### Separation of Duties

Different administrative responsibilities require different roles.

```text
User Administrator
→ users and groups

Privileged role administration
→ privileged roles

Security roles
→ security monitoring

Global Administrator
→ broad tenant administration
```

### Identity Lifecycle

Identity administration extends beyond simply creating users:

```text
Create
  ↓
Assign
  ↓
Manage
  ↓
Reset / Update
  ↓
Disable
  ↓
Delete
  ↓
Restore or permanently remove
```

# Skills Practiced

- Microsoft Entra ID
- Identity and Access Management
- Authentication
- Authorization
- User administration
- Security Group administration
- Group membership
- Group ownership
- Identity lifecycle
- Deleted-user recovery
- Password administration
- Account disablement
- Microsoft Entra roles
- Least privilege
- Separation of duties
- Permission troubleshooting
- Secure lab cleanup

---

# TryHackMe Room Completion

The **MS Entra ID: Introduction** room was successfully completed.

![TryHackMe room completed](./TryHackMe%20room%20completed.png)
