# Assign and set severity

`azuredeploy.json` deploys a Consumption Logic App named `pb-lab-assign-severity`.

On incident creation it updates the incident:

- Owner: the lab analyst object id, display name, mail, and user principal name you pass as parameters
- Severity: `High` unless you change `targetSeverity`
- Status: `Active` unless you change `targetStatus`

It then adds a short comment that states what it requested. It does not close the incident. It does not contain a sample incident or a sample owner.

Use your lab analyst account as the owner. Use a different, non admin test user for the containment drill in the runbook. Do not point this playbook at a user in a tenant you do not control.

## Parameters

| Name | Required | Default |
| --- | --- | --- |
| `playbookName` | no | `pb-lab-assign-severity` |
| `assigneeObjectId` | yes | none |
| `assigneeDisplayName` | yes | none |
| `assigneeEmail` | yes | none. Use the user principal name if Mail is empty |
| `assigneeUserPrincipalName` | yes | none |
| `targetSeverity` | no | `High` (`Informational`, `Low`, `Medium`, or `High`) |
| `targetStatus` | no | `Active` (`New` or `Active`) |

The test analytics rule creates an Informational incident, so the default severity makes a visible change. Record the before and after values from the incident itself.

## After deployment

Grant the system assigned identity **Microsoft Sentinel Responder** on the Sentinel resource group.

If the update action fails, the playbook adds a comment and stops. Open the run, then set the owner in the designer if the service rejected the owner object. Do not paste the run payload into git.

Object ids and email addresses are identifiers. Redact them in screenshots. See [evidence/README.md](../../evidence/README.md).
