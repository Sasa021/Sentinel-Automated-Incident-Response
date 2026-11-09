# Teams adaptive card

`azuredeploy.json` deploys a Consumption Logic App named `pb-lab-teams-card`.

On incident creation it posts an adaptive card to one standard channel. The card shows the incident title, number, severity, status, and created time from the trigger payload, plus a link to the incident. It does not wait for a button press, and it does not close or edit the incident.

Severity on the card is the value at the moment Sentinel starts the playbook. The assign playbook may change severity a moment later. The incident blade is the place to confirm the final value.

## Parameters

| Name | Required | Default |
| --- | --- | --- |
| `playbookName` | no | `pb-lab-teams-card` |
| `teamId` | yes | none. The `groupId` from the channel link |
| `channelId` | yes | none. The channel id from the channel link |

There is no webhook URL in this template. The Teams connection uses a user sign in, which you authorize in the portal after deployment. That consent is stored by Azure as a connection. Do not export the connection and commit it.

## After deployment

1. Grant the system assigned identity **Microsoft Sentinel Reader** on the Sentinel resource group. This playbook does not update the incident, so Responder is not required for it.
2. Open the Teams API connection and choose Authorize. Sign in as a lab user who is a member of the team.
3. Confirm the Workflows app is allowed in the Teams admin center. The Teams connector posts as Flow bot and needs that app.
4. Use a standard channel. This connector does not post to private channels.

## Cost

Teams needs a work or school account in the lab tenant. A Microsoft 365 Developer Program sandbox is the free option if you are eligible. Read the current program terms. Sandboxes can expire. Do not buy a Microsoft 365 licence for this lab.

The Logic App is still billed per action. Delete it with the other playbooks when you finish.
