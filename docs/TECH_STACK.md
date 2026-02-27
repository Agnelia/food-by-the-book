# Technology Stack

> **Note:** For alternatives considered and decision rationale, see [DECISION_HISTORY.md](DECISION_HISTORY.md)

## Budget Constraint: Free / Low-Cost Hobby Project 💰

**Goal:** Build and deploy MVP with $0-10/month costs

**Strategy:**
- Use free tiers and open-source alternatives
- Organizational backing preferred over single-maintainer libraries
- Accept trade-offs (lower OCR accuracy) for zero cost

---

## Frontend

### Framework
**Selected:** Vite + React 18 + React Router + TypeScript

**Key Benefits:**
- Pure React hooks-based development
- Blazing fast HMR with Vite
- Simple static deployment
- Perfect fit with separated backend (Azure Functions)
- No SSR overhead for private app

**Cost:** FREE

> **Alternatives Considered:** Next.js, Vue.js, Angular, Svelte — [See comparison](DECISION_HISTORY.md#frontend-framework-decision)

### UI Component Library
**Selected:** Arbetsförmedlingens Designsystem (Digi)

**Key Features:**
- Accessibility-focused (WCAG compliant)
- Design tokens for customization
- Web Components (framework-agnostic)
- Apache 2.0 Open Source

**Resources:**
- Library: [@designsystem-se/af](https://gitlab.com/arbetsformedlingen/designsystem/digi)
- Docs: [designsystem.arbetsformedlingen.se](https://designsystem.arbetsformedlingen.se/)

> **Alternatives Considered:** Material-UI, Ant Design, Chakra UI, Tailwind CSS, Bootstrap — [See comparison](DECISION_HISTORY.md#ui-component-library-decision)

### Data Fetching & State Management
**Selected:** Redux Toolkit (RTK) + RTK Query

**Key Benefits:**
- **Organizational Backing:** Redux core team (Mark Erikson + contributors)
- **Enterprise Adoption:** Microsoft, Spotify, Amazon use it
- **Integrated Solution:** State management + data fetching in one
- Auto-generated hooks from API definitions
- Tag-based cache invalidation
- Excellent TypeScript support
- Redux DevTools for debugging

**Bundle Size:** ~20KB (includes full state management)

**Cost:** FREE (MIT licensed)

> **Why RTK Query over alternatives:** Organizational backing (Redux team) vs single maintainer — [See detailed comparison](DECISION_HISTORY.md#data-fetching--state-management-decision)

### Implementation Strategy

**Redux Toolkit:** Global state (user session, UI state)  
**RTK Query:** Server state (recipes, ratings) with auto-generated hooks  
**useState:** Local component state  
**React Hook Form:** Form inputs

### Forms & Validation
**Selected:** React Hook Form + Zod

**Key Benefits:**
- Hooks-based API
- Minimal re-renders (performance)
- TypeScript-first with Zod schemas
- Small bundle (~8KB gzipped)

**Use Cases:** Recipe forms, OCR text editing, search inputs

**Cost:** FREE

---

## Backend

### Framework/Runtime
**Selected:** Azure Functions (.NET 8 Isolated Worker) + C#

**Key Benefits:**
- **Free Tier:** 1M executions/month + 400,000 GB-seconds compute
- Fast cold starts (~500ms)
- Strong typing (matches TypeScript frontend)
- Native Microsoft OAuth integration
- Serverless auto-scaling
- Industry-standard microservices pattern

**Architecture:**
- HTTP-triggered functions for REST API
- Dependency injection for services
- CORS enabled for frontend domain

**API Endpoints (MVP):**
- GET /api/recipes - List recipes
- POST /api/recipes - Create recipe
- GET /api/recipes/{id} - Get recipe
- PUT /api/recipes/{id} - Update recipe
- DELETE /api/recipes/{id} - Delete recipe
- POST /api/recipes/{id}/images - Upload image

**Cost:** FREE (within free tier limits)

> **Alternatives Considered:** Node.js, NestJS, Python (FastAPI/Django), Go — [See comparison](DECISION_HISTORY.md#backend-framework-decision)

---

## Database

### Primary Database
**Selected:** PostgreSQL on Supabase (Free Tier)

**Key Benefits:**
- **Free Tier:** 500MB database, 2GB file storage
- All PostgreSQL features (JSONB, full-text search, ACID)
- Hosted (no server management)
- Authentication included
- Realtime subscriptions built-in

**Storage:** Supabase Storage (2GB FREE for images)

**Cost:** FREE (up to 500MB data)

> **Alternatives Considered:** Railway, Neon, Vercel Postgres — [See comparison](DECISION_HISTORY.md#database-decision)

### Caching
**Selected:** Skip for MVP

**Future Option:** Upstash Redis free tier (10k requests/day)

---

## AI & OCR Services

### OCR (Optical Character Recognition)
**Selected:** Tesseract.js (Client-Side, FREE)

**Key Benefits:**
- 100% FREE (open source)
- Client-side processing (privacy)
- No API costs or server load

**Trade-offs Accepted:**
- Lower accuracy (70-85% vs 95% for cloud OCR)
- Slower processing (5-10s vs 1-2s)
- **Acceptable for hobby project with 1 user**

**Cost:** FREE

> **Future Upgrade Options:** Azure Vision, Google Vision, OCR.space — [See comparison](DECISION_HISTORY.md#ocr-service-decision)

---

## Authentication & Authorization
**Selected:** Microsoft OAuth 2.0 + JWT Tokens

**Key Benefits:**
- 100% FREE (no per-user costs)
- No password management
- OAuth 2.0 industry standard
- Native C# integration with Azure Functions

**Flow:**
1. User clicks "Sign in with Microsoft"
2. Microsoft OAuth consent page
3. Frontend receives authorization code
4. Azure Function exchanges code for JWT
5. Frontend uses JWT for API calls

**Libraries:**
- Frontend: `@azure/msal-browser`
- Backend: `Microsoft.Identity.Web`

**Cost:** FREE

> **Alternatives Considered:** Email/password, Auth0, Google OAuth — [See comparison](DECISION_HISTORY.md#authentication-decision)

---

## Cloud Hosting & Deployment

### Frontend Hosting
**Selected:** Azure Static Web Apps (FREE tier)

**Key Benefits:**
- Same ecosystem as backend (simplified management)
- No CORS complexity
- 100GB bandwidth/month
- GitHub integration (auto-deploy on push)
- Custom domains with free SSL
- Preview environments for PRs

**Cost:** FREE (Standard: $9/month if exceeding limits)

> **Alternatives Considered:** Vercel, Netlify — [Detailed comparison](DECISION_HISTORY.md#frontend-hosting-decision)

### Backend Hosting
**Selected:** Azure Functions (FREE tier)
- 1M executions/month
- Deployed separately from frontend
- CORS configured for frontend domain

**Deployment Architecture:**
```
┌─────────────────────────────────────┐
│  Frontend: Vercel/Netlify/Azure SWA │
│  (Vite React App - Static Build)    │
└──────────────┬──────────────────────┘
               │ HTTPS API Calls
               │ https://your-app.azurewebsites.net/api/*
               ▼
┌──────────────────────────────────────┐
│  Backend: Azure Functions            │
│  (C# .NET 8 HTTP Functions)          │
└──────────────┬───────────────────────┘
               │ PostgreSQL Connection
               ▼
┌──────────────────────────────────────┐
│  Database: Supabase PostgreSQL       │
│  Storage: Supabase Storage           │
└──────────────────────────────────────┘
```

**Total Cost:** $0/month

---

### Infrastructure as Code
**Selected:** Terraform

**Key Benefits:**
- Cloud-agnostic (works with Azure, AWS, GCP)
- Declarative (describe desired state)
- Version controlled (track changes in Git)
- Easy migration between cloud providers
- Free and open source

**Infrastructure Components Managed by Terraform:**
1. Azure Static Web App (frontend hosting)
2. Azure Functions App (backend hosting)
3. Azure Storage Account (required for Functions)
4. Application Insights (monitoring, optional for MVP)
5. Resource Group (container for all resources)

**Terraform Configuration Structure:**
```
/infrastructure
├── main.tf                    # Main Terraform configuration
├── variables.tf               # Input variables
├── outputs.tf                 # Output values (URLs, etc.)
├── terraform.tfvars          # Variable values (gitignored)
├── terraform.tfvars.example  # Example template (committed)
├── provider.tf               # Azure provider configuration
└── README.md                 # Infrastructure documentation
```

**Example Terraform Configuration:**
```hcl
# infrastructure/main.tf
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

provider "azurerm" {
  features {}
}

# Resource Group
resource "azurerm_resource_group" "main" {
  name     = var.resource_group_name
  location = var.location
}

# Static Web App (Frontend)
resource "azurerm_static_site" "frontend" {
  name                = var.static_web_app_name
  resource_group_name = azurerm_resource_group.main.name
  location           = var.location
  sku_tier           = "Free"
  sku_size           = "Free"
}

# Storage Account (required for Azure Functions)
resource "azurerm_storage_account" "functions" {
  name                     = var.storage_account_name
  resource_group_name      = azurerm_resource_group.main.name
  location                 = azurerm_resource_group.main.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
}

# App Service Plan (for Azure Functions)
resource "azurerm_service_plan" "functions" {
  name                = var.app_service_plan_name
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  os_type             = "Linux"
  sku_name            = "Y1"  # Consumption plan (FREE tier)
}

# Azure Functions App (Backend)
resource "azurerm_linux_function_app" "backend" {
  name                       = var.function_app_name
  resource_group_name        = azurerm_resource_group.main.name
  location                   = azurerm_resource_group.main.location
  service_plan_id            = azurerm_service_plan.functions.id
  storage_account_name       = azurerm_storage_account.functions.name
  storage_account_access_key = azurerm_storage_account.functions.primary_access_key

  site_config {
    application_stack {
      dotnet_version              = "8.0"
      use_dotnet_isolated_runtime = true
    }
    
    cors {
      allowed_origins = [
        azurerm_static_site.frontend.default_host_name,
        "http://localhost:5173"  # Vite dev server
      ]
      support_credentials = true
    }
  }

  app_settings = {
    "FUNCTIONS_WORKER_RUNTIME"       = "dotnet-isolated"
    "SupabaseConnectionString"       = var.supabase_connection_string
    "SupabaseUrl"                    = var.supabase_url
    "SupabaseKey"                    = var.supabase_key
    "AzureAd__TenantId"             = var.azure_ad_tenant_id
    "AzureAd__ClientId"             = var.azure_ad_client_id
    "AzureAd__ClientSecret"         = var.azure_ad_client_secret
  }
}
```

**Variables Configuration:**
```hcl
# infrastructure/variables.tf
variable "resource_group_name" {
  description = "Name of the Azure Resource Group"
  type        = string
  default     = "rg-foodbythebook"
}

variable "location" {
  description = "Azure region for resources"
  type        = string
  default     = "westeurope"  # Change to your preferred region
}

variable "static_web_app_name" {
  description = "Name of the Static Web App"
  type        = string
}

variable "function_app_name" {
  description = "Name of the Azure Functions App"
  type        = string
}

variable "supabase_connection_string" {
  description = "Supabase PostgreSQL connection string"
  type        = string
  sensitive   = true
}

variable "azure_ad_tenant_id" {
  description = "Microsoft Entra ID Tenant ID"
  type        = string
  sensitive   = true
}

# ... more variables
```

**Usage:**
```bash
# Initialize Terraform
cd infrastructure
terraform init

# Create terraform.tfvars from example
cp terraform.tfvars.example terraform.tfvars
# Edit terraform.tfvars with your values

# Preview changes
terraform plan

# Apply infrastructure
terraform apply

# Get output values (URLs, etc.)
terraform output

# Destroy infrastructure (cleanup)
terraform destroy
```

> **Alternative Considered:** Azure Bicep (Azure-only) — [See comparison](DECISION_HISTORY.md#infrastructure-as-code-decision)

---

## Development Tools

### Version Control
**Selected:** Git + GitHub
- Code repository on GitHub
- Branch protection on `main`
- PR reviews
- Issues and project boards

### CI/CD
**Selected:** GitHub Actions

**Key Benefits:**
- **Free:** 2000 build minutes/month (private), unlimited (public)
- Automatic deployments on push to `main`
- PR preview deployments
- Run tests before deployment

**Workflow:**
```
1. Push to GitHub → GitHub Actions triggers
2. Build Vite app → Deploy to Azure Static Web Apps
3. Build .NET project → Deploy to Azure Functions
4. Run tests (optional)
5. Notify on success/failure
```

### Code Quality (MVP)
- ✅ ESLint (JavaScript/TypeScript linting)
- ✅ Prettier (code formatting)
- ⏳ Husky (Git hooks) - Phase 2
- ⏳ SonarQube (code analysis) - Phase 2

### Testing (Phase 2+)
- ⏳ Jest (unit testing)
- ⏳ React Testing Library
- ⏳ Cypress/Playwright (E2E)

**MVP Philosophy:** Deploy working version first, add testing later

---

## Additional Services

### Image Storage
**Selected:** Supabase Storage (FREE: 2GB)

**Alternative:** Cloudinary (FREE: 25GB bandwidth/month)

### Email Service
**Deferred to Phase 2**

Options when needed: SendGrid, Mailgun, AWS SES, Postmark

### CDN
**Included:** Azure Static Web Apps includes global CDN (FREE)

---

## Monthly Cost Summary

**Phase 1 (MVP):**
- Frontend Hosting: **$0** (Azure Static Web Apps free tier)
- Backend Hosting: **$0** (Azure Functions free tier)
- Database + Storage: **$0** (Supabase free tier)
- OCR: **$0** (Tesseract.js open source)
- Domain (optional): **~$1/month** ($12/year)

**Total: $0-1/month** 🎉

**Phase 2 (When scaling):**
- Email service: **$0-20**
- AI suggestions (OpenAI): **$5-10**
- Better OCR: **$0-10**

**Total: $5-30/month** (only when needed)

---

## Development Phases

### Phase 1: MVP (Weeks 1-6) - FREE
- Vite + React setup
- Supabase database + auth
- Recipe CRUD operations
- Photo upload + Tesseract OCR
- Manual text editing
- Deploy to Azure
- **Goal: 1 real user**

### Phase 2: Enhanced Features (Months 2-3)
- Email service (when needed)
- AI recipe suggestions
- Advanced filtering
- Better OCR if needed
- **Goal: 10-50 users**

### Phase 3: Scale (Months 4+)
- Family features
- Advanced AI
- Mobile app
- Upgrade paid tiers as needed
- **Goal: 100+ users**

---

## Upgrade Triggers

**When to upgrade from free tiers:**

1. **Azure Static Web Apps → Standard ($9/mo):**
   - Exceeding 100GB bandwidth/month
   - Need team collaboration
   - Want advanced analytics

2. **Supabase → Pro ($25/mo):**
   - Exceeding 500MB database (~thousands of recipes)
   - Exceeding 2GB storage (~hundreds of photos)
   - Need daily backups

3. **Add AI Services ($10-20/mo):**
   - Users actively request AI suggestions
   - Have 50+ recipes per user

**Rule:** Don't pay until users prove they want it!
