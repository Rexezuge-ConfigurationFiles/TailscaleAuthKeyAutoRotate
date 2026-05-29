# Tailscale Auth Key Auto Rotate

Automatically rotate Tailscale device auth keys and securely store them in [Cloudflare Secrets Store](https://developers.cloudflare.com/workers/runtime-apis/secrets-store/).

---

## Overview

This repository contains GitHub Actions workflows that periodically generate a new Tailscale auth key and update it in Cloudflare Secrets Store. This helps maintain secure, short-lived credentials for Tailscale devices without manual intervention.

## Features

- **Automated key rotation** on a scheduled basis (1st and 15th of each month)
- **Manual triggering** via `workflow_dispatch` for on-demand rotation
- **Secure secret handling** with masked outputs and Cloudflare Secrets Store
- **Repository backup** to Azure DevOps on every push to `main`
- **Scheduled version updates** to keep the repository active

## Workflows

| Workflow | File | Trigger | Description |
|----------|------|---------|-------------|
| **Rotate Tailscale Auth Key** | `auth-key-rotate.yml` | Schedule / Manual | Creates a new Tailscale auth key and writes it to Cloudflare Secrets Store |
| **Backup to Azure DevOps** | `azure-devops-backup.yml` | Push to `main` | Mirrors the `main` branch to an Azure DevOps repository |
| **Scheduled Version Update** | `scheduled-version-update.yml` | Schedule | Writes a Unix timestamp to `.github/.version` to keep the repository active |

## How It Works

### 1. Authentication
The workflow authenticates with Tailscale using an OAuth client (with `auth_keys` scope) to obtain a short-lived access token.

### 2. Create Auth Key
A new Tailscale auth key is created with the following properties:
- **Reusable**: Can be used to authenticate multiple devices
- **Ephemeral**: Devices are automatically removed from the tailnet when they go offline
- **Pre-authorized**: Devices skip admin approval
- **Tagged**: Assigned to a specific tag (configured via `TS_TAG`)
- **Expiry**: ~31 days (2,700,000 seconds)

### 3. Store in Cloudflare
The new auth key is written to Cloudflare Secrets Store using the Wrangler CLI, overwriting the previous secret value.

## Setup

### Prerequisites

- A Tailscale account with OAuth clients enabled
- A Cloudflare account with Secrets Store configured
- (Optional) An Azure DevOps repository for backups

### Required GitHub Variables

| Variable | Description |
|----------|-------------|
| `TS_TAG` | Tailscale tag to assign to created auth keys (e.g., `tag:server`) |

### Required GitHub Secrets

| Secret | Description |
|--------|-------------|
| `TS_OAUTH_CLIENT_ID` | Tailscale OAuth client ID |
| `TS_OAUTH_CLIENT_SECRET` | Tailscale OAuth client secret |
| `CLOUDFLARE_ACCOUNT_ID` | Cloudflare account ID |
| `CLOUDFLARE_API_TOKEN` | Cloudflare API token with Secrets Store permissions |
| `CF_STORE_ID` | Cloudflare Secrets Store ID |
| `CF_SECRET_ID` | ID of the secret entry to update in the Secrets Store |
| `BACKUP_REPO_AZURE_DEVOPS_REPO_URL` | SSH URL of the Azure DevOps backup repository |
| `BACKUP_REPO_AZURE_DEVOPS_SSH_KEY` | SSH private key for pushing to Azure DevOps |

### Finding Your Cloudflare Secret ID

After configuring Wrangler, run:

```bash
wrangler secrets-store secret list "$CF_STORE_ID" --remote
```

## License

This project is licensed under the [MIT License](LICENSE).
