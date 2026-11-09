# IP enrichment playbook

`azuredeploy.json` deploys a Consumption Logic App named `pb-lab-ip-enrichment`.

On incident creation it:

1. Reads IP entities from the incident.
2. If there is no IP, it comments on the incident and does not call AbuseIPDB.
3. Reads the API key from Key Vault at run time. The key is not in this repo and is hidden in the run history.
4. Calls `GET https://api.abuseipdb.com/api/v2/check` for up to 5 addresses.
5. Adds one comment with the fields the API returned: `abuseConfidenceScore`, `totalReports`, `numDistinctUsers`, `countryCode`, `usageType`, `isp`, `domain`, `lastReportedAt`, and `isWhitelisted`.

Those fields are filled only when a run happens in your tenant. This template does not contain sample scores.

## Parameters

| Name | Required | Default | Notes |
| --- | --- | --- | --- |
| `playbookName` | no | `pb-lab-ip-enrichment` | Logic App name |
| `keyVaultName` | yes | none | Existing vault. Not the secret value |
| `secretName` | no | `abuseipdb-api-key` | Secret name only |
| `maxIps` | no | `5` | Hard cap is 5, to limit the free daily quota |
| `maxAgeInDays` | no | `90` | Passed to AbuseIPDB |

Copy `azuredeploy.parameters.example.json` to a file that matches `*.parameters.local.json` if you want a parameter file. That pattern is gitignored. Do not commit a filled copy.

## After deployment

Grant the system assigned identity:

- **Microsoft Sentinel Responder** on the Sentinel resource group, so it can add a comment.
- **Key Vault Secrets User** on the vault, so it can read the secret.

If the Sentinel or Key Vault connection shows unauthorized, open the action in the designer and create a connection with the system assigned managed identity. Steps are in the lab guide.

The HTTP action does not retry. A 429 from the free quota is recorded as a failed lookup in the comment, without the response body.

## VirusTotal free API option

AbuseIPDB is the path this template deploys. To use the VirusTotal free public API instead, change the HTTP action after deployment:

- Method `GET`
- URI `https://www.virustotal.com/api/v3/ip_addresses/` plus the IP entity
- Header `x-apikey` from a Key Vault secret
- Keep secure inputs turned on for that action
- Change the comment so it uses the VirusTotal fields you actually need, such as `last_analysis_stats`

Confirm the current free quota on the VirusTotal site before you test. When this guide was written, the public API was 4 requests per minute and 500 requests per day. Do not put the key in git, and do not buy a paid VirusTotal plan for this lab.

## Cost

AbuseIPDB's free plan is enough. Do not add a payment card. Confirm the daily check limit on your account. This playbook calls the API at most 5 times per incident.

The Logic App is billed per action on the Consumption plan. The Key Vault read is a small extra Azure charge. Delete both when you finish if you do not need them. Key Vault soft delete can keep the name reserved until the retention period ends.
