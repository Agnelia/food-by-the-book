# Technology Stack

## Budget Constraint: Free / Low-Cost Hobby Project 💰

**Goal:** Build and deploy MVP with $0-10/month costs

**Strategy:**
- Use free tiers and open-source alternatives
- Start small, upgrade services only when needed
- Accept trade-offs (lower OCR accuracy, manual processes)

---

## Decision Criteria
When selecting technologies, consider:
- Team expertise and learning curve
- Community support and documentation
- Long-term maintenance and updates
- Performance requirements
- Cost (licensing, hosting, etc.)
- Ecosystem and tooling
- Scalability needs

---

## Frontend

### Framework
**Selected:** Vite + React 18 + React Router + TypeScript

**Decision Rationale:**
- **Pure React Experience:** Full hooks-based development (useState, useEffect, custom hooks)
- **No Server Complexity:** Client-only app, everything runs in browser
- **Blazing Fast Dev:** Vite's HMR is significantly faster than Next.js
- **Lightweight:** No SSR overhead since backend is separate (Azure Functions)
- **Simple Deployment:** Static build deployed to Vercel/Netlify/Azure Static Web Apps
- **Perfect Fit:** With Azure Functions handling backend, don't need Next.js API routes
- **React Router:** Manual routing setup (trivial for experienced developers)

**Why Not Next.js (Anymore):**
- Backend moved to Azure Functions (don't need Next.js API routes)
- Private app for 1 user (don't need SSR/SEO)
- Prefer simplicity and speed for hobby project

**Cost:** FREE (Vercel/Netlify/Azure Static Web Apps free tier)

**Options Considered:****

| Technology | Pros | Cons | Decision |
|------------|------|------|----------|
| **React** | - Huge ecosystem<br>- Component reusability<br>- Strong community<br>- Great tooling | - Steeper learning curve<br>- Needs additional libraries for routing, state | ⬜ |
| **Vue.js** | - Easier learning curve<br>- Good documentation<br>- Progressive framework<br>- Single-file components | - Smaller ecosystem than React<br>- Less job market demand | ⬜ |
| **Angular** | - Complete framework<br>- TypeScript by default<br>- Enterprise-ready<br>- Opinionated structure | - Steepest learning curve<br>- More verbose<br>- Heavier bundle size | ⬜ |
| **Svelte** | - Best performance<br>- Less boilerplate<br>- Smaller bundle size<br>- Easy to learn | - Smaller ecosystem<br>- Fewer job opportunities<br>- Less mature | ⬜ |
| **Next.js** | - React with SSR/SSG<br>- Great for SEO<br>- Built-in routing<br>- API routes | - Learning curve<br>- Vendor lock-in | ⬜ |

### UI Component Library
**Selected:** Arbetsförmedlingens Designsystem (Digi)

**Decision:**
- **Library:** [@designsystem-se/af](https://gitlab.com/arbetsformedlingen/designsystem/digi)
- **Documentation:** [designsystem.arbetsformedlingen.se](https://designsystem.arbetsformedlingen.se/)
- **License:** Apache 2.0 (Open Source)
- **Technology:** Web Components built with Stencil (framework-agnostic)
- **Compatibility:** Works with React, Angular, Vue, vanilla JavaScript
- **Features:**
  - Accessibility-focused (WCAG compliant)
  - Design tokens for customization
  - Swedish government design patterns
  - Open source with active development

**Customization Notes:**
- Design tokens allow customization of colors, spacing, typography
- Components are built to be themeable
- Multi-language support possible (Swedish by default)
- Can override CSS variables for branding

**Other Options Considered:**
- Material-UI / MUI
- Ant Design
- Chakra UI
- Tailwind CSS + Headless UI
- Bootstrap
- Shadcn/ui

### Data Fetching
**Selected:** SWR (Stale-While-Revalidate)

**Rationale:**
- **Hooks-based API:** `useSWR('https://your-functions.azurewebsites.net/api/recipes', fetcher)`
- **Works with Any Backend:** Perfect for calling Azure Functions endpoints
- **Auto-revalidation:** Keeps data fresh automatically
- **Caching:** Built-in cache, reduces API calls
- **FREE:** Open source, no cost
- **Small bundle:** Lightweight library

**Features:**
- Automatic refetching on focus
- Optimistic UI updates
- Pagination support
- TypeScript support

**Alternative:** React Query (more features, slightly heavier)

### State Management
**Selected:** React Hooks (useState, useContext, custom hooks)

**Rationale:**
- **Built-in:** No extra library needed
- **Sufficient for MVP:** User session, recipe list, forms are not complex
- **Hooks-based:** Fits chosen React approach
- **Custom hooks:** Can extract reusable logic (usePhotoUpload, useRecipes)

**Implementation:**
- `useState` for local component state
- `useContext` for global state (if needed)
- Custom hooks for shared logic
- NextAuth's `useSession` for auth state
- SWR's hooks for data fetching

**Options:**
- Zustand
- Redux Toolkit
- Recoil
- MobX

### Forms & Validation
**Selected:** React Hook Form + Zod

**Rationale:**
- **Hooks-based:** `useForm()` hook API
- **Performance:** Minimal re-renders
- **TypeScript:** Full type safety with Zod schemas
- **FREE:** Open source
- **Small bundle:** ~8kb gzipped

**Use cases:**
- Recipe create/edit forms
- OCR text editing
- Search inputs

---

## Backend

### Framework/Runtime
**Selected:** Azure Functions (.NET 8 Isolated Worker) + C#

**Decision Rationale:**
- **Decoupled Architecture:** Separate frontend and backend deployments
- **Free Tier:** 1 million executions/month FREE + 400,000 GB-seconds compute FREE
- **Fast Cold Starts:** ~500ms on C# (better than Python)
- **Strong Typing:** C# matches TypeScript approach on frontend
- **Microsoft Ecosystem:** Native integration with Microsoft OAuth, Azure services
- **Scalability:** Auto-scales (though not needed for 1 user MVP)
- **Industry Standard:** Microservices pattern, easy to understand

**Architecture:**
- HTTP-triggered functions for REST API endpoints
- Dependency injection for services
- Separate from frontend (Vite app calls Azure Functions URLs)
- CORS enabled for frontend domain

**Cost:** FREE (stays within free tier limits for hobby project)

**MVP Approach:**
Build HTTP-triggered Azure Functions for REST API:
- GET /api/recipes - List all recipes
- POST /api/recipes - Create recipe
- GET /api/recipes/{id} - Get single recipe
- PUT /api/recipes/{id} - Update recipe
- DELETE /api/recipes/{id} - Delete recipe
- POST /api/recipes/{id}/images - Upload image

**Implementation:**
- .NET 8 Isolated Worker Model (recommended by Microsoft)
- Dependency injection for database connections
- Entity Framework Core or Dapper for Supabase PostgreSQL
- JWT bearer authentication
- CORS middleware for Vercel frontend domain

**Deployment:**
- Azure Functions Core Tools CLI
- Deploy from VS Code or GitHub Actions
- Environment variables for Supabase connection string

**Options Considered:**
|------------|------|------|----------|
| **Node.js (Express)** | - JavaScript everywhere<br>- Fast development<br>- Large ecosystem<br>- Great for real-time | - Callback hell (if not using async/await)<br>- Less structured | ⬜ |
| **Node.js (NestJS)** | - TypeScript first<br>- Angular-like structure<br>- Built-in modules<br>- Great for enterprise | - Learning curve<br>- More opinionated | ⬜ |
| **.NET (C#)** | - Type-safe<br>- Excellent tooling<br>- Great performance<br>- Enterprise-ready | - Steeper learning curve<br>- Windows-centric (changing) | ⬜ |
| **Python (FastAPI)** | - Easy to learn<br>- Great for ML/AI<br>- Fast development<br>- Automatic docs | - Slower than compiled languages<br>- Async still maturing | ⬜ |
| **Python (Django)** | - Batteries included<br>- Admin panel<br>- ORM built-in<br>- Mature | - Heavier framework<br>- Less flexible | ⬜ |
| **Go** | - Excellent performance<br>- Simple concurrency<br>- Static typing<br>- Fast compilation | - Less web frameworks<br>- Smaller ecosystem | ⬜ |

---

## Database

### Primary Database
**Selected:** PostgreSQL on Supabase (Free Tier)

**Rationale:**
- **Free Tier:** 500MB database, 2GB file storage, unlimited API requests
- **PostgreSQL:** All benefits (JSONB, full-text search, relational)
- **Easy Setup:** Database + authentication included
- **Hosted:** No server management
- **Realtime:** Built-in realtime subscriptions (bonus)
- **Authentication:** Can optionally use Supabase Auth (vs custom JWT)

**Cost:** FREE (up to 500MB data, more than enough for MVP)

**Alternatives:**
- **Railway:** PostgreSQL free tier (500MB, $5/month after)
- **Neon:** Serverless Postgres free tier (512MB)
- **Vercel Postgres:** $0.25/month minimum

**Additional Storage:**
- **Supabase Storage:** 2GB included in free tier for images
- **Alternative:** Cloudinary free tier (25GB bandwidth, 25k transformations/month)

**Cache (Phase 2):**
- **Upstash Redis:** Free tier (10k requests/day)
- **Or skip caching initially**
|------------|----------|------|------|----------|
| **PostgreSQL** | Relational, complex queries | - Robust<br>- JSON support<br>- ACID compliant<br>- Open source | - Vertical scaling limits<br>- More complex setup | ⬜ |
| **MySQL/MariaDB** | Relational, traditional | - Mature<br>- Fast reads<br>- Wide support | - Less features than PostgreSQL<br>- Licensing (MySQL) | ⬜ |
| **MongoDB** | Document store, flexible schema | - Schema flexibility<br>- Easy to start<br>- Horizontal scaling | - Less structured<br>- Memory intensive | ⬜ |
| **SQLite** | Lightweight, embedded | - Serverless<br>- Zero config<br>- Great for dev | - Not for production at scale<br>- Limited concurrency | ⬜ |

### Caching Layer
**Selected:** Skip for MVP (add Upstash Redis free tier if needed)

---

## AI & OCR Services (Free Alternatives)

### OCR (Optical Character Recognition)
**Selected:** Tesseract.js (Client-Side, FREE)

**Rationale:**
- **100% Free:** Open source, runs in browser
- **No API Costs:** Processing happens on user's device
- **Privacy:** Images never leave user's device
- **Good Enough:** 70-85% accuracy for printed text
- **No Server Load:** Offloads work to client

**Trade-offs:**
- Lower accuracy than Azure/Google (85% vs 95%)
- Slower processing (5-10 seconds vs 1-2 seconds)
- User must manually fix more mistakes
- **Acceptable for hobby project!**

**Implementation:**
```javascript
import Tesseract from 'tesseract.js';

// Client-side OCR
const result = await Tesseract.recognize(image, 'eng+swe');
const text = result.data.text;
```

**Future Upgrade Path (if budget allows):**
- OCR.space API (free tier: 500 requests/day)
- Google Cloud Vision (free tier: 1000 requests/month)
- Azure Computer Vision (if using Azure later)

### Natural Language Processing (Deferred to Phase 2)
**Selected:** Defer AI suggestions to Phase 2

**Rationale:**
- AI APIs cost money (OpenAI: ~$0.01-0.05 per suggestion)
- Need recipe collection first anyway
- Can prototype with keyword matching initially

**Phase 2 Options:**
- **OpenAI GPT-3.5:** ~$0.002 per 1k tokens (cheap)
- **Claude Haiku:** Similar pricing
- **Groq (free tier):** Very fast, free tier available
- **Local LLM:** Llama 3 via Ollama (free, self-hosted)

**MVP Workaround:**
Simple keyword search + filtering instead of AI

### Image Processing
**Selected:** Sharp (Node.js) or Built-in Browser APIs

**Purpose:**
- Thumbnail generation (Sharp on server)
- Image compression (Canvas API in browser)
- Format conversion

**Cost:** FREE (library)

---

## Authentication & Authorization
**Selected:** Microsoft OAuth 2.0 + JWT Tokens (FREE)

**Architecture:**
- **Frontend (Vite React):** OAuth redirect flow, stores JWT in memory/localStorage
- **Backend (Azure Functions):** Validates JWT on protected endpoints
- **Identity Provider:** Microsoft Entra ID (formerly Azure AD) - FREE

**Implementation Flow:**
1. User clicks "Sign in with Microsoft" in Vite app
2. Redirect to Microsoft OAuth consent page
3. Microsoft redirects back with authorization code
4. Frontend calls Azure Function endpoint with code
5. Azure Function exchanges code for tokens, creates/updates user in database
6. Azure Function returns JWT to frontend
7. Frontend includes JWT in Authorization header for API calls
8. Azure Functions validate JWT on each request

**Libraries:**
- **Frontend:** `@azure/msal-browser` (Microsoft Authentication Library for React)
- **Backend:** `Microsoft.Identity.Web` (C# JWT validation middleware)

**Why Microsoft OAuth:**
- **100% FREE:** No cost for authentication
- **No Password Management:** Microsoft handles all security
- **Industry Standard:** OAuth 2.0 + OpenID Connect
- **Native Integration:** C# libraries work perfectly with Azure Functions
- **Users May Already Have Accounts:** Microsoft accounts are common

**Cost:** FREE (no per-user or API call costs)

**Alternatives Considered:**
- Email/password: More work, security concerns, need password reset
- Auth0: $23+/month for features we don't need
- Google OAuth: Could work, but Microsoft fits better with Azure
- GitHub OAuth: Could work for developers only

---

## Cloud Hosting & Deployment

### Hosting Platform
**Selected:** 
- **Frontend:** Azure Static Web Apps (FREE tier) ✅
- **Backend:** Azure Functions (FREE tier) ✅
- **Infrastructure as Code:** Terraform ✅

---

## Frontend Hosting Options - Detailed Comparison

### Option 1: Azure Static Web Apps (⭐ RECOMMENDED)

**Why Recommended:**
- **Same Ecosystem:** Frontend and backend both in Azure = simplified management
- **No CORS Complexity:** Can use same domain with routing rules
- **Managed Identity:** Backend can authenticate to frontend without API keys
- **Free Tier:** 100GB bandwidth/month, unlimited apps
- **GitHub Integration:** Auto-deploys on push
- **Custom Domains:** Free SSL certificates
- **Staging Environments:** Every PR gets preview URL

**Free Tier Limits:**
- 100GB bandwidth/month
- 0.5GB storage
- 2 custom domains
- Unlimited apps

**Deployment Steps:**
1. Create Azure Static Web App in Azure Portal
2. Connect to GitHub repository
3. Configure build settings:
   ```yaml
   app_location: "/" 
   api_location: "" # Not used (separate Azure Functions)
   output_location: "dist"
   ```
4. Push to GitHub → auto-deploys

**Build Command:** `npm run build` (Vite outputs to `dist/`)

**Cost Beyond Free Tier:** $9/month (Standard plan) - only if you exceed 100GB bandwidth

**Best For:** You! Everything stays in Azure ecosystem.

---

### Option 2: Vercel

**Pros:**
- **Best Developer Experience:** Fastest deployments, great CLI
- **Created Vite:** Perfect optimization for Vite apps
- **Preview Deployments:** Every branch/PR gets unique URL
- **Edge Network:** Global CDN, very fast
- **Analytics:** Built-in Web Vitals tracking (free tier)
- **Zero Config:** Auto-detects Vite, no configuration needed

**Free Tier Limits:**
- 100GB bandwidth/month
- 6000 build minutes/month
- Unlimited projects
- 1 concurrent build

**Deployment Steps:**
1. Install Vercel CLI: `npm i -g vercel`
2. In project: `vercel` (first time links to GitHub)
3. Or: Import project in Vercel dashboard
4. Push to GitHub → auto-deploys

**Environment Variables:**
```bash
VITE_API_URL=https://your-functions.azurewebsites.net/api
VITE_MSAL_CLIENT_ID=your-client-id
```

**CORS Configuration Required:** Azure Functions must allow Vercel domain
```csharp
builder.WithOrigins("https://your-app.vercel.app")
```

**Cost Beyond Free Tier:** $20/month (Pro plan)

**Best For:** Want fastest deployments and best DX, don't mind managing two separate platforms (Vercel + Azure).

---

### Option 3: Netlify

**Pros:**
- **Easiest Setup:** Drag-and-drop deployment or Git integration
- **Form Handling:** Built-in form submissions (bonus feature)
- **Split Testing:** A/B testing built-in
- **Netlify Functions:** Has serverless functions (but you're using Azure)
- **Build Plugins:** Ecosystem of build-time plugins

**Free Tier Limits:**
- 100GB bandwidth/month
- 300 build minutes/month
- Unlimited sites
- 1 concurrent build

**Deployment Steps:**
1. Connect GitHub repo in Netlify dashboard
2. Configure build:
   - Build command: `npm run build`
   - Publish directory: `dist`
3. Add environment variables in Netlify UI
4. Push to GitHub → auto-deploys

**CORS Configuration Required:** Azure Functions must allow Netlify domain
```csharp
builder.WithOrigins("https://your-app.netlify.app")
```

**Cost Beyond Free Tier:** $19/month (Pro plan)

**Best For:** Want simplest possible setup, value form handling and A/B testing features.

---

## Side-by-Side Comparison

| Feature | Azure Static Web Apps | Vercel | Netlify |
|---------|----------------------|--------|---------|
| **Free Bandwidth** | 100GB | 100GB | 100GB |
| **Build Minutes** | Unlimited | 6000 | 300 |
| **Deploy Speed** | ~2-3 min | ~1-2 min | ~2-3 min |
| **Azure Integration** | ⭐ Native | Via CORS | Via CORS |
| **Custom Domains** | ✅ Free SSL | ✅ Free SSL | ✅ Free SSL |
| **Preview Deployments** | ✅ Per PR | ✅ Per PR | ✅ Per PR |
| **CLI Tool** | Azure CLI | Vercel CLI | Netlify CLI |
| **DX (Developer Experience)** | Good | ⭐ Excellent | Good |
| **Edge Network** | Azure CDN | ⭐ Vercel Edge | Netlify Edge |
| **Analytics** | ❌ | ✅ Free | ❌ (Paid) |
| **Learning Curve** | Medium | Low | Low |
| **Pro Plan Cost** | $9/mo | $20/mo | $19/mo |

---

## ✅ SELECTED: Azure Static Web Apps

**Decision Made:** Azure Static Web Apps for frontend hosting

**Reasons for Choosing Azure:**
1. **Unified Platform:** Both frontend and backend in Azure (simpler billing, management)
2. **CORS Simplicity:** Can configure backend to trust frontend without hardcoded domains
3. **Future Growth:** Easy to add Azure services later (Computer Vision, App Insights, etc.)
4. **Cost:** Same free tier as competitors, cheaper Pro plan ($9 vs $19-20)
5. **GitHub Integration:** Just as good as Vercel/Netlify
6. **Infrastructure as Code:** Terraform manages all Azure resources consistently

**Configuration Strategy:**
- All Azure infrastructure defined in Terraform
- Environment-specific variables separate from code
- Easy to migrate to Vercel/Netlify by changing Terraform config only

---

**Backend Hosting:**
- **Azure Functions:** Already selected, FREE tier (1M executions/month)
- **Deployed separately** from frontend (even if using Azure Static Web Apps)
- **CORS configured** to allow frontend domain

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

**Cost:** 
- Frontend: FREE
- Backend: FREE (Azure Functions free tier)
- Total: $0/month

**Upgrade Path:**
- Frontend: If exceeding limits, ~$20/month
- Backend: If exceeding 1M requests, pay-per-use (~$0.20 per million)

---

## Infrastructure as Code

**Selected:** Terraform

**Why Terraform:**
- **Cloud-Agnostic:** Same tool works for Azure, AWS, GCP (future-proof)
- **Declarative:** Describe what you want, Terraform figures out how
- **Version Controlled:** Infrastructure changes tracked in Git
- **Easy Migration:** Change provider = change hosting platform
- **State Management:** Terraform tracks what's deployed
- **Free & Open Source:** No cost, large community

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

**Benefits:**
- Change hosting: Edit `provider` in Terraform, reapply
- Multiple environments: Use workspaces (`terraform workspace new staging`)
- Team collaboration: Store Terraform state in Azure Storage
- Reproducible: Same configuration = same infrastructure every time

**Alternative Considered:** Azure Bicep (Azure-only, not cloud-agnostic)

---

### Deployment Strategy (GitHub-Based)

**Version Control:** Git + GitHub (chosen)

**CI/CD:** GitHub Actions (FREE for public repos, 2000 minutes/month for private)

**Deployment Flow:**
```
1. Developer pushes to GitHub branch
2. GitHub Actions triggers (on push to main/develop)
3. Frontend: Build Vite app → Deploy to Azure Static Web Apps/Vercel/Netlify
4. Backend: Build .NET project → Deploy to Azure Functions
5. Run tests (optional)
6. Notify on success/failure
```

**GitHub Actions Workflow Example:**
```yaml
# .github/workflows/deploy.yml
name: Deploy Frontend & Backend

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  deploy-frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
        working-directory: ./frontend
      - run: npm run build
        working-directory: ./frontend
      # Deploy step depends on chosen platform (Azure SWA/Vercel/Netlify)

  deploy-backend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-dotnet@v3
        with:
          dotnet-version: '8.0'
      - run: dotnet build
        working-directory: ./backend
      - uses: Azure/functions-action@v1
        with:
          app-name: your-function-app-name
          package: ./backend
```

**Benefits:**
- **Free:** GitHub Actions included with GitHub
- **Integrated:** Same platform as code repository
- **Automatic:** Deploy on every push to main
- **Preview Deployments:** PRs can trigger staging deployments

---

## Development Tools

### Version Control
**Selected:** Git + GitHub
- Code repository hosted on GitHub
- Branch protection rules on `main`
- Pull request reviews
- Issues and project boards

### CI/CD
**Selected:** GitHub Actions
- **Free Tier:** 2000 build minutes/month (private repos), unlimited (public repos)
- Automatic deployments on push to `main`
- PR preview deployments
- Run tests before deployment

### Code Quality (Recommended for MVP)
- ✅ **ESLint:** JavaScript/TypeScript linting
- ✅ **Prettier:** Code formatting (auto-format on save)
- ⏳ **Husky:** Git hooks (skip for MVP, add later)
- ⏳ **SonarQube:** Code analysis (skip for MVP, add later)

### Testing (Phase 2+)
- ⏳ Jest (Unit testing) - Add after MVP works
- ⏳ React Testing Library - Add after MVP works
- ⏳ Cypress (E2E testing) - Add after MVP works

**MVP Philosophy:** Get working version deployed first, add testing later
- [ ] Playwright (E2E testing)
- [ ] Postman/Insomnia (API testing)

### Monitoring & Logging
- [ ] Sentry (Error tracking)
- [ ] Datadog (APM)
- [ ] New Relic (APM)
- [ ] LogRocket (Frontend monitoring)
- [ ] ELK Stack (Logging)
- [ ] Azure Application Insights

---

## Additional Services

### Email
- [ ] SendGrid
- [ ] Mailgun
- [ ] AWS SES
- [ ] Postmark

### File Storage
✅ **Selected:** Supabase Storage (FREE: 2GB) or Cloudinary (FREE: 25GB bandwidth)

**Other options:**
- AWS S3 (costs money)
- Azure Blob Storage (costs money)
- Google Cloud Storage (costs money)

### CDN
✅ **Included:** Vercel includes global CDN (FREE)

**Other options (not needed):**
- Cloudflare
- AWS CloudFront
- Azure CDN

---

## Integration Requirements (MVP Phase 1)

### Phase 1 (MVP) - All FREE
- ✅ **OCR** - Tesseract.js (FREE, client-side)
- ✅ **Database** - Supabase PostgreSQL (FREE tier: 500MB)
- ✅ **Image Storage** - Supabase Storage (FREE: 2GB) or Cloudinary (FREE: 25GB bandwidth)
- ✅ **Authentication** - Microsoft OAuth + NextAuth.js (FREE)
- ✅ **Hosting** - Vercel (FREE: 100GB bandwidth)
- ✅ **Image Processing** - Sharp (FREE library)
- ❌ **Email Service** - Deferred to Phase 2
- ❌ **AI/NLP** - Deferred to Phase 2

### Phase 2 (When Budget Allows)
- [ ] **Email** - Resend (FREE: 100 emails/day, then $20/month for 50k)
- [ ] **AI Suggestions** - OpenAI GPT-3.5 (~$0.002/request) or Groq (FREE tier)
- [ ] **Better OCR** - OCR.space (FREE: 500/day) or Google Vision (FREE: 1000/month)
- [ ] **Monitoring** - Sentry free tier (5k events/month)
- [ ] **Analytics** - Vercel Analytics (FREE basic, $10/month pro)

---

## Final Stack Summary (FREE/Low-Cost)

```
Frontend:     Next.js 14 (App Router) + TypeScript
UI Library:   Arbetsförmedlingens Designsystem (Digi) + Tailwind CSS
Backend:      Next.js API Routes (serverless)
Database:     PostgreSQL on Supabase (FREE tier: 500MB)
Image Store:  Supabase Storage (FREE: 2GB) or Cloudinary (FREE: 25GB)
OCR:          Tesseract.js (FREE, client-side)
AI/NLP:       Deferred to Phase 2 (keyword search for MVP)
Auth:         Microsoft OAuth + NextAuth.js (FREE)
Email:        Deferred to Phase 2
Hosting:      Vercel (FREE: 100GB bandwidth/month)
CI/CD:        GitHub + Vercel (automatic)
Monitoring:   Vercel logs (FREE) + Sentry (Phase 2)
Cache:        Skip for MVP (add Upstash Redis free tier if needed)
```

### Monthly Cost Breakdown

**Phase 1 (MVP):**
- Vercel Hosting: **$0** (free tier)
- Supabase Database + Storage: **$0** (free tier)
- Tesseract.js OCR: **$0** (open source)
- Domain (optional): **$12/year** (~$1/month)

**Total: $0-1/month** 🎉

**Phase 2 (When You Have Users):**
- Everything above: **$0**
- Email (Resend): **$0-20** (100/day free, then $20/month)
- AI Suggestions (OpenAI): **$5-10** (depends on usage)
- Better OCR: **$0-10** (free tiers usually enough)

**Total: $5-30/month** (only when you have paying users)

---

### Technology Justification for Hobby Project

**Why This Stack:**

1. **$0 to Deploy MVP**
   - Everything uses free tiers
   - No credit card required (except domain)
   - Can have 1-100 users on free tier

2. **Next.js All-in-One**
   - Frontend + Backend in one project
   - No complex deployment
   - Push to GitHub = live in production

3. **Supabase = Database + Storage + Auth**
   - One service for three needs
   - 500MB database enough for thousands of recipes
   - 2GB storage enough for hundreds of photos

4. **Tesseract.js = Free OCR**
   - Trade accuracy for zero cost
   - Users can edit results anyway
   - Good enough for MVP validation

5. **Defer Expensive Features**
   - AI suggestions: Phase 2 (need recipes first anyway)
   - Email: Phase 2 (not critical for first user)
   - Focus on core: upload, save, view recipes

---

### Development Phases

**Phase 1: Free MVP (Weeks 1-6)**
- Next.js project setup
- Supabase database + auth
- Basic recipe CRUD
- Photo upload + Tesseract OCR
- Manual text editing
- Deploy to Vercel
- **Goal: 1 real user**

**Phase 2: Enhanced Features (Months 2-3)**
- Add email service (when needed)
- Implement basic AI suggestions
- Recipe filtering
- Better OCR if needed
- **Goal: 10-50 users**

**Phase 3: Scale (Months 4+)**
- Family features
- Advanced AI
- Mobile app
- Upgrade paid tiers as needed
- **Goal: 100+ users**

---

### Free Tier Upgrade Paths

**When to Upgrade:**

1. **Vercel ($20/month):**
   - Exceeding 100GB bandwidth
   - Need team collaboration
   - Want advanced analytics

2. **Supabase ($25/month):**
   - Exceeding 500MB database (thousands of recipes)
   - Exceeding 2GB storage (hundreds of photos)
   - Need daily backups

3. **Add AI ($10-20/month):**
   - When users actively request suggestions
   - When you have 50+ recipes per user

**Rule of Thumb:** Don't pay until users prove they want it!

---

## Decision Log

### [Date] - [Technology Category]
**Decision:** [What was chosen]  
**Rationale:** [Why this choice was made]  
**Team Agreement:** [Who approved/agreed]
