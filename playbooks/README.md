# Playbooks

Three Consumption Logic Apps for the lab Sentinel workspace. Each folder has an ARM template that deploys with parameters. None of the files contain an API key, a password, or a result from a run.

| Folder | Logic App name | What it does |
| --- | --- | --- |
| `ip-enrichment` | `pb-lab-ip-enrichment` | Reads IP entities, checks AbuseIPDB, adds one incident comment |
| `assign-and-set-severity` | `pb-lab-assign-severity` | Assigns the incident and sets severity and status |
| `teams-adaptive-card` | `pb-lab-teams-card` | Posts an adaptive card to a Teams channel |

Deploy them into the same resource group as the Sentinel workspace from [Sentinel-Home-SOC-Lab](https://github.com/Sasa021/Sentinel-Home-SOC-Lab). Full click by click steps, cost warnings, and role assignments are in [docs/lab-guide.md](../docs/lab-guide.md).

The incident trigger does not poll. A run starts when an automation rule calls the playbook. Keep that rule filtered to the lab incident title. See [automation-rules/README.md](../automation-rules/README.md).

Logic Apps on the Consumption plan are billed per action. Delete the Logic Apps and their API connections when the lab is finished if you do not want more runs.
