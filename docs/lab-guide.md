# Lab guide

Build the three Sentinel playbooks, the automation rule, and one safe test incident on the workspace you already created for [Sentinel-Home-SOC-Lab](https://github.com/Sasa021/Sentinel-Home-SOC-Lab).

Plan on about 5 to 6 hours. The scheduled rule can add up to 15 minutes of waiting. Do the test in the same sitting so the analytics rule is not left enabled overnight.

| Step | Time |
| --- | --- |
| Read cost and safety, confirm the workspace | 30 minutes |
| Budget alert | 10 minutes |
| Test user | 15 minutes |
| Teams channel and ids | 20 minutes |
| AbuseIPDB account and Key Vault | 30 minutes |
| Deploy IP enrichment and roles | 35 minutes |
| Deploy assign playbook and role | 25 minutes |
| Deploy Teams playbook and authorize | 35 minutes |
| Automation rule and playbook permission | 25 minutes |
| Safe test, [testing.md](testing.md) | 40 minutes |
| Screenshots | 30 minutes |
| Report | 30 minutes |
| Cleanup | 20 minutes |

## Cost, safety, and cleanup

Read this before you create anything.

This lab uses your own Azure subscription and your own tenant. Do not connect an employer tenant, a customer tenant, or real mail. Do not paste production logs into the workspace.

Charges you can incur:

- **Logic Apps, Consumption.** Billed per action. One test is small. A rule that fires all day is not. Check current prices in the portal before you deploy, then check Cost Management after the test.
- **Key Vault, Standard.** Billed per operation. One secret is a small charge. There is no separate free vault product. Soft delete keeps a deleted vault for the retention period you set, and the name stays reserved until that ends or you purge it.
- **Microsoft Sentinel and Log Analytics.** Reuse the existing workspace. A second workspace is a second bill. The test query uses `print` and does not ingest a file. Sign-in and audit logs, if you turn them on for the runbook, do count as ingestion. Do not enable extra connectors that show a price.
- **AbuseIPDB.** Use the free plan. Do not add a payment card. Confirm the daily check limit on the account page. This playbook calls the API at most 5 times per incident.
- **VirusTotal.** Optional swap only. Stay on the free public API and confirm the current quota. Do not buy a plan.
- **Teams.** Needs a work or school account in this tenant. If you have no Teams, the Microsoft 365 Developer Program is the free route when Microsoft accepts you. Read the current terms. A sandbox can expire. Do not buy a Microsoft 365 licence for this lab.

Set a budget you are willing to spend, with an alert before it is used up. Azure portal, Cost Management, Budgets. The amount is your choice. The portal shows prices in the billing currency of the subscription.

Cleanup when you are finished is [at the end of this guide](#cleanup). The short version: disable both rules, delete the three Logic Apps and their API connections, delete the vault if you created it only for this lab, revoke the AbuseIPDB key if you do not need it, and re-enable or delete `lab-phish-test`.

Never commit an API key, a password, a connection export, or an unredacted screenshot. See [evidence/README.md](../evidence/README.md).

## Prerequisites

- The Sentinel workspace from Sentinel-Home-SOC-Lab, in a subscription you control
- A work or school account in that tenant. A personal Microsoft account is not enough for Sentinel
- **Owner**, or **Contributor** plus **User Access Administrator**, on the Sentinel resource group. Contributor alone cannot grant the managed identity roles
- Permission to create Logic Apps, API connections, and a Key Vault in that resource group
- A browser. Azure CLI is optional and is shown beside the portal steps
- A password manager for the test user password and for nowhere else in this repo

You do not need a paid Sentinel commitment beyond the workspace you already have. You do not need a new resource group if the workspace resource group can host Logic Apps.

## 1. Confirm the workspace

Time: 30 minutes.

1. Azure portal, Microsoft Sentinel, select the existing workspace.
2. Write down the workspace name, the resource group name, and the region. Keep the subscription id and tenant id in your private notes.
3. Confirm you are not about to create another Sentinel or another Log Analytics workspace.

Screenshot checkpoint: `evidence/01-workspace-sentinel.png`. Cover the subscription id and tenant id.

If the resource group region does not support Consumption Logic Apps, deployment fails with a location error. Prefer fixing that by deploying the playbooks into the same group only when the region supports them. A second group in a supported region in the same subscription is a fallback. If you use a second group, grant Microsoft Sentinel Automation Contributor on that group, and assign Microsoft Sentinel Responder on the group that holds the workspace. The main path below assumes one resource group.

## 2. Set a budget alert

Time: 10 minutes.

Cost Management, Budgets, add a monthly budget at an amount you choose, alert before the full amount. This does not stop spend by itself. It tells you to look.

## 3. Create the test user

Time: 15 minutes.

This user is the containment target in the runbook. It is not the account that owns incidents, and it is not your administrator.

Entra admin center, `https://entra.microsoft.com`, Identity, Users, New user, Create new user.

| Field | Value |
| --- | --- |
| User principal name | `lab-phish-test` at your lab domain |
| Display name | `Lab Phish Test` |
| Password | Generate one and store it in a password manager |
| Roles | None |
| Groups | None |
| Block sign-in | No |

Open the new user and confirm Directory roles is empty.

If you are the only Global Administrator, stop and re-read that. You will not block sign-in on your administrator during this project.

The assign playbook should target your lab analyst, which can be the administrator account you are using. Copy that account's object id, display name, user principal name, and mail. If Mail is empty, use the user principal name for the email parameter. Keep those values in private notes.

## 4. Create the Teams channel and collect ids

Time: 20 minutes.

Use a team in this same lab tenant. Create a standard channel named `lab-sentinel-alerts`. Do not use a private channel. The connector cannot post there.

Get the ids:

1. In Teams, open the channel, then Get link to channel.
2. The link contains `/channel/CHANNEL_ID/` and a query parameter `groupId=TEAM_ID`.
3. `teamId` is the `groupId` value.
4. `channelId` is the channel segment. If you see `%3a` and `%40`, replace them with `:` and `@`. The id often ends with `@thread.tacv2`.
5. Drop the `tenantId` parameter. Do not put it in a parameter file you might commit.

Teams admin center, Teams apps, Manage apps, search **Workflows**, status **Allowed**. The post action uses Flow bot and fails when that app is blocked. In a lab tenant you are the admin. If you are not, and the later run returns a bot error, that setting is the first check.

If you have no Teams licence, join the Microsoft 365 Developer Program with the account that owns this tenant, or use a developer sandbox you already have. Do not buy a licence. Do not point the playbook at a channel in a tenant you do not control.

## 5. Put the AbuseIPDB key in Key Vault

Time: 30 minutes.

### AbuseIPDB

1. Register at [https://www.abuseipdb.com/register](https://www.abuseipdb.com/register) with your own email.
2. Stay on the free plan. Do not enter a card.
3. Create an API key. Read the daily limit on the page and write that number in your report later.
4. Copy the key straight into Key Vault in the next minute. Then close the page that shows the key.

### Key Vault

Create a resource, Key Vault, in the Sentinel resource group.

| Setting | Value |
| --- | --- |
| Name | A name you choose, 3 to 24 characters, letters and numbers, starting with a letter. It must be unique in Azure |
| Region | Same as the resource group |
| Pricing tier | Standard |
| Permission model | Azure role-based access control |
| Soft-delete | On |
| Retention days | The smallest number the portal allows |
| Purge protection | Disable. If you turn it on, you cannot turn it off, and you cannot purge the vault until retention ends |
| Public access | Allow, for this lab vault only |

Create the vault. Then Objects, Secrets, Generate/Import.

| Setting | Value |
| --- | --- |
| Name | `abuseipdb-api-key` |
| Secret value | The key you just copied |
| Enabled | Yes |
| Expiration | Optional. 90 days is a reasonable backstop |

Do not download the secret as a file into this repo. Do not put the value in `azuredeploy.parameters.example.json`.

Your user needs a role that can create secrets. On a new RBAC vault, **Key Vault Administrator** on the vault is enough for your user account. The playbook gets **Key Vault Secrets User** in the next step. That role can read secrets and cannot change vault settings.

Screenshot checkpoint: `evidence/02-key-vault-secret-name.png`. Show the secret name. Do not open the secret value.

## 6. Deploy the IP enrichment playbook

Time: 35 minutes.

Template: `playbooks/ip-enrichment/azuredeploy.json`.

Portal:

1. Search **Deploy a custom template**.
2. Build your own template in the editor, Load file, select `azuredeploy.json`, Save.
3. Subscription: your lab subscription. Resource group: the Sentinel resource group.
4. Parameters:

| Parameter | Value |
| --- | --- |
| Playbook name | `pb-lab-ip-enrichment` |
| Key vault name | The vault name from step 5 |
| Secret name | `abuseipdb-api-key` |
| Max IPs | `5` |
| Max age in days | `90` |

5. Review and create.
6. Open Outputs. Copy `playbookResourceId` and `managedIdentityPrincipalId` into private notes.

CLI, from a clone of this repo. The command does not take the key.

```bash
az deployment group create \
  --resource-group "<sentinel-resource-group>" \
  --template-file playbooks/ip-enrichment/azuredeploy.json \
  --parameters keyVaultName="<your-vault-name>" secretName=abuseipdb-api-key maxIps=5 maxAgeInDays=90
```

### Roles for this identity

Logic App, Identity, System assigned, status On.

Add role assignment:

| Role | Scope |
| --- | --- |
| Microsoft Sentinel Responder | Sentinel resource group |
| Key Vault Secrets User | This Key Vault only |

Select the roles by name. Responder's built in role id is `3e150937-b8fe-4cfb-8069-0eaf05ecd056`.

Wait a few minutes, then open the Logic App designer. If the Sentinel trigger or the Key Vault action says the connection is unauthorized, open that action, add a connection, and choose **Connect with managed identity**, system assigned, Create.

Screenshot checkpoints: `evidence/03-ip-enrichment-designer.png`, `evidence/06-responder-role.png`, `evidence/07-key-vault-role.png`.

## 7. Deploy the assign playbook

Time: 25 minutes.

Template: `playbooks/assign-and-set-severity/azuredeploy.json`.

Same deploy path. Parameters:

| Parameter | Value |
| --- | --- |
| Playbook name | `pb-lab-assign-severity` |
| Assignee object id | Object id of your lab analyst |
| Assignee display name | Display name of that account |
| Assignee email | Mail, or the user principal name if Mail is empty |
| Assignee user principal name | User principal name |
| Target severity | `High` |
| Target status | `Active` |

`High` is deliberate. The test incident is created as Informational, so you can see a change. Pick another allowed value if you want. Record the value you deployed.

Grant **Microsoft Sentinel Responder** on the Sentinel resource group to this playbook's identity. Fix the Sentinel connection with managed identity if it shows unauthorized.

This playbook does not close incidents.

Screenshot checkpoint: `evidence/04-assign-designer.png`. Cover the email and object id.

CLI shape:

```bash
az deployment group create \
  --resource-group "<sentinel-resource-group>" \
  --template-file playbooks/assign-and-set-severity/azuredeploy.json \
  --parameters \
      assigneeObjectId="<object-id>" \
      assigneeDisplayName="<display-name>" \
      assigneeEmail="<upn-or-mail>" \
      assigneeUserPrincipalName="<upn>" \
      targetSeverity=High \
      targetStatus=Active
```

Do not commit the shell history if it contains those values.

## 8. Deploy the Teams playbook

Time: 35 minutes.

Template: `playbooks/teams-adaptive-card/azuredeploy.json`.

| Parameter | Value |
| --- | --- |
| Playbook name | `pb-lab-teams-card` |
| Team id | `groupId` from step 4 |
| Channel id | Channel id from step 4 |

Grant **Microsoft Sentinel Reader** on the Sentinel resource group. This playbook does not update the incident. Reader is enough for the trigger. Responder is the role for the other two playbooks.

Authorize Teams:

1. Open the Logic App, API connections, or open the Teams connection resource created next to it (name starts with `teams-`).
2. Edit API connection, Authorize, sign in as a user who is a member of the team.
3. Status should become Connected.
4. Open the designer and confirm the post action still has your team id and channel id.

There is no webhook secret to copy. Do not export the connection into the repo.

Screenshot checkpoint: `evidence/05-teams-designer.png`.

## 9. Create the automation rule

Time: 25 minutes.

### Let Sentinel run playbooks

Microsoft Sentinel, Settings, Playbook permissions, Configure permissions. Select the resource group that contains the three Logic Apps. Grant permission.

You need Owner or User Access Administrator. The grant assigns **Microsoft Sentinel Automation Contributor** to the **Azure Security Insights** service principal. The role id is `f4c81013-99ee-4d62-a7ee-b3f1f648599a`. Assign it to that service principal.

Confirm it under the resource group, Access control (IAM).

Screenshot checkpoint: `evidence/09-playbook-permission.png`.

### Create the rule

Automation, Create, Automation rule.

| Setting | Value |
| --- | --- |
| Name | `LAB incident creation SOAR` |
| Trigger | When incident is created |
| Condition | Incident title contains `LAB-SOAR-Test-Incident` |
| Action order 1 | Run playbook `pb-lab-ip-enrichment` |
| Action order 2 | Run playbook `pb-lab-assign-severity` |
| Action order 3 | Run playbook `pb-lab-teams-card` |
| Order | `100` |
| Expiration | The day after you intend to finish, if the field is shown |
| Status | Disabled until [testing.md](testing.md) tells you to enable it |

Leave the title condition in place. Sentinel starts the three playbooks in order and does not wait for each Logic App to finish. The card can show the original Informational severity. The incident blade shows the later update.

If a playbook is missing from the list, check that it is in the same subscription, that the trigger is the Sentinel incident trigger, and that your user has Microsoft Sentinel Contributor.

Optional template: `automation-rules/lab-incident-creation.json` deploys the same rule with `isEnabled` defaulting to false. Portal steps above are the ones to use if the template returns an error. Details are in [automation-rules/README.md](../automation-rules/README.md).

Screenshot checkpoint: `evidence/08-automation-rule.png`.

## 10. Run one safe test

Time: 40 minutes.

Do [testing.md](testing.md) now. The analytics rule name is `LAB-SOAR-Test-Incident`, severity Informational, query is a single `print` row, IP `203.0.113.50`, suppression 24 hours, and you disable the analytics rule as soon as one incident exists.

Do not skip the disable step. Come back here only after both rules are disabled.

Screenshot checkpoints from the test: `evidence/10-analytics-rule-disabled.png`, `evidence/11-test-incident.png`, `evidence/12-enrichment-comment.png`, `evidence/13-owner-and-severity.png`, `evidence/14-teams-card.png`, `evidence/15-logic-app-runs.png`, `evidence/17-rules-disabled.png`.

## 11. Optional containment drill

Time: about 60 minutes if you include the queries.

Walk [runbooks/phishing-account-takeover.md](../runbooks/phishing-account-takeover.md) against `lab-phish-test` only. Screenshot `evidence/16-containment-test-user.png` only if you do this, and only if the blade is the test user.

Re-enable the test user before you finish, unless you have deleted it on purpose.

## 12. Write the report

Time: 30 minutes.

Copy `docs/report-template.md` to `docs/report.md`. Fill it from the incident, the runs, and the comment. If enrichment returned an error or an empty score for `203.0.113.50`, write that. Do not replace it with a number from someone else's report.

Update the Results and Lessons learned sections in the root README when you are ready for them to be public.

## Troubleshooting

**Sentinel connection unauthorized.** Confirm the identity has Responder or Reader, wait five minutes, then recreate the connection with system assigned managed identity.

**Key Vault 403.** The enrichment identity needs Key Vault Secrets User on that vault. Your own Key Vault Administrator role does not help the playbook.

**AbuseIPDB 401.** The secret value is wrong or disabled. Create a new key, put it in a new secret version, and do not turn off secure inputs to debug. The header would then sit in the run history.

**AbuseIPDB 429.** You hit the free daily limit. Stop the test. Do not upgrade to a paid plan to finish the lab.

**Comment says no IP entities.** The test rule's IP mapping must be entity type IP, identifier Address, value `IPCustomEntity`. If the run shows entities on a different array, follow the note in [testing.md](testing.md).

**Playbook not listed on the rule.** Same subscription, incident trigger present, your user has Microsoft Sentinel Contributor, and the playbook permission grant succeeded.

**Teams post fails.** Standard channel, Workflows app allowed, connection authorized by a member of the team, team id and channel id copied from the channel link.

**Many incidents appeared.** Disable `LAB-SOAR-Test-Incident` and the automation rule immediately. Record how many incidents were created. Do not delete them until you have the incident number you are writing up, then you can close the extras as benign or delete the lab rule's incidents if the portal allows it. Closing is a manual step. Do not add a playbook that closes every incident.

## Cleanup

Do this when the report is drafted.

1. Analytics rule `LAB-SOAR-Test-Incident`: Disabled, then delete if you do not need the definition.
2. Automation rule `LAB incident creation SOAR`: Disabled, then delete if you do not need it.
3. Delete the three Logic Apps: `pb-lab-ip-enrichment`, `pb-lab-assign-severity`, `pb-lab-teams-card`.
4. Delete the API connections left behind (names start with `azuresentinel-`, `keyvault-`, and `teams-`).
5. Delete the Key Vault if it exists only for this lab. Because soft delete is on, also purge it if you need the name, and only if purge protection is off. Portal: Key Vault, Manage deleted vaults.
6. Revoke the AbuseIPDB API key on the AbuseIPDB account if you are done. Revoke a VirusTotal key too if you created one.
7. Re-enable `lab-phish-test`, or delete that user if the drill is over. Do not delete your administrator.
8. If you enabled Entra sign-in or audit ingestion only for this lab, turn those categories off.
9. Cost Management: confirm you do not still have a second workspace, a Logic App on the Standard plan, or a connector you did not mean to leave running.

Role assignments on a deleted identity are removed by Azure. You can delete leftover assignments in IAM if they still appear.

The Sentinel workspace from the home lab stays. This guide does not tell you to delete it.
