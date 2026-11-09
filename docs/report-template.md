# Lab report: Sentinel SOAR

Copy this file to `docs/report.md` and fill `docs/report.md`. Leave this template blank so the prompts stay in git.

Author: Mosa Mofokeng

Date of the test (UTC): [write the date you ran the test]

Workspace name: [name only, no subscription id]

## 1. Purpose

[In your own words, what you built and what you wanted to show.]

## 2. Environment

Answer from your tenant. Do not paste identifiers you do not want to publish.

- Sentinel workspace came from the Sentinel-Home-SOC-Lab repo: [yes or no]
- Resource group location:
- Logic Apps deployed: [list the three names]
- Key Vault created for this lab: [yes or no, name only]
- Teams channel used: [channel name only]
- AbuseIPDB plan shown on the account page: [for example free, and the daily limit the site displayed]
- Data tables that already existed (`SigninLogs`, `AuditLogs`, `OfficeActivity`, others):

## 3. What you deployed

[List the playbooks, the automation rule, and the analytics rule. Note anything you changed from the templates, such as severity or a VirusTotal swap.]

## 4. How you tested

Follow [testing.md](testing.md). Then write what you did, including the time you enabled the analytics rule and the time you disabled it.

- Analytics rule disabled after the first incident: [yes or no]
- Automation rule disabled after the test: [yes or no]
- IP used in the query: [expect `203.0.113.50` unless you document a different lab IP and redact it in screenshots]

## 5. Results

Write only what the portal and the Logic App runs showed. If a step failed, say it failed and paste the error text after you remove secrets, tenant ids, emails, and IPs.

- Incident number:
- Incident title:
- Created time (UTC):
- Severity on the new incident:
- Severity after the assign playbook:
- Status after the assign playbook:
- Owner after the assign playbook: [describe it without a raw email if the report is public]
- Enrichment comment summary: [score and report counts from the comment, or the failure line]
- Teams card: [arrived or did not arrive]
- `pb-lab-ip-enrichment` run:
- `pb-lab-assign-severity` run:
- `pb-lab-teams-card` run:
- Did Sentinel start the Teams playbook before the other runs finished: [yes, no, or not sure]

## 6. Containment drill

Optional. Only for `lab-phish-test`. If you did not do it, write "not done".

- Revoke sessions:
- Password reset, without the password:
- Block sign-in:
- Second revoke after the password reset:
- Sign-in you observed afterwards, with the IP redacted:
- Account re-enabled:

## 7. Lessons learned

- What you would change in the automation rule:
- What you would change in the playbooks:
- What you would not automate:
- What broke, and how you noticed:
- What you still want to practise:

## 8. Personal information

Did this lab process personal information of anyone other than you? [Expected answer if you followed the guide: No.]

POPIA reminder: in a real incident, personal information of people in South Africa can trigger a duty to notify the Information Regulator and the data subjects under section 22 of the Protection of Personal Information Act 4 of 2013. This report is not a notification, and this lab is not that incident. See the runbook for the note you would escalate in a job.

## 9. Evidence index

| Checklist file | Added | Notes |
| --- | --- | --- |
| `evidence/01-workspace-sentinel.png` | [yes or no] | |
| `evidence/02-key-vault-secret-name.png` | | |
| `evidence/03-ip-enrichment-designer.png` | | |
| `evidence/04-assign-designer.png` | | |
| `evidence/05-teams-designer.png` | | |
| `evidence/06-responder-role.png` | | |
| `evidence/07-key-vault-role.png` | | |
| `evidence/08-automation-rule.png` | | |
| `evidence/09-playbook-permission.png` | | |
| `evidence/10-analytics-rule-disabled.png` | | |
| `evidence/11-test-incident.png` | | |
| `evidence/12-enrichment-comment.png` | | |
| `evidence/13-owner-and-severity.png` | | |
| `evidence/14-teams-card.png` | | |
| `evidence/15-logic-app-runs.png` | | |
| `evidence/16-containment-test-user.png` | | |
| `evidence/17-rules-disabled.png` | | |

## 10. Cleanup

- Analytics rule disabled or deleted:
- Automation rule disabled or deleted:
- Logic Apps deleted or kept, and why:
- API connections deleted or kept:
- Key Vault deleted, soft deleted, or kept:
- AbuseIPDB key revoked or kept:
- Test user re-enabled, left blocked, or deleted:
- Budget or Cost Management checked on this date:
