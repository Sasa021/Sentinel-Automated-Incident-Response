# Automation rule

One Sentinel automation rule starts the three playbooks when a lab incident is created. The rule must stay narrow. A rule with no conditions will run the playbooks, and the AbuseIPDB call, for every new incident in the workspace.

## Portal settings

Create the rule in Microsoft Sentinel, under Automation, after the playbooks are deployed and the Teams connection is authorized.

| Setting | Value |
| --- | --- |
| Name | `LAB incident creation SOAR` |
| Trigger | When incident is created |
| Condition | Incident title contains `LAB-SOAR-Test-Incident` |
| Action 1 | Run playbook `pb-lab-ip-enrichment` |
| Action 2 | Run playbook `pb-lab-assign-severity` |
| Action 3 | Run playbook `pb-lab-teams-card` |
| Order | `100` |
| Expiration | The day after you plan to finish the lab, if the field is shown |
| Enabled | Only during the test window in [docs/testing.md](../docs/testing.md) |

Sentinel starts the playbooks in that order. It does not wait for one Logic App run to finish before it starts the next. The Teams card can arrive before the comment or the severity change. Confirm both on the incident.

Do not delete the title condition to "make it fire". If a test incident title does not match, copy the title you see and update the condition so it still matches only that lab rule.

## Permission for Sentinel to run playbooks

Your user needs **Microsoft Sentinel Contributor** to attach a playbook to the rule.

Microsoft Sentinel uses its own service principal, **Azure Security Insights**, to start incident playbooks. That principal needs **Microsoft Sentinel Automation Contributor** on the resource group that contains the playbooks. The role is for that service principal. Do not assign it to your user as a substitute.

Grant it from the product blade:

1. Microsoft Sentinel, then Settings, then Playbook permissions, then Configure permissions.
2. Select the resource group that contains `pb-lab-ip-enrichment`, `pb-lab-assign-severity`, and `pb-lab-teams-card`.
3. Grant permission.

You need **Owner** or **User Access Administrator** on that resource group for the button to work. Contributor alone cannot assign roles.

Confirm on the resource group, Access control (IAM), that **Microsoft Sentinel Automation Contributor** is assigned to **Azure Security Insights**. The built in role definition id is `f4c81013-99ee-4d62-a7ee-b3f1f648599a`. Select the role by name in the portal.

## Managed identity and Microsoft Sentinel Responder

Each playbook has a system assigned managed identity. The ARM templates turn it on. The identity is what the Sentinel connector uses. It is separate from the Azure Security Insights principal above.

| Playbook | Role | Scope | Why |
| --- | --- | --- | --- |
| `pb-lab-ip-enrichment` | Microsoft Sentinel Responder | Sentinel resource group | Add a comment |
| `pb-lab-ip-enrichment` | Key Vault Secrets User | The Key Vault only | Read the API key secret |
| `pb-lab-assign-severity` | Microsoft Sentinel Responder | Sentinel resource group | Update owner, severity, and status, and add a comment |
| `pb-lab-teams-card` | Microsoft Sentinel Reader | Sentinel resource group | Receive the incident trigger. This playbook does not write to the incident |

**Microsoft Sentinel Responder** includes read access and incident write access (owner, severity, status, comments). The built in role definition id is `3e150937-b8fe-4cfb-8069-0eaf05ecd056`. Select the role by name.

Portal path for each Logic App:

1. Open the Logic App, then Identity, and confirm System assigned status is On.
2. Azure role assignments, then Add role assignment.
3. Scope: Resource group for the Sentinel roles. For the Key Vault role, set the scope to the vault.
4. Save.

Azure CLI equivalent, after you copy the principal id from the deployment outputs. Do not commit the command with real ids.

```bash
az role assignment create \
  --assignee-object-id "<principal-id>" \
  --assignee-principal-type ServicePrincipal \
  --role "Microsoft Sentinel Responder" \
  --scope "/subscriptions/<subscription-id>/resourceGroups/<sentinel-resource-group>"
```

```bash
az role assignment create \
  --assignee-object-id "<enrichment-principal-id>" \
  --assignee-principal-type ServicePrincipal \
  --role "Key Vault Secrets User" \
  --scope "/subscriptions/<subscription-id>/resourceGroups/<sentinel-resource-group>/providers/Microsoft.KeyVault/vaults/<vault-name>"
```

Use **Microsoft Sentinel Reader** in the first command for `pb-lab-teams-card`.

If a Sentinel connection still shows unauthorized after the role assignment, open the trigger in the designer, add a connection, and choose Connect with managed identity, system assigned. Role assignment can take a few minutes to apply.

## Optional ARM template

`lab-incident-creation.json` deploys the same rule. `isEnabled` defaults to `false`, and the title filter defaults to `LAB-SOAR-Test-Incident`. Deploy it to the same resource group as the playbooks and the workspace. It does not create a workspace.

```bash
az deployment group create \
  --resource-group "<sentinel-resource-group>" \
  --template-file automation-rules/lab-incident-creation.json \
  --parameters workspaceName="<workspace-name>" isEnabled=false
```

Leave `isEnabled=false` until [docs/testing.md](../docs/testing.md) says to enable it. If the deployment returns an error about scope or api version, skip the file and use the portal settings above. Do not remove the title condition to get a successful wider rule.

The template does not grant Automation Contributor. Use the Playbook permissions button even if you deploy the JSON.
