# Runbook: phishing and account takeover

Lab worksheet for one test user in your own tenant. Follow it while you practise. Write answers in your report.

The phases follow NIST SP 800-61: preparation, detection and analysis, containment, eradication, recovery, and post incident. Containment, eradication, and recovery are separate sections of that life cycle.

This is not legal advice. The POPIA note in the post incident section is a reminder to escalate. It does not decide that a notice is required.

## Scope

In scope:

- A cloud account in your lab tenant, `lab-phish-test`, with no directory roles
- Sign-in and audit activity for that account, if those tables already exist or you enable the free Entra connector with the cost warning in the lab guide
- The Sentinel playbooks in this repo, as the notification and enrichment path

Out of scope:

- Employer, customer, university, or family data
- A real phishing message, a real payload, or a link you do not control
- Disabling your only administrator
- Paid trials for Defender for Office 365, Entra ID Protection, or another product that can start billing

If the only Global Administrator is you, create `lab-phish-test` first and never use block sign-in, password reset, or revoke sessions on the administrator account as part of this drill.

## 1. Preparation

Time: about 45 minutes the first time, shorter when the lab guide is already done.

Confirm:

- [ ] You can sign in to the lab tenant with an administrator account you will not disable
- [ ] `lab-phish-test` exists, has no directory role, and the password is in a password manager
- [ ] Microsoft Sentinel is the workspace from Sentinel-Home-SOC-Lab
- [ ] The three playbooks and the automation rule are deployed
- [ ] You know how to disable the test analytics rule
- [ ] A Teams channel exists for lab alerts, or you have written down another way you will notice a test incident

Contacts for this lab. Add rows when you have them. The only required contact is you.

| Role | Name | How to reach | When |
| --- | --- | --- | --- |
| Analyst | Mosa Mofokeng | [your lab channel or email] | First response |
| Backup admin | [name or "not created"] | | If the analyst account is locked |
| Privacy or legal | [not in this lab] | | Real incidents only |

Tables this runbook queries:

| Table | Connector | Use it when |
| --- | --- | --- |
| `SigninLogs` | Microsoft Entra ID, sign-in logs | The table is already in the workspace, or you accept the ingestion cost and enable only this category |
| `AuditLogs` | Microsoft Entra ID, audit logs | Same as above |
| `OfficeActivity` | Office 365 | Only if the connector is already on. If the table is missing, write "not present" and stop. Do not start a paid trial to create it |

Do not enable Entra ID Protection or email security products for this runbook. If `AADUserRiskEvents` or `EmailEvents` are missing, that is an acceptable lab result. Write "table not present".

## 2. Detection and analysis

Goal: decide whether the test account shows activity you cannot explain, and collect notes you can defend.

### Triage questions

Answer from data you queried or from the incident. If you did not look, write "not checked".

1. Which account is it? Confirm it is `lab-phish-test`. Leave your administrator account out of the drill.
2. What is the first and last time in scope, in UTC?
3. Did a person type a password into a site they did not mean to trust? In this lab the expected answer is no, unless you ran a drill you designed.
4. Did sign-in succeed? From which application?
5. Was MFA required, satisfied, or not configured?
6. Is there a new inbox rule, forwarding address, or mailbox delegate?
7. Is there a new OAuth consent or enterprise application?
8. Did the account send mail after the suspicious time?
9. Does the account have a directory role? The test user must not.
10. What personal information could this account read? For the test user, list the mail and files it can actually open. If the answer is none, write none.
11. Is the activity your own test? Say so in the report. Do not describe the synthetic `print` incident as a phishing email you received.

### Queries

Replace `<test-user-upn>` with the test user principal name. Run them in the Sentinel logs blade for the last 7 or 14 days. Paste the result into your report only after you redact IPs, emails, and tenant ids, or describe the row count and the columns you saw.

Sign-ins:

```kusto
SigninLogs
| where TimeGenerated > ago(7d)
| where UserPrincipalName =~ "<test-user-upn>"
| project TimeGenerated, UserPrincipalName, IPAddress, Location, AppDisplayName, ResultType, ResultDescription, AuthenticationRequirement, ConditionalAccessStatus, RiskLevelDuringSignIn
| order by TimeGenerated desc
```

If `SigninLogs` is missing, stop and record that. Enabling the Entra sign-in category ingests data into Log Analytics and can incur a charge. Turn it off again when the lab is over if you enabled it only for this drill.

Directory audit, broad:

```kusto
AuditLogs
| where TimeGenerated > ago(14d)
| where TargetResources has "<test-user-upn>"
| project TimeGenerated, OperationName, Result, InitiatedBy
| order by TimeGenerated desc
```

Directory audit, actions you would hunt for after a phish. Operation names differ by tenant. If this returns nothing, use the broad query and read the operation names yourself.

```kusto
AuditLogs
| where TimeGenerated > ago(14d)
| where OperationName in (
    "Reset user password",
    "Disable account",
    "Enable account",
    "Consent to application",
    "Add service principal",
    "Update application")
| project TimeGenerated, OperationName, Result, InitiatedBy, TargetResources
| order by TimeGenerated desc
```

Mailbox changes, only when `OfficeActivity` exists:

```kusto
OfficeActivity
| where TimeGenerated > ago(14d)
| where UserId =~ "<test-user-upn>"
| where Operation has "Inbox" or Operation has "Forward"
| project TimeGenerated, UserId, Operation, ClientIP
| order by TimeGenerated desc
```

Your own containment actions, after you perform them:

```kusto
AuditLogs
| where TimeGenerated > ago(1d)
| where OperationName has "revoke" or OperationName has "password" or OperationName has "Disable"
| project TimeGenerated, OperationName, Result, InitiatedBy, TargetResources
| order by TimeGenerated desc
```

### What to write down

- Query, time range, and row count
- The fields you used to decide
- Anything you expected and did not find
- Queries you could not run because the table is absent

A score from the enrichment playbook is context about an IP. It is not, by itself, proof that the account was taken over.

## 3. Containment

Do this only on `lab-phish-test`. Stop if you are looking at any other user.

Containment limits damage while you still need the account object for evidence. Block sign-in stops new authentications. Revoke sessions is the step that deals with tokens that already exist. Deleting the user is not part of this lab.

Microsoft Entra admin center, `https://entra.microsoft.com`:

1. Identity, Users, All users. Open `lab-phish-test`. Check that directory roles are empty.
2. Revoke sessions. Confirm the prompt. This is the control for existing refresh tokens and session cookies.
3. Reset password. Generate a password, store it in a password manager, and require a change at next sign-in if that box is shown. Do not put the password in the repo, the incident comment, or Teams.
4. Block sign-in. On the user properties, set account enabled to no, or use Block sign-in, and save. The user object stays in the directory.
5. Open Authentication methods. Remove a method only if you added it for the drill and you know it is not your admin method. If the list is empty, write "no methods".
6. Check enterprise applications and consent for grants you do not recognise. If there are none, write "no unexpected grants". Removing a grant you do not understand can break a lab app. Record the application name before you remove anything.

Communication while the account is blocked. Send this to yourself in the lab Teams channel or keep it in the report.

```
Status: containment in progress
Time (UTC):
Account: lab-phish-test
What I know:
What I have done:
What I have not done:
Next update:
```

Do not include the password, an API key, or a full incident URL if the channel is wider than your lab team.

## 4. Eradication

Remove what the attacker would have changed. In a lab where you did not plant a change, the result is a list of checks that found nothing. That is a valid result. Write it that way.

On `lab-phish-test` only:

1. Inbox rules and forwarding. If the user has no mailbox, write "no mailbox" and do not assign a paid Exchange licence.
2. Delegates and mail forwarding addresses, same condition.
3. OAuth grants and new app registrations tied to the test user.
4. Authentication methods you did not create.
5. Revoke sessions again after the password reset. A password change does not always remove every existing session. The second revoke is the step that closes that gap.

Run the audit query from the detection section again and note the operations that match your own actions, so you can tell them apart from anything else.

## 5. Recovery

1. Unblock sign-in for `lab-phish-test`.
2. Sign in as that user from a device you control, with the new password, and complete MFA if you configured it for the drill.
3. Confirm a new row in `SigninLogs` for that success, if the table exists. Redact the IP in anything you commit.
4. Confirm the user still has no directory roles.
5. Tell yourself, in the same status note, that the test account is usable again.

If sign-in fails, leave the account blocked and record the error. Do not weaken a conditional access policy on the whole tenant to finish the drill.

## 6. Post incident

### Notes to file

- Timeline of what you did, in UTC, with no secret values
- Which queries returned rows, and which tables were missing
- Whether the SOAR playbooks helped you see the incident faster
- What you would automate next, and what you would keep as a human step (password reset and block sign-in are easy to automate and easy to aim at the wrong user)

Put the write-up in `docs/report.md`, using [docs/report-template.md](../docs/report-template.md).

### POPIA

This lab must not use personal information of other people. If you followed the guide, there is no breach to notify.

In a real incident involving personal information of people in South Africa, section 22 of the Protection of Personal Information Act 4 of 2013 can require the responsible party to notify the Information Regulator and the data subjects as soon as reasonably possible after discovering that personal information was accessed or acquired by an unauthorised person. The Act includes exceptions, including where the identity of the data subject cannot be established. The Regulator describes the notification process at [https://inforegulator.org.za](https://inforegulator.org.za).

In a job, you would tell the privacy officer or legal counsel what data was involved, when you found out, and what you have contained. You would not send the notice yourself unless that was your assigned role. This runbook does not tell you that a notice is required in any particular case.

### MITRE ATT&CK mapping for this scenario

Use this as the behaviour the runbook is written for. Do not mark a technique as observed unless your notes show it.

| Technique | Name | Where this runbook looks |
| --- | --- | --- |
| T1566 | Phishing | The scenario, and the triage question about a site the user did not mean to trust |
| T1566.002 | Spearphishing Link | Same triage question, focused on a link |
| T1078 | Valid Accounts | Use of a real account after the phish |
| T1078.004 | Cloud Accounts | The Entra test user |
| T1621 | Multi-Factor Authentication Request Generation | MFA outcome in sign-in logs |
| T1098 | Account Manipulation | New methods, roles, or consents in audit logs |
| T1114.003 | Email Forwarding Rule | `OfficeActivity` hunt, when the table exists |
| T1528 | Steal Application Access Token | OAuth consent review |
| T1550.004 | Web Session Cookie | Revoke sessions during containment |

## Close the lab

Return to the cleanup list in [docs/lab-guide.md](../docs/lab-guide.md). Leave `lab-phish-test` unblocked unless you have a reason to keep it blocked, and never leave your only administrator blocked.
