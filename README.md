# LetsEncrypt ACME Cert Automation - SBX

Automates the issuance, validation, and deployment of Let's Encrypt wildcard SAN certificates to Azure Key Vault using [acme.sh](https://github.com/acmesh-official/acme.sh) with Azure DNS challenge.

---

## Overview

| Property       | Value                          |
|----------------|--------------------------------|
| Environment    | SBX (Sandbox)                  |
| Certificate    | Wildcard SAN (`*.sbx.yourdomain.com`) |
| CA             | Let's Encrypt                  |
| DNS Provider   | Azure DNS                      |
| Key Vault      | `your-keyvault-name`           |
| Schedule       | Monthly (1st of month, 02:00 UTC) |
| Runner         | Self-hosted                    |

---

## Workflow Triggers

- **Scheduled** — Runs automatically on the 1st of every month at 02:00 UTC
- **Manual** — Can be triggered anytime via `workflow_dispatch` from the GitHub Actions UI

---

## Prerequisites

### Self-Hosted Runner
The workflow runs on a self-hosted runner tagged `your-runner`. Ensure the runner has:
- `curl` installed
- `openssl` installed
- `az` (Azure CLI) installed and accessible

### Azure Service Principal
A service principal with the following permissions:
- **DNS Zone Contributor** on the Azure DNS zone for `sbx.yourdomain.com`
- **Key Vault Certificates Officer** on `your-keyvault-name`

### GitHub Secrets

| Secret Name             | Description                        |
|-------------------------|------------------------------------|
| `AZURE_TENANT_ID`       | Azure Active Directory Tenant ID   |
| `AZURE_CLIENT_ID`       | Service Principal Application ID   |
| `AZURE_CLIENT_SECRET`   | Service Principal Client Secret    |

---

## Workflow Steps

1. **Install acme.sh** — Installs acme.sh on the runner if not already present
2. **Issue SAN Cert** — Issues a wildcard cert for `*.sbx.yourdomain.com` using Azure DNS-01 challenge with 4096-bit key
3. **Convert to PFX** — Exports the cert and private key into a passwordless PFX file
4. **Validate Certificate Chain** — Verifies the cert is:
   - Not expired
   - Matches the expected domain
   - Contains a valid private key
5. **Login to Azure** — Authenticates using the service principal
6. **Push to Key Vault** — Imports the PFX into Azure Key Vault under the name `wildcard-san-cert`
7. **Cleanup** — Removes the local PFX file and logs out of Azure CLI

---

## Certificate Details

| Property       | Value                  |
|----------------|------------------------|
| Type           | Wildcard SAN           |
| Domain         | `*.sbx.yourdomain.com` |
| Key Length     | 4096-bit RSA           |
| Format         | PFX (PKCS#12)          |
| Password       | None (passwordless)    |

---

## Customization

To adapt this workflow for another environment (e.g., `dev`, `prod`), update:
- `-d "*.sbx.yourdomain.com"` — change the domain
- `--vault-name your-keyvault-name` — change the Key Vault name
- `AZUREDNS_SUBSCRIPTIONID` — update the subscription ID
- `email=cloudservices@yourdomain.com` — update the ACME registration email

---

## Troubleshooting

| Issue | Resolution |
|-------|------------|
| `remote: Not Found` on cert issue | Verify Azure DNS credentials and DNS zone name |
| `FAIL - Cert is expired or invalid` | Check acme.sh logs at `/tmp/acme.log` on the runner |
| `FAIL - Cert push failed` | Verify service principal has Key Vault Certificates Officer role |
| acme.sh not found | Ensure runner has internet access to `get.acme.sh` |
