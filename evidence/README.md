# Evidence

This folder is where Mosa adds proof from her own lab. It is empty on purpose. Do not treat anything in the templates or the guides as a result from a real run.

Add the files named in [docs/screenshot-checklist.md](../docs/screenshot-checklist.md). Write-up notes go in `docs/report.md`. A short caption next to a screenshot is fine.

## Redact before you commit

Check every screenshot and every text file before `git add`. Remove or cover:

- IP addresses
- Tenant IDs
- Email addresses
- Secrets

Secrets include API keys, passwords, tokens, client secrets, connection strings, Shared Access Signatures, and webhook URLs.

Also cover these if they are visible:

- Subscription IDs
- User object IDs
- The full incident URL when it contains a tenant id
- The browser address bar

Do not commit a picture of a Key Vault secret value, an AbuseIPDB or VirusTotal page that shows the key, or a Logic App run with the HTTP `Key` or `x-apikey` header visible. The enrichment playbook hides that header in the run history. If a run still shows it, leave that step collapsed and do not screenshot it.

If a screenshot shows personal information of anyone other than you, do not commit it. This lab uses your own tenant and your own test user only.

A documentation address such as `203.0.113.50` is still an IP address. Crop it if you are unsure, or leave it only when your report says it is the RFC 5737 address from the test procedure.
