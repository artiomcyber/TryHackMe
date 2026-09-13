# Microsoft Entra ID Protection — Risk Detection, Conditional Access and Identity Response

This project extends the Microsoft Entra ID Protection material from TryHackMe into a practical lab in my own Microsoft Entra tenant.

The original exercise focuses on **Identity Protection**, **risky users**, **risky sign-ins**, **risk-based Conditional Access**, and remediation. My tenant uses **Microsoft 365 Business Premium / Entra ID P1**, so some of the P2-only risk conditions from the original lab were unavailable.

Rather than stopping at that limitation, I adapted the exercise to the capabilities available in my tenant, validated Conditional Access behavior with other signals, generated a real Identity Protection detection through a controlled Tor sign-in, investigated the resulting risky identity, performed remediation, and then cleaned up the environment.

---

## Lab Scenario and Goal

The objective was to understand how Microsoft Entra can detect and respond to suspicious identity activity.

The lab focused on the following security workflow:

```text
Suspicious authentication activity
        ↓
Microsoft Entra evaluates identity signals
        ↓
Risk detection generated
        ↓
User/sign-in becomes risky
        ↓
Administrator investigates
        ↓
Conditional Access / manual remediation
        ↓
Identity contained or remediated
```

The main areas practiced were:

- Microsoft Entra Identity Protection
- Risk detections
- Risky users
- Suspicious sign-ins
- Microsoft Authenticator suspicious activity reporting
- Conditional Access
- Authentication strength
- Conditional Access What If analysis
- Least-privilege administrative roles
- Identity investigation and remediation
- Controlled risk simulation
- Cleanup and privilege removal

---

# 1. Reviewing Identity Protection and Available Risk Data

I first reviewed the Identity Protection area to understand what information was available in the tenant.

The dashboard provides visibility into:

- risky users
- risky sign-ins
- risk detections
- workload identity risk
- identity risk trends

![Identity Protection risk overview](./Identity%20risks.png)

At the start of the exercise there were no active risky users.

![Initial risky users view](./Risky%20users.png)

This established the baseline before generating any suspicious activity.

---

# 2. Licensing Limitation — P1 vs P2

The original TryHackMe lab required Conditional Access conditions based directly on:

- **User risk**
- **Sign-in risk**

These controls depend on Microsoft Entra ID P2 / Identity Protection capabilities.

My tenant uses **Microsoft 365 Business Premium with Entra ID P1**, so the risk-level conditions were not available inside the Conditional Access policy editor.

![Risk conditions unavailable](./Risk%20users%20level%20not%20available%20in%20P1%20Business%20Premium%20licences.png)

This was an important practical finding because the Conditional Access interface itself changes depending on licensing.

Instead of purchasing P2 purely for the lab, I adapted the exercise while preserving the same security concepts.

The revised objective became:

```text
Original THM exercise
Risk signal → Conditional Access → MFA/remediation

Adapted P1 exercise
Available signal → Conditional Access → authentication control

PLUS

Controlled suspicious activity
        ↓
Identity Protection detection
        ↓
Risky user investigation
        ↓
Manual remediation
```

This allowed me to practice both sides of the workflow without adding unnecessary licensing cost.

---

# 3. Applying Least Privilege to the Lab Administrator

Rather than performing all tasks permanently as Global Administrator, I used the dedicated lab account:

`THM-Lab-UserAdmin`

The account was assigned **Security Administrator** for Identity Protection investigation and security administration.

![Security Administrator assigned](./Sec%20admin%20role%20assigened.png)

During the lab I also discovered that some authentication-policy configuration required additional privileges.

I therefore added **Authentication Policy Administrator** rather than using Global Administrator for normal authentication-policy work.

![Authentication Policy Administrator added](./Adapting%20to%20the%20lab%20by%20creating%20Auth%20policy%20admin%20to%20the%20user.png)

The final lab account therefore temporarily held:

- Security Administrator
- Authentication Policy Administrator

![Lab administrator roles](./Conditions%20of%20CA.png)

This demonstrated an important IAM principle:

> Administrative access should be expanded only when a specific task requires it, rather than using Global Administrator for routine security operations.

---

# 4. Creating an Isolated Test Identity

A dedicated user was created for the risk simulation:

`riskyusertest`

![Risk test user created](./New%20riskyusertest%20created.png)

The account was then added to the existing `Project23` security group.

![Risk test user added to group](./Assigned%20new%20user%20to%20security%20group.png)

Using an isolated identity reduced the possibility of affecting normal lab users while allowing Conditional Access and Identity Protection behavior to be tested safely.

---

# 5. Configuring Microsoft Authenticator

Microsoft Authenticator was enabled as an authentication method in the tenant.

![Microsoft Authenticator settings](./Microsoft%20Authenticator%20settings%20PUSH.png)

The `riskyusertest` account then registered Microsoft Authenticator successfully.

![Microsoft Authenticator registered](./Auth%20added.png)

This was necessary because one of the later tests depended on the user being able to:

- receive authentication requests
- deny requests they did not initiate
- report suspicious authentication activity

---

# 6. Enabling Report Suspicious Activity

One particularly useful Identity Protection control is the ability for users to report an unexpected Microsoft Authenticator request as suspicious.

I enabled **Report suspicious activity** and scoped it to the test group rather than immediately enabling it tenant-wide.

![Suspicious activity reporting enabled](./Report%20suspicious%20activity%20enabled%20to%20the%20selected%20group.png)

The security model becomes:

```text
Unexpected authentication request
        ↓
User rejects request
        ↓
User reports it as suspicious
        ↓
Microsoft Entra receives the signal
        ↓
Identity risk can increase
        ↓
Security team investigates
```

This is especially useful for detecting scenarios such as:

- stolen passwords
- MFA fatigue attacks
- unauthorized login attempts
- credential stuffing
- attacker-triggered MFA prompts

---

# 7. Building the Adapted Conditional Access Policy

Because **sign-in risk** and **user risk** conditions were unavailable with P1, I created an adapted Conditional Access policy.

Policy name:

`LV426-SignInRisk-AZURE_LAB_ID`

![New Conditional Access policy](./New%20CA.png)

Initially the policy was scoped to the relevant test identity/group.

![Conditional Access scope](./CA%20to%20security%20group%20assigned.png)

---

## Target Resource

The policy targeted **Office 365**.

![Conditional Access resource](./CA%20resouces%20set.png)

This allowed the test to focus on a realistic cloud resource rather than applying the policy tenant-wide.

---

## Substitute Condition

Since the P2 risk conditions were unavailable, I used another Conditional Access signal that could be tested with my existing environment:

**Device platform = Android**

![Conditional Access condition](./Conditions%20of%20CA.png)

This did not attempt to reproduce Identity Protection risk scoring.

Instead, it allowed me to practice the same Conditional Access decision structure:

```text
WHO?
Specific test identity

WHAT?
Office 365

UNDER WHAT CONDITION?
Android device

WHAT CONTROL?
Authentication requirement
```

---

# 8. Configuring the Grant Control

Instead of using only the generic **Require multifactor authentication** control, I tested **Authentication strength**.

![Conditional Access grant](./CA%20Grant%20configured.png)

This demonstrated an important distinction.

Traditional MFA asks:

```text
Did the user perform MFA?
```

Authentication strength can ask:

```text
Which authentication methods are acceptable?
```

This makes it possible to enforce stronger authentication methods for sensitive resources.

Examples include:

- Multifactor authentication
- Passwordless MFA
- Phishing-resistant MFA
- custom authentication strengths

---

# 9. Enabling the Conditional Access Policy

After configuring:

- the test user
- Office 365 as the resource
- Android as the condition
- authentication strength as the grant control

the policy was enabled.

![Conditional Access policy enabled](./CA%20policies%20overview.png)

The tenant now contained both Microsoft-managed Conditional Access policies and my lab policy.

---

# 10. Validating the Policy with Conditional Access What If

Before relying on a Conditional Access policy, I used Microsoft's **What If** tool to evaluate the expected result.

The simulation used:

- User: `riskyusertest`
- Device platform: Android
- Client app: Browser
- Target resource: Office 365

![Conditional Access What If configuration](./What%20if%20setup.png)

The result showed that my policy would apply.

![Conditional Access What If result](./risk-based%20Conditional%20Access.png)

This was useful because the What If tool validates policy logic without repeatedly attempting live authentication.

It is also an important operational control before deploying broad Conditional Access policies.

---

# 11. Simulating Suspicious Authentication Activity

The original TryHackMe lab included an optional exercise for generating an **Anonymous IP address** risk detection using Tor.

I reproduced this in my own tenant.

The first attempts were useful but did not immediately create the desired risk detection.

I initially tested behavior similar to a password-reset attack:

```text
Attacker attempts password reset
        ↓
Victim receives Authenticator request
        ↓
Victim reports that the request was not initiated
```

The failed password-reset authentication appeared in the sign-in logs, but failed credentials alone did not represent the final Identity Protection scenario I wanted to test.

I then moved to a more realistic compromised-credential simulation.

---

# 12. Simulating a Compromised Password

I treated the test account password as though it had already been leaked.

Using the correct credentials, I attempted to authenticate to the account.

Microsoft Authenticator generated an approval request.

Instead of approving it, I rejected the request and reported that I had **not initiated the authentication**.

This models a common real-world scenario:

```text
Attacker obtains correct password
        ↓
Attacker attempts login
        ↓
MFA request reaches legitimate user
        ↓
User denies the request
        ↓
User reports suspicious activity
        ↓
Identity Protection receives another security signal
```

The authentication attempt was denied.

---

# 13. Anonymous IP Simulation with Tor

I also performed the TryHackMe optional risk simulation through the Tor network.

The test account was used to attempt authentication through a Tor exit node.

The Microsoft Authenticator request was again rejected rather than approved.

After allowing Identity Protection time to process the event, Microsoft Entra generated a real detection:

**Anonymous IP address**

The detected source appeared from:

`Kitchener, Ontario, CA`

![Risk detection generated](./Risky%20users%20detection.png)

This was one of the most important results of the project because the lab moved beyond simply reading about Identity Protection.

A real risk detection was generated inside my own tenant.

---

# 14. Risky User Generated

After the detection was processed, `riskyusertest` appeared in the **Risky users** report.

![Risky user appears](./Risky%20user%20appear.png)

The user profile also showed that the account was now considered risky.

![Risky user overview](./Risky%20user%20overview.png)

Opening the risky identity provided access to investigation information including:

- recent risky sign-ins
- risk detections
- risk history
- user information
- remediation actions

![Risky user details](./Risky%20user%20details%201.png)

Because the tenant does not include full P2 visibility, some information such as the detailed risk level appeared as **Hidden**.

![Risk level hidden](./Risky%20user%20compromised.png)

This clearly demonstrated the difference between:

```text
Detection capability
```

and

```text
Full Identity Protection licensing / risk detail visibility
```

---

# 15. Investigating the Detection

The Identity Protection detection history showed several security events associated with the account.

These included:

- Anonymous IP address
- generic risk-detected events
- risk state = At risk

![Risk detection history](./Risky%20users%20detection.png)

The anonymous IP event provided useful investigation fields including:

- user
- source IP
- geographic location
- detection type
- risk state
- request ID
- detection timestamp

This is the type of information an IAM or SOC analyst would correlate with:

- sign-in logs
- device information
- user activity
- MFA events
- Conditional Access results
- known travel
- IP reputation
- threat intelligence

before deciding whether an account has actually been compromised.

---

# 16. Confirming the User as Compromised

For the purposes of the lab, the suspicious activity was treated as a confirmed compromise.

Microsoft Entra provided the **Confirm user compromised** action.

![Confirm compromised prompt](./Admin%20confirmation%20of%20compromising.png)

This action tells Identity Protection that the activity represents a real compromise rather than a false positive.

After confirmation, another Identity Protection event appeared:

**Admin confirmed user compromised**

![Confirmed compromise detection](./Risky%20user%20compromised.png)

This is important because administrator feedback also contributes to Microsoft's future risk evaluation.

---

# 17. Role Boundary Discovered During Remediation

An additional permission finding appeared during the remediation process.

The dedicated security administration account could perform much of the investigation workflow but did not have permission for every remediation action available in the portal.

For the restricted action, I switched to the Global Administrator account and completed the remediation.

This was useful because it demonstrated that:

```text
Being able to view security risk
        ≠
Being authorized to perform every identity remediation action
```

Administrative roles in Entra intentionally separate capabilities.

This reinforces the value of:

- role separation
- least privilege
- temporary elevation
- privileged access management
- understanding the exact scope of Entra roles

---

# 18. Containing the Compromised Identity

The compromised test account was then disabled.

![Risky user blocked](./Risky%20user%20blocked%20via%20admin.png)

The user profile showed both:

- **Account status: Disabled**
- **Risky user: Confirmed compromised**

This represents the containment stage of identity incident response.

```text
Detection
   ↓
Investigation
   ↓
Confirm compromise
   ↓
Disable identity
   ↓
Stop further authentication
```

---

# 19. Key Findings

### P1 still allowed useful Identity Protection experimentation

Although P2-only risk-based Conditional Access conditions were unavailable, the tenant still provided enough functionality to:

- generate an Identity Protection detection
- identify a risky user
- review the detection
- test Conditional Access separately
- use authentication strength
- simulate Conditional Access with What If
- perform administrative remediation

---

### Risk detections are not always immediate

The Tor sign-in did not appear instantly.

Identity Protection required processing time before the **Anonymous IP address** detection became visible.

This is important during investigations because the absence of an immediate risk detection does not necessarily mean that the activity has not been detected.

---

### Failed passwords and compromised credentials are different scenarios

An incorrect password attempt generated a failed sign-in but was not equivalent to an attacker successfully possessing valid credentials.

The more realistic identity compromise scenario was:

```text
Correct password
+
unexpected MFA request
+
user rejects/report suspicious
```

This represents a stronger indicator that a valid credential may have been compromised.

---

### MFA denial can itself become a security signal

Users are not only authentication endpoints.

They can become part of the detection system.

When a legitimate user reports an unexpected Authenticator request, Microsoft Entra can use that feedback as an identity-risk signal.

---

### Anonymous networks can generate identity risk

The Tor simulation successfully created an **Anonymous IP address** detection.

This showed how network characteristics can contribute to identity-risk evaluation even when the username and password are valid.

---

### Conditional Access and Identity Protection solve different parts of the problem

Identity Protection answers:

```text
How suspicious is this identity or sign-in?
```

Conditional Access answers:

```text
Given the available signals, what should happen next?
```

Together they can form an automated identity-defense system.

---

### Authentication strength is more precise than generic MFA

A generic MFA policy only requires a second factor.

Authentication strength allows an organization to define **which authentication technologies are acceptable**.

For higher-risk systems this enables controls such as:

- passwordless MFA
- phishing-resistant MFA
- FIDO2/passkeys
- Windows Hello for Business
- certificate-based authentication

---

### What If should be part of Conditional Access deployment

The Conditional Access What If tool allowed the policy to be validated before depending on a real sign-in.

This helps reduce:

- accidental lockouts
- incorrect targeting
- missing exclusions
- unexpected policy combinations

---

### Security Administrator is powerful but not equivalent to Global Administrator

The lab exposed a real permission boundary.

The security role provided significant investigation capabilities, but one remediation action still required a more privileged administrator in this tenant.

That is a useful demonstration of RBAC and administrative separation rather than a failure of the lab.

---

# 20. Security Workflow Practiced

The complete project ultimately followed this sequence:

```text
Create isolated test identity
        ↓
Register Microsoft Authenticator
        ↓
Enable suspicious-activity reporting
        ↓
Configure Conditional Access
        ↓
Validate CA using What If
        ↓
Attempt authentication through Tor
        ↓
Reject unexpected MFA request
        ↓
Identity Protection detects Anonymous IP
        ↓
User becomes risky
        ↓
Investigate risk evidence
        ↓
Confirm compromise
        ↓
Disable compromised identity
        ↓
Remove temporary privileges and test objects
```

---

# 21. Skills Demonstrated

This project provided practical experience with:

**Identity Security**
- Microsoft Entra ID Protection
- risky users
- risky sign-ins
- identity risk signals
- suspicious authentication reporting
- compromised account response

**Conditional Access**
- policy scoping
- target resources
- device platform conditions
- authentication strength
- Microsoft-managed policies
- Conditional Access What If analysis

**Authentication**
- Microsoft Authenticator
- MFA
- push authentication
- authentication strength
- suspicious MFA reporting

**IAM Administration**
- security groups
- administrative roles
- least privilege
- Security Administrator
- Authentication Policy Administrator
- privilege escalation only when operationally required

**Incident Response**
- detection
- investigation
- validation
- containment
- account disablement
- remediation decision-making

**Security Testing**
- Tor-based anonymous IP simulation
- attacker-style credential scenarios
- validation of identity controls
- testing authentication failure behavior

---

# 22. Cleanup

After collecting the required evidence, the temporary lab configuration was removed.

Cleanup included:

- deleting the lab Conditional Access policies
- deleting `riskyusertest`
- removing temporary administrative roles from `THM-Lab-UserAdmin`
- removing test-specific access configuration
- leaving the tenant in a clean state

This is an important part of security lab work because privileged assignments and temporary policies should not remain active after they are no longer required.

---

## Final Architecture

```text
                   ┌──────────────────────┐
                   │   Microsoft Entra    │
                   │ Identity Protection  │
                   └──────────┬───────────┘
                              │
                     risk detections
                              │
             ┌────────────────┴───────────────┐
             │                                │
       Risky Sign-in                    Risky User
             │                                │
             └──────────────┬─────────────────┘
                            │
                    Investigation
                            │
              ┌─────────────┴─────────────┐
              │                           │
      Conditional Access            Admin Response
              │                           │
      authentication               confirm compromise
         strength                         │
              │                      disable account
              └─────────────┬─────────────┘
                            │
                       Containment
```

The biggest lesson from this project was that identity protection is not a single control.

Effective identity defense combines:

**signals → detection → policy → authentication → investigation → remediation.**
