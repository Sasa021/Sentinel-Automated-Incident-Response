# Screenshot checklist

Take these yourself after the lab steps. Save them under `evidence/` with the file names below. The repo does not include images.

Read [evidence/README.md](../evidence/README.md) before you commit. Cover IP addresses, tenant IDs, email addresses, and secrets. Also cover subscription IDs, object IDs, and the browser address bar.

Skip a shot if publishing it would reveal a secret. Mark the row in your report as "not taken" and say why.

1. `evidence/01-workspace-sentinel.png`
   Microsoft Sentinel open on the workspace you already use for Sentinel-Home-SOC-Lab. The workspace name is readable. The subscription id and tenant id are covered.

2. `evidence/02-key-vault-secret-name.png`
   Key Vault, Secrets, showing the name `abuseipdb-api-key` and that it is enabled. The secret value is not on screen. Do not open the current version if the value would be displayed.

3. `evidence/03-ip-enrichment-designer.png`
   Logic App designer for `pb-lab-ip-enrichment`. The incident trigger, the action that gets IP entities, the Key Vault get secret action, the HTTP check, and the comment action are visible. Parameters do not show a key.

4. `evidence/04-assign-designer.png`
   Designer for `pb-lab-assign-severity`. The update incident action is visible. Cover the analyst email and object id if the action body shows them.

5. `evidence/05-teams-designer.png`
   Designer for `pb-lab-teams-card`. The compose card action and the post card action are visible. The Teams connection shows Connected. No webhook URL is on screen.

6. `evidence/06-responder-role.png`
   Role assignment for the enrichment or assign playbook identity. The role name Microsoft Sentinel Responder is readable. Cover the principal id and your email.

7. `evidence/07-key-vault-role.png`
   Key Vault Secrets User assigned to the enrichment playbook identity, scoped to the vault. Cover object ids.

8. `evidence/08-automation-rule.png`
   Automation rule `LAB incident creation SOAR`. Show the title condition, the three playbook actions in order, and whether the rule is enabled or disabled at the time of the shot. Say which state it was in when you took it.

9. `evidence/09-playbook-permission.png`
   Resource group IAM, or the Sentinel playbook permissions blade, showing Microsoft Sentinel Automation Contributor for Azure Security Insights. Cover extra principals that are not part of the lab.

10. `evidence/10-analytics-rule-disabled.png`
    Analytics rule `LAB-SOAR-Test-Incident` after the test, with Status Disabled. Include enough of the query to show `print` and `203.0.113.50`, or describe those settings in the report if the screenshot would also show an email address.

11. `evidence/11-test-incident.png`
    The test incident overview: number, title, and severity. Cover the address bar. Show the lab incident from this test.

12. `evidence/12-enrichment-comment.png`
    The incident comment written by `pb-lab-ip-enrichment`. The text must be the text from your run. If the comment contains an IP other than the documentation address, cover the IP. No API key is visible.

13. `evidence/13-owner-and-severity.png`
    Incident owner and severity after `pb-lab-assign-severity`. Cover the owner email and object id. The display name can stay if you are comfortable publishing it. If you are not, cover it and describe the change in the report without the name.

14. `evidence/14-teams-card.png`
    The adaptive card in the lab Teams channel. Crop the member list and any tenant id in the incident link.

15. `evidence/15-logic-app-runs.png`
    Run history for the three playbooks for this one incident, showing the status of each run. Do not expand the HTTP action or the Key Vault action.

16. `evidence/16-containment-test-user.png`
    Optional. Only if you did the containment drill. The Entra user blade for `lab-phish-test` after block sign-in or revoke sessions. It must not be your administrator account. Cover the user principal name if the repo is public.

17. `evidence/17-rules-disabled.png`
    Both the analytics rule and the automation rule set to Disabled after the test. One picture is enough if both statuses are readable. Otherwise take two and note the extra file name in the report.
