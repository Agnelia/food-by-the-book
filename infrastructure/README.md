# Infrastructure - Terraform (Azure)

This directory contains Infrastructure as Code (IaC) using Terraform to provision Azure resources.

## Structure

```
infrastructure/
├── main.tf                      # Main resource definitions
├── variables.tf                 # Variable declarations
├── outputs.tf                   # Output definitions
├── provider.tf                  # Azure provider configuration
├── terraform.tfvars.example     # Template with placeholders (SAFE TO COMMIT)
├── terraform.tfvars            # Actual secrets (NEVER COMMIT - in .gitignore)
├── .gitignore                  # Prevents secrets from being committed
└── README.md                   # This file
```

## Resources Provisioned

- **Resource Group** - Container for all Azure resources
- **Azure Static Web App** - Hosts the frontend (free tier)
- **Azure Functions App** - Hosts the backend API (consumption plan, free tier)
- **Storage Account** - Required for Azure Functions runtime
- **App Service Plan** - Defines the hosting plan for Functions

## External Services (Not in Terraform)

- **Supabase** - PostgreSQL database and storage (managed externally)
- **Microsoft Entra ID** - OAuth 2.0 provider (manual setup required)

## Getting Started

### Prerequisites
- Azure CLI (`az login`)
- Terraform 1.0+
- Azure account (free tier)

### Initial Setup

1. **Create your secrets file** (NEVER commit this file):
```bash
cp terraform.tfvars.example terraform.tfvars
# Edit terraform.tfvars with your actual values
```

2. **Initialize Terraform**:
```bash
terraform init
```

3. **Preview changes**:
```bash
terraform plan
```

4. **Apply infrastructure**:
```bash
terraform apply
```

5. **Get outputs**:
```bash
terraform output
```

## Secrets Management

**CRITICAL**: `terraform.tfvars` contains sensitive secrets and must NEVER be committed to Git.

### What to Commit
- ✅ `main.tf`, `variables.tf`, `outputs.tf`, `provider.tf` - Infrastructure code
- ✅ `terraform.tfvars.example` - Template with placeholders
- ✅ `.gitignore` - Prevents secrets from being committed
- ❌ `terraform.tfvars` - Contains actual secrets (passwords, keys)
- ❌ `*.tfstate` - Terraform state files (may contain sensitive data)

### Where to Store Secrets

**Local Development**: 
- Store in `terraform.tfvars` on your machine only (added to `.gitignore`)

**CI/CD (GitHub Actions)**: 
- Store in GitHub Repository Settings → Secrets and variables → Actions

**Team Sharing**: 
- Use Azure Key Vault or password manager (1Password, Bitwarden)
- Never share via email, Slack, or commit to Git

## Cost Monitoring

All resources use FREE tiers:
- Static Web App: Free (100GB bandwidth/month)
- Azure Functions: Free (1M executions/month)
- Storage Account: ~$0.01/month (minimal usage)

**Total: $0-1/month**

## Cleanup

To destroy all infrastructure:
```bash
terraform destroy
```

**WARNING**: This deletes everything!

## Reference
See [INFRASTRUCTURE.md](../docs/INFRASTRUCTURE.md) for detailed documentation.
