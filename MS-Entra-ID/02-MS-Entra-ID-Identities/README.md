# MS Entra ID: Identities

Hands-on Microsoft Entra ID lab based on the TryHackMe **MS Entra ID: Identities** room.

This lab focused on **Microsoft Entra administrative roles, role assignment, permission boundaries, and least privilege**.

Because the TryHackMe cloud environment was unavailable, I recreated the practical exercise in my own Microsoft Entra lab tenant.

---

## Lab Scenario and Goal

Two new users, **Corporal Hicks** and **Private Vasquez**, need application-management access.

To assign them the **Application Administrator** role, my existing **THM-Lab-UserAdmin** account was temporarily granted the powerful **Privileged Role Administrator (PRA)** role from my Global Administrator account.

The problem is that PRA provides far more authority than should be kept for normal day-to-day work.

The goal of the lab was therefore to:

- create the two users
- assign both users the Application Administrator role
- practice two different role-assignment workflows
- verify the assignments
- remove the elevated PRA role from the lab administrator once the work was complete
- maintain the principle of **least privilege**

```text
Need elevated access
        ↓
Grant PRA temporarily
        ↓
Assign required roles
        ↓
Verify access
        ↓
Task completed
        ↓
Remove PRA
        ↓
Return to lower privilege
```

---

# Hands-on Tasks

## 1. Temporarily Elevate the Lab Administrator

The existing **THM-Lab-UserAdmin** account already had the **User Administrator** role from the previous lab.

Using my Global Administrator account, I temporarily added:

**Privileged Role Administrator**

![Privileged Role Administrator added](./Privileged%20Role%20Administrator%20added.png)

The lab account now had:

```text
THM-Lab-UserAdmin

User Administrator
        +
Privileged Role Administrator
```

The PRA role provided the authority required to manage Microsoft Entra administrative role assignments.

---

## 2. Create the Two Lab Users

From the Global Administrator account, I created:

- **Corporal Hicks**
- **Private Vasquez**

![Users created](./Users%20created.png)

These were temporary fictional identities created specifically for the lab.

---

## 3. Assign Application Administrator — User → Role

After switching to **THM-Lab-UserAdmin**, I started from the user account and assigned **Application Administrator** to Corporal Hicks.

```text
Corporal Hicks
      ↓
Assigned roles
      ↓
Add assignment
      ↓
Application Administrator
```

![Corporal Hicks Application Administrator](./Corporal%20Hicks%20Application%20Administrator.png)

This workflow starts with:

```text
USER
 ↓
ROLE
```

---

## 4. Assign Application Administrator — Role → User

For Private Vasquez, I used the opposite workflow.

Instead of starting from the user, I opened the **Application Administrator** role and added Private Vasquez to it.

```text
Application Administrator
        ↓
Add assignment
        ↓
Private Vasquez
```

![Private Vasquez Application Administrator](./Private%20Vasquez%20Application%20Administrator.png)

This workflow starts with:

```text
ROLE
 ↓
USER
```

Both methods produced the same final result.

---

## 5. Verify the Role Assignments

I opened the **Application Administrator** role and reviewed its assignments.

Both users were listed:

- Corporal Hicks
- Private Vasquez

![Application Administrator assignments](./Application%20Administrator%20assignments.png)

This also demonstrated two useful administrative views:

```text
User → Which roles does this user have?

Role → Which users have this role?
```

---

# Least Privilege

## 6. Remove My Own Privileged Role

The required administrative work was now complete.

There was no longer a reason for THM-Lab-UserAdmin to retain **Privileged Role Administrator**.

I therefore attempted to remove the elevated permissions from my own lab administrator account.

At that moment the account had:

- Privileged Role Administrator
- User Administrator

I selected both roles for removal.

Microsoft Entra returned:

> **Assignments removal partially succeeded**

![Role removal partially succeeded](./Role%20removal%20partially%20succeeded.png)

After refreshing, **Privileged Role Administrator had successfully been removed**, while User Administrator remained.

![Privileged Role Administrator removed](./Privileged%20Role%20Administrator%20removed.png)

The important security goal had been achieved:

```text
Privileged task completed
        ↓
PRA no longer required
        ↓
PRA removed
```

---

# Additional Findings

While completing the lab, I continued testing the environment instead of stopping after the required TryHackMe tasks.

This produced two useful permission-boundary findings.

---

## Finding 1 — Privilege Reduction Took Effect Immediately

I attempted to remove both PRA and User Administrator in one operation.

Only the PRA removal completed.

Once the higher privileged role was removed, the account no longer had the same authority to continue managing its remaining administrative role assignment.

```text
PRA available
      ↓
Remove PRA
      ↓
Higher authority lost
      ↓
Remaining role stays assigned
```

This showed how changes to administrative privilege can immediately affect what an account is authorized to do.

---

## Finding 2 — User Administrator Could Not Delete the New Administrators

After PRA was removed, THM-Lab-UserAdmin still had **User Administrator**.

I then attempted to delete Corporal Hicks and Private Vasquez.

Microsoft Entra returned:

> **Insufficient privileges to delete the selected users**

![User deletion denied](./User%20deletion%20denied.png)

Both users had already been assigned the **Application Administrator** role.

In this configuration, the remaining User Administrator role was not sufficient to delete those privileged identities.

This was a useful practical reminder that:

```text
Administrator
      ≠
Authority over every administrator
```

Administrative roles have their own permission boundaries and protections.

---

# Final Cleanup

Because THM-Lab-UserAdmin no longer had sufficient authority to finish the cleanup, I returned to the **Global Administrator** account.

From there I:

- removed the remaining **User Administrator** role from THM-Lab-UserAdmin
- deleted Corporal Hicks
- deleted Private Vasquez
- reviewed the Deleted users area
- permanently removed the disposable lab identities after confirming they were no longer required

![Lab users permanently deleted](./Lab%20users%20permanently%20deleted.png)

The final lab administrator state was:

```text
THM-Lab-UserAdmin

Privileged Role Administrator   ❌
User Administrator              ❌
```

This returned the account to a non-privileged state after the work was finished.

---

# Permission Testing Summary

| Action | Result |
|---|---|
| Grant PRA to THM-Lab-UserAdmin | ✅ Allowed from Global Admin |
| Create Corporal Hicks and Private Vasquez | ✅ Allowed |
| Assign Application Administrator | ✅ Allowed with PRA |
| Assign role using User → Role workflow | ✅ Allowed |
| Assign role using Role → User workflow | ✅ Allowed |
| Verify role membership | ✅ Allowed |
| Remove own PRA role | ✅ Allowed |
| Remove both own admin roles together | ⚠️ Partially succeeded |
| Delete Application Administrator users with User Administrator | ❌ Insufficient privilege |
| Remove remaining User Administrator role | ✅ Completed from Global Admin |
| Delete temporary users | ✅ Completed from Global Admin |
| Permanently clean up disposable identities | ✅ Completed |

---

# Key Takeaways

### Least Privilege

The main lesson from this lab was simple:

```text
Use elevated privilege
ONLY when required
        ↓
Complete the task
        ↓
Remove the privilege
```

Keeping PRA permanently assigned would create unnecessary risk.

---

### Two Ways to Assign Roles

Microsoft Entra allows role assignments from either direction:

```text
User → Role
```

or:

```text
Role → User
```

Knowing both workflows is useful when managing or auditing access.

---

### Privileged Roles Change Permission Boundaries

After PRA was removed, the lab account immediately had fewer capabilities.

The failed cleanup operations helped demonstrate that administrative access is not unlimited simply because an account still holds another administrator role.

---

### Errors Were Part of the Learning

The two most useful unexpected results were:

```text
Assignments removal partially succeeded
```

and:

```text
Insufficient privileges to delete the selected users
```

These provided practical evidence of Microsoft Entra authorization and administrative-role boundaries rather than only following a successful walkthrough.

---

# Skills Practiced

- Microsoft Entra ID
- Microsoft Entra administrative roles
- Privileged Role Administrator
- Application Administrator
- User Administrator
- Role assignment
- User → Role workflow
- Role → User workflow
- Role membership verification
- Least privilege
- Privileged access reduction
- Administrative permission boundaries
- Authorization troubleshooting
- Secure identity cleanup

---

# TryHackMe Room Completion

The **MS Entra ID: Identities** room was successfully completed.

![TryHackMe room completed](./TryHackMe%20room%20completed.png)

**Status:** ✅ Completed

**Next room:** MS Entra ID: Hybrid Identities
