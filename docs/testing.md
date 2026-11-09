# Safe test incident

Use this procedure once, on the lab workspace from [Sentinel-Home-SOC-Lab](https://github.com/Sasa021/Sentinel-Home-SOC-Lab). It creates a synthetic incident with a `print` query. It does not send email, open a link, or run malware.

Finish [docs/lab-guide.md](lab-guide.md) through the automation rule before you enable anything here.

## Cost and safety

- The analytics rule can create a new incident every time it runs. Each incident can start three Logic Apps and up to five AbuseIPDB calls.
- Set suppression as written below, and disable the analytics rule as soon as one incident exists. Disable the automation rule when the test is over.
- Leave the IP as `203.0.113.50`. That address is reserved for documentation by RFC 5737. It is not a host you are attacking. AbuseIPDB may return a score, empty fields, or an error. Write down what you actually got.
- Do not replace that address with an IP you found on a blog, a person, or an employer network, and then describe the API response as an incident you investigated.
- If you later look up an IP from a sign-in to your own lab tenant, redact it before you commit notes or screenshots.
- Do not enable extra paid data connectors, Identity Protection, or Defender for Office 365 for this test.

Time: about 40 minutes, including up to 15 minutes of waiting for the schedule.

## 1. Confirm the guardrails

- Playbooks are deployed and the identities have the roles in [automation-rules/README.md](../automation-rules/README.md).
- The Teams connection status is Connected.
- The automation rule exists, the title condition is in place, and you know how to disable it.
- The automation rule is still disabled while you create the analytics rule.

## 2. Create the analytics rule

Microsoft Sentinel, Analytics, Create, Scheduled query rule.

General:

| Setting | Value |
| --- | --- |
| Name | `LAB-SOAR-Test-Incident` |
| Description | Lab only. Synthetic incident for the SOAR playbooks. Disable this rule after one incident. |
| Severity | Informational |
| Status | Disabled, until step 4 |
| Tactics | Initial Access |
| Techniques | T1566, if the picker is shown |

Set rule logic. Query:

```kusto
print TimeGenerated = now(),
      AccountCustomEntity = "lab-phish-test@<your-lab-domain>",
      IPCustomEntity = "203.0.113.50"
```

Replace `<your-lab-domain>` with the domain of the test user from the lab guide. Leave the IP line as written.

| Setting | Value |
| --- | --- |
| Run query every | 5 minutes |
| Lookup data from the last | 5 minutes |
| Alert threshold | Greater than 0 |
| Event grouping | Group all events into a single alert |
| Suppression | On. Stop running the query for 24 hours after an alert is generated |

Entity mapping:

| Entity type | Identifier | Value |
| --- | --- | --- |
| IP | Address | `IPCustomEntity` |
| Account | Name | `AccountCustomEntity` |

If Name is not in the account identifier list, choose UPN and keep the test user principal name in `AccountCustomEntity`.

Incident settings:

- Create incidents from alerts triggered by this analytics rule: on
- Alert grouping: off, so this alert gets its own incident

Automated response:

- Select `LAB incident creation SOAR` if it is listed
- If it is not listed, leave this empty and enable the automation rule in the Automation blade instead

Review and create. The rule status stays Disabled until the next step.

Run the same query once under Logs. You should see one row. If Logs returns an error, fix the query before you enable the rule. Do not paste a screenshot of Logs into git if the account name is a real address you do not want to publish. The test user principal name is lab data. Redact it anyway if the screenshot will be public.

## 3. Enable the automation rule only

Automation, open `LAB incident creation SOAR`, set it to Enabled, and save.

Read the condition again. It must still be: incident title contains `LAB-SOAR-Test-Incident`.

## 4. Enable the analytics rule and catch one incident

1. Open `LAB-SOAR-Test-Incident` and set Status to Enabled. Save.
2. Stay in the portal. Open Incidents and refresh.
3. When the first incident appears, open the analytics rule again and set Status to Disabled. Save.
4. Do this before you inspect the playbook runs. Suppression is a backstop. Disable the rule as well.

If nothing appears after 15 minutes:

- The analytics rule status is Enabled, and the workspace is the Sentinel workspace.
- The query returns a row in Logs.
- You did not create a second copy of the rule.
- The incident title contains `LAB-SOAR-Test-Incident`. If Sentinel changed the title, edit the automation rule condition to the exact title you see. Keep the condition specific to this lab rule.

Do not raise the schedule to every minute.

## 5. Record what the product showed

Fill these from the live incident and the Logic App runs. Leave them blank in git until you have done the test.

- Incident number:
- Incident title:
- Time created (UTC):
- Severity before you rely on the assign playbook:
- Severity after the assign playbook:
- Owner after the assign playbook:
- Did an enrichment comment appear, and did it include a score, empty fields, or a failure line:
- Teams card arrived (yes or no):
- Run status of `pb-lab-ip-enrichment`:
- Run status of `pb-lab-assign-severity`:
- Run status of `pb-lab-teams-card`:

Copy the same notes into [docs/report-template.md](report-template.md) after you copy that file to `docs/report.md`.

## 6. Turn the test off

- Analytics rule `LAB-SOAR-Test-Incident`: Disabled. Delete it during cleanup if you do not need the definition.
- Automation rule `LAB incident creation SOAR`: Disabled.
- Confirm no second incident was created. If one was, record the number and disable both rules again.

## If enrichment finds no IP

The incident comment will say that no IP entity was found, or that entities could not be read.

1. Open the enrichment run. In the trigger output, see where the IP sits. Redact the payload before any screenshot.
2. The template sends `object.properties.relatedEntities` into the action that gets IP entities. That matches the usual incident trigger.
3. If your run stores entities on a different array, point that action at the array that contains the IP, save, and run this test once more. Disable the analytics rule again as soon as that incident exists.
4. A second test still uses `203.0.113.50`.

## What this test does not prove

A green run proves the playbooks started and that you can read their output. It does not prove a phishing email was detected. The phishing response steps are the separate drill in [runbooks/phishing-account-takeover.md](../runbooks/phishing-account-takeover.md), and only against `lab-phish-test`.
