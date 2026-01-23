# Infrastructure Setup

This document describes the Azure infrastructure for FoodByTheBook, managed with Terraform.

**🔒 Secrets Management:** This document is safe to commit to public Git repositories. All sensitive information (passwords, API keys, connection strings) is stored separately in local files and GitHub Secrets. See "Secrets Management Summary" section below.

---

## Secrets Management Summary

**Problem:** Infrastructure needs secrets (passwords, API keys) but these must NEVER be in Git.

**Solution:** Separate configuration from secrets

| File | Contains | Safe to Commit? | Where Stored |
|------|----------|-----------------|--------------|
| `infrastructure/main.tf` | Infrastructure code | ✅ Yes | Git repository |
| `infrastructure/terraform.tfvars.example` | Template with placeholders | ✅ Yes | Git repository |
| `infrastructure/terraform.tfvars` | **Actual secrets** | ❌ NEVER | Your local machine only |
| `frontend/.env.example` | Template with placeholders | ✅ Yes | Git repository |
| `frontend/.env.local` | **Actual values** | ❌ NEVER | Your local machine only |

**For CI/CD (GitHub Actions):**
- Secrets stored in: GitHub Repository Settings → Secrets and variables → Actions
- Never in workflow YAML files
- Injected as environment variables during deployment

**For Team Collaboration:**
- Share secrets via: Azure Key Vault, 1Password, Bitwarden, or secure channel
- Each developer creates their own local `terraform.tfvars` and `.env.local`
- Never share via email, Slack, or commit to Git

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                     Azure Subscription                       │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐ │
│  │         Resource Group: rg-foodbythebook               │ │
│  │                                                         │ │
│  │  ┌──────────────────────────────────────────────────┐  │ │
│  │  │  Azure Static Web App (Frontend)                 │  │ │
│  │  │  - Free tier                                      │  │ │
│  │  │  - Auto-deploy from GitHub                       │  │ │
│  │  │  - Custom domain support                         │  │ │
│  │  │  - Free SSL                                       │  │ │
│  │  └──────────────────────────────────────────────────┘  │ │
│  │                                                         │ │
│  │  ┌──────────────────────────────────────────────────┐  │ │
│  │  │  Azure Functions App (Backend)                   │  │ │
│  │  │  - Consumption plan (Free tier)                  │  │ │
│  │  │  - .NET 8 isolated worker                        │  │ │
│  │  │  - HTTP-triggered functions                      │  │ │
│  │  │  - CORS configured for frontend                  │  │ │
│  │  └──────────────────────────────────────────────────┘  │ │
│  │                                                         │ │
│  │  ┌──────────────────────────────────────────────────┐  │ │
│  │  │  Storage Account                                 │  │ │
│  │  │  - Required for Azure Functions                 │  │ │
│  │  │  - LRS replication                               │  │ │
│  │  └──────────────────────────────────────────────────┘  │ │
│  └─────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────┘

External Services (Not in Terraform):
┌──────────────────────────────────┐
│  Supabase                        │
│  - PostgreSQL database           │
│  - Storage (images)              │
│  - Managed externally            │
└──────────────────────────────────┘

┌──────────────────────────────────┐
│  Microsoft Entra ID              │
│  - OAuth 2.0 provider            │
│  - App registration              │
│  - Manual setup required         │
└──────────────────────────────────┘
```

## Prerequisites

**Important Note on Secrets:** This document is safe to commit to Git. All sensitive information (passwords, API keys, connection strings) is stored separately and never committed. See "Secrets Management Summary" below.

1. **Azure Account**
   - Free tier account: https://azure.microsoft.com/free/
   - Or existing subscription

2. **Terraform**
   - Install: https://www.terraform.io/downloads
   - Version: 1.0+
   ```bash
   # Windows (via Chocolatey)
   choco install terraform
   
   # Or download from terraform.io
   ```

3. **Azure CLI**
   - Install: https://docs.microsoft.com/cli/azure/install-azure-cli
   ```bash
   # Windows (via Chocolatey)
   choco install azure-cli
   
   # Login
   az login
   ```

4. **Supabase Account**
   - Sign up: https://supabase.com
   - Create project (free tier)
   - Note connection string and API keys

5. **Microsoft Entra ID App Registration**
   - Register app: https://portal.azure.com/#view/Microsoft_AAD_RegisteredApps
   - Note: Tenant ID, Client ID, Client Secret

## Terraform Configuration

### Directory Structure

```
/infrastructure
├── main.tf                    # Main resources definition
├── variables.tf               # Variable declarations
├── outputs.tf                 # Output definitions
├── provider.tf                # Azure provider configuration
├── terraform.tfvars          # Your secrets (NEVER commit - in .gitignore)
├── terraform.tfvars.example  # Template with placeholders (commit this)
├── .gitignore                # Ensures secrets aren't committed
└── README.md                 # Quick reference
```

**What Gets Committed to Git:**
- ✅ `main.tf`, `variables.tf`, `outputs.tf`, `provider.tf` - Infrastructure code
- ✅ `terraform.tfvars.example` - Template showing what variables are needed
- ✅ `.gitignore` - Prevents secrets from being committed
- ❌ `terraform.tfvars` - Contains actual secrets (passwords, keys)
- ❌ `*.tfstate` - Terraform state files (may contain sensitive data)

### Setup Steps

#### 1. Create terraform.tfvars (Contains Secrets - Store Locally Only)

**IMPORTANT:** `terraform.tfvars` contains **sensitive secrets** (passwords, API keys, client secrets) that must **NEVER** be committed to Git.

**Where to Store Secrets:**
- **Local Development:** Store in `terraform.tfvars` on your machine only (added to `.gitignore`)
- **CI/CD (GitHub Actions):** Store in GitHub Secrets (Repository Settings → Secrets)
- **Team Sharing:** Use Azure Key Vault or password manager (1Password, Bitwarden)

**Security Practice:**
```
✅ Commit to Git: terraform.tfvars.example (template with placeholders)
❌ NEVER commit: terraform.tfvars (actual values with secrets)
✅ Commit to Git: .gitignore (ensures terraform.tfvars is ignored)
```

**Step 1: Create your local secrets file**

```bash
cd infrastructure
cp terraform.tfvars.example terraform.tfvars
```

**Step 2: Get your secret values from their sources**

Edit `terraform.tfvars` and fill in with **real values**:

```hcl
# infrastructure/terraform.tfvars (LOCAL ONLY - NEVER COMMIT)
resource_group_name      = "rg-foodbythebook-dev"
location                 = "westeurope"
static_web_app_name      = "swa-foodbythebook-dev"
function_app_name        = "func-foodbythebook-dev"
storage_account_name     = "stfoodbythebook123"  # Must be globally unique
app_service_plan_name    = "asp-foodbythebook-dev"

# Supabase (Get from: https://supabase.com/dashboard → Your Project → Settings)
supabase_connection_string = "Host=db.xxx.supabase.co;Database=postgres;Username=postgres;Password=YOUR_ACTUAL_PASSWORD"
supabase_url               = "https://xxx.supabase.co"
supabase_key               = "YOUR_ACTUAL_SUPABASE_ANON_KEY"

# Microsoft OAuth (Get from: https://portal.azure.com → Entra ID → App Registrations)
azure_ad_tenant_id     = "YOUR_ACTUAL_TENANT_ID"
azure_ad_client_id     = "YOUR_ACTUAL_CLIENT_ID"
azure_ad_client_secret = "YOUR_ACTUAL_CLIENT_SECRET"
```

**Where to Get Each Secret:**

| Secret | Where to Get It |
|--------|-----------------|
| `supabase_connection_string` | Supabase Dashboard → Project Settings → Database → Connection String (Transaction mode) |
| `supabase_url` | Supabase Dashboard → Project Settings → API → Project URL |
| `supabase_key` | Supabase Dashboard → Project Settings → API → Project API Keys (anon/public) |
| `azure_ad_tenant_id` | Azure Portal → Entra ID → Overview → Tenant ID |
| `azure_ad_client_id` | Azure Portal → Entra ID → App Registrations → Your App → Application (client) ID |
| `azure_ad_client_secret` | Azure Portal → Entra ID → App Registrations → Your App → Certificates & secrets → Create new secret |

**Step 3: Verify .gitignore**

Ensure `infrastructure/.gitignore` contains:
```
# Terraform
*.tfstate
*.tfstate.*
.terraform/
.terraform.lock.hcl
terraform.tfvars      # ← This prevents committing secrets
*.tfvars             # ← Catch all .tfvars files
!terraform.tfvars.example  # ← Except the example template
```

#### 2. Initialize Terraform

```bash
terraform init
```

This downloads the Azure provider and prepares Terraform.

#### 3. Preview Changes

```bash
terraform plan
```

Review what Terraform will create. Should show:
- 1 Resource Group
- 1 Static Web App
- 1 Storage Account
- 1 App Service Plan
- 1 Function App

#### 4. Apply Infrastructure

```bash
terraform apply
```

Type `yes` when prompted. Takes ~2-3 minutes.

#### 5. Get Outputs

```bash
terraform output
```

Shows:
- Frontend URL: `https://xxx.azurestaticapps.net`
- Backend URL: `https://func-foodbythebook-dev.azurewebsites.net`

### Update Frontend Configuration

After Terraform creates infrastructure, update your frontend `.env`:

**Note:** Frontend `.env` also contains non-secret configuration that IS committed to Git (see section below).

```bash
# frontend/.env.local (LOCAL ONLY - NEVER COMMIT)
VITE_API_URL=<backend_url from terraform output>
VITE_MSAL_CLIENT_ID=<your Microsoft client ID>
VITE_MSAL_TENANT_ID=<your Microsoft tenant ID>
```

**Frontend .gitignore:**
```
# frontend/.gitignore
.env.local       # ← Your local secrets
.env.*.local     # ← Environment-specific secrets
```

**Frontend Secret Management:**
- ✅ Commit: `.env.example` (template with placeholders)
- ❌ Never commit: `.env.local` (actual values)
- **GitHub Actions:** Store in GitHub Secrets (see CI/CD section below)

---

## CI/CD Secrets Management (GitHub Actions)

**For Automated Deployments:** Store secrets in GitHub repository settings.

### Setting Up GitHub Secrets

1. Go to your GitHub repository
2. Settings → Secrets and variables → Actions
3. Click "New repository secret"
4. Add each secret:

| Secret Name | Value Source |
|-------------|--------------|
| `AZURE_CREDENTIALS` | Azure service principal JSON (see below) |
| `SUPABASE_CONNECTION_STRING` | From Supabase dashboard |
| `SUPABASE_URL` | From Supabase dashboard |
| `SUPABASE_KEY` | From Supabase dashboard |
| `AZURE_AD_TENANT_ID` | From Azure Portal |
| `AZURE_AD_CLIENT_ID` | From Azure Portal |
| `AZURE_AD_CLIENT_SECRET` | From Azure Portal |

### Creating Azure Service Principal

For GitHub Actions to deploy to Azure:

```bash
# Create service principal with contributor role
az ad sp create-for-rbac \
  --name "github-actions-foodbythebook" \
  --role contributor \
  --scopes /subscriptions/{subscription-id}/resourceGroups/rg-foodbythebook \
  --sdk-auth

# Output (save this as AZURE_CREDENTIALS secret in GitHub):
# {
#   "clientId": "...",
#   "clientSecret": "...",
#   "subscriptionId": "...",
#   "tenantId": "..."
# }
```

### Using Secrets in GitHub Actions

```yaml
# .github/workflows/deploy.yml
name: Deploy Infrastructure and Apps

on:
  push:
    branches: [main]

jobs:
  terraform:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v2
      
      - name: Azure Login
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}
      
      - name: Terraform Init
        run: terraform init
        working-directory: ./infrastructure
      
      - name: Terraform Apply
        run: terraform apply -auto-approve
        working-directory: ./infrastructure
        env:
          TF_VAR_supabase_connection_string: ${{ secrets.SUPABASE_CONNECTION_STRING }}
          TF_VAR_supabase_url: ${{ secrets.SUPABASE_URL }}
          TF_VAR_supabase_key: ${{ secrets.SUPABASE_KEY }}
          TF_VAR_azure_ad_tenant_id: ${{ secrets.AZURE_AD_TENANT_ID }}
          TF_VAR_azure_ad_client_id: ${{ secrets.AZURE_AD_CLIENT_ID }}
          TF_VAR_azure_ad_client_secret: ${{ secrets.AZURE_AD_CLIENT_SECRET }}
```

**Benefits:**
- ✅ Secrets never in Git repository
- ✅ Team members don't need to share credentials
- ✅ Automatic deployments without exposing secrets
- ✅ Easy to rotate credentials (update in one place)

---

## Environment Management

### Development Environment

```bash
terraform workspace new dev
terraform workspace select dev
# Use terraform.tfvars with dev values
terraform apply
```

### Production Environment

```bash
terraform workspace new prod
terraform workspace select prod
# Use terraform.prod.tfvars with prod values
terraform apply -var-file="terraform.prod.tfvars"
```

## Changing Hosting Provider

To migrate to Vercel/Netlify in the future:

1. **Frontend Only Migration:**
   ```hcl
   # Comment out Azure Static Web App in main.tf
   # resource "azurerm_static_site" "frontend" { ... }
   
   # Deploy to Vercel via CLI instead
   ```

2. **Keep Backend in Azure:**
   - No changes needed to Azure Functions
   - Update CORS in Functions to allow new domain

3. **Full Migration (if needed):**
   - Create new Terraform config for target cloud
   - Deploy new infrastructure
   - Update DNS
   - Destroy old Azure resources

## Cost Monitoring

All resources use **FREE tiers**:
- Static Web App: Free (100GB bandwidth/month)
- Functions: Free (1M executions/month)
- Storage Account: ~$0.01/month (minimal usage)

**Total: $0-1/month**

Monitor costs:
```bash
# Check current costs
az consumption usage list --output table

# Set up budget alert (optional)
az consumption budget create --amount 5 --category cost --name "FoodByTheBook-Budget"
```

## Cleanup

To destroy all infrastructure:

```bash
terraform destroy
```

Type `yes` when prompted. **WARNING:** This deletes everything!

## Troubleshooting

### "Storage account name already taken"
Storage account names must be globally unique. Change `storage_account_name` in terraform.tfvars.

### "Insufficient permissions"
Ensure you're logged in: `az login`
Check subscription: `az account show`

### "Function app deployment failed"
- Check Azure Portal → Function App → Deployment Center
- Verify GitHub connection
- Check logs in Azure Portal

### CORS errors from frontend
Verify `cors` block in `azurerm_linux_function_app` includes your frontend URL.

## Migration Path

If you decide to change hosting later:

**Scenario 1: Move frontend to Vercel**
1. Keep Terraform for backend (Azure Functions)
2. Remove Static Web App from Terraform
3. Deploy frontend to Vercel via Vercel CLI
4. Update CORS in Function App

**Scenario 2: Move everything to AWS**
1. Create new Terraform config with AWS provider
2. Deploy to AWS
3. Update DNS
4. Run `terraform destroy` for Azure resources

**Key Point:** All configuration is in code, migration is just changing Terraform files.

## Next Steps

1. ✅ Run `terraform apply` to create infrastructure
2. ✅ Note output URLs
3. ✅ Update frontend `.env` with backend URL
4. ✅ Configure GitHub Actions to deploy to Static Web App
5. ✅ Push code to GitHub
6. ✅ Verify deployment

See [TECH_STACK.md](TECH_STACK.md) for deployment workflow with GitHub Actions.
