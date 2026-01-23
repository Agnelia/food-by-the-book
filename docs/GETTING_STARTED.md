# Getting Started - AI Implementation Guide

This guide provides step-by-step instructions for AI-assisted development of FoodByTheBook.

**Current Status:** 📋 Planning Complete → Ready for Implementation

---

## Prerequisites

Before starting, ensure these tools are installed:

### Required Tools
- [ ] **Node.js 18+** - For frontend development
- [ ] **npm or yarn** - Package manager
- [ ] **.NET 8 SDK** - For Azure Functions backend
- [ ] **Azure Functions Core Tools v4** - Local function runtime
- [ ] **Git** - Version control
- [ ] **VS Code** - Recommended IDE

### Recommended VS Code Extensions
- [ ] C# (ms-dotnettools.csharp)
- [ ] Azure Functions (ms-azuretools.vscode-azurefunctions)
- [ ] ESLint (dbaeumer.vscode-eslint)
- [ ] Prettier (esbenp.prettier-vscode)
- [ ] TypeScript and JavaScript (built-in)

### Cloud Services Setup
- [ ] **Supabase Account** - Database and storage (free tier)
- [ ] **Microsoft Entra ID** - OAuth app registration (free)
- [ ] **Azure Account** - Functions hosting (free tier)

---

## Project Structure Overview

```
FoodByTheBook/
├── docs/                          # ✅ Complete
│   ├── GETTING_STARTED.md        # ⬅️ You are here
│   ├── CODE_PATTERNS.md          # ✅ Complete
│   ├── ARCHITECTURE.md           # ✅ Complete
│   ├── TECH_STACK.md             # ✅ Complete
│   ├── REQUIREMENTS.md           # ✅ Complete
│   ├── DATA_MODELS.md            # ✅ Complete
│   ├── INFRASTRUCTURE.md         # ✅ Complete
│   └── PROJECT_PLAN.md           # ✅ Complete
├── frontend/                      # 🚧 TO BUILD
│   ├── src/
│   ├── public/
│   ├── package.json
│   ├── vite.config.ts
│   ├── tsconfig.json
│   └── .env.example
├── backend/                       # 🚧 TO BUILD
│   ├── Functions/
│   ├── Services/
│   ├── Repositories/
│   ├── Models/
│   ├── backend.csproj
│   ├── host.json
│   ├── Program.cs
│   └── local.settings.example.json
├── infrastructure/                # 🚧 TO BUILD
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── terraform.tfvars.example
└── README.md                      # ✅ Complete
```

---

## Implementation Order

Follow this sequence for optimal AI-assisted development:

### Phase 1: Project Scaffolding ⏳ CURRENT PHASE

#### Step 1.1: Frontend Setup
```bash
cd frontend
npm create vite@latest . -- --template react-ts
npm install
npm install react-router-dom @tanstack/react-query axios
npm install -D @types/react-router-dom
```

**Files to create:**
- [ ] `frontend/src/types/` - TypeScript type definitions
- [ ] `frontend/src/components/ui/` - UI component wrappers
- [ ] `frontend/src/hooks/` - Custom React hooks
- [ ] `frontend/src/services/` - API services
- [ ] `frontend/src/pages/` - Page components
- [ ] `frontend/.env.example` - Environment variables template
- [ ] `frontend/vite.config.ts` - Vite configuration with path aliases

**Reference:** See [CODE_PATTERNS.md](CODE_PATTERNS.md#project-structure) for exact structure

#### Step 1.2: Backend Setup
```bash
mkdir backend
cd backend
func init . --worker-runtime dotnet-isolated --target-framework net8.0
```

**Files to create:**
- [ ] `backend/Functions/` - HTTP trigger functions
- [ ] `backend/Services/` - Business logic
- [ ] `backend/Repositories/` - Data access
- [ ] `backend/Models/` - Domain models and DTOs
- [ ] `backend/Program.cs` - Dependency injection setup
- [ ] `backend/local.settings.example.json` - Configuration template

**Reference:** See [CODE_PATTERNS.md](CODE_PATTERNS.md#backend-c-patterns)

#### Step 1.3: Database Setup
- [ ] Create Supabase project
- [ ] Run SQL migrations from `database/migrations/`
- [ ] Configure connection string

**Reference:** See [DATA_MODELS.md](DATA_MODELS.md) for schema

---

### Phase 2: Authentication 🔐

#### Step 2.1: Microsoft OAuth Setup
- [ ] Register app in Microsoft Entra ID
- [ ] Configure redirect URIs
- [ ] Get client ID and secret
- [ ] Document in `.env.example`

#### Step 2.2: Frontend Auth Implementation
- [ ] Install `@azure/msal-browser`
- [ ] Create `AuthContext` and provider
- [ ] Implement login/logout flows
- [ ] Add protected routes

**Reference:** See [ARCHITECTURE.md](ARCHITECTURE.md#authentication--authorization)

#### Step 2.3: Backend Auth Implementation
- [ ] Add JWT validation middleware
- [ ] Create `AuthFunctions.cs` for token exchange
- [ ] Implement user creation/lookup

---

### Phase 3: Recipe CRUD Operations 📝

#### Step 3.1: Backend API
- [ ] Create `Recipe` model
- [ ] Implement `IRecipeRepository` and `RecipeRepository`
- [ ] Implement `IRecipeService` and `RecipeService`
- [ ] Create `RecipesFunctions` with HTTP triggers:
  - GET /api/recipes (list)
  - POST /api/recipes (create)
  - GET /api/recipes/{id} (read)
  - PUT /api/recipes/{id} (update)
  - DELETE /api/recipes/{id} (delete)

**Reference:** See [CODE_PATTERNS.md](CODE_PATTERNS.md#backend-c-patterns) for exact patterns

#### Step 3.2: Frontend Implementation
- [ ] Create `recipeService.ts` API client
- [ ] Create `useRecipes` and `useRecipe` hooks
- [ ] Create Recipe list page
- [ ] Create Recipe detail page
- [ ] Create Recipe create/edit form

**Reference:** See [CODE_PATTERNS.md](CODE_PATTERNS.md#api-service-pattern)

---

### Phase 4: Image Upload & OCR 📸

#### Step 4.1: Image Upload
- [ ] Create `ImageUpload` component
- [ ] Implement image compression (client-side)
- [ ] Add upload to Supabase Storage
- [ ] Generate thumbnails

#### Step 4.2: OCR Integration
- [ ] Install Tesseract.js
- [ ] Create `useOCR` hook
- [ ] Extract text from images (client-side)
- [ ] Add text editing interface

**Reference:** See [TECH_STACK.md](TECH_STACK.md#ocr-optical-character-recognition)

---

### Phase 5: Testing & Deployment 🚀

#### Step 5.1: Testing
- [ ] Add unit tests (backend services)
- [ ] Add component tests (React Testing Library)
- [ ] Manual testing checklist

#### Step 5.2: Deployment
- [ ] Configure Azure Static Web App
- [ ] Deploy Azure Functions
- [ ] Set up CI/CD with GitHub Actions
- [ ] Configure environment variables

**Reference:** See [INFRASTRUCTURE.md](INFRASTRUCTURE.md)

---

## Development Workflow

### Daily Development Cycle
1. Pull latest changes: `git pull`
2. Start backend: `cd backend && func start`
3. Start frontend: `cd frontend && npm run dev`
4. Make changes following [CODE_PATTERNS.md](CODE_PATTERNS.md)
5. Test locally
6. Commit with clear message
7. Push to feature branch
8. Create pull request

### Code Quality Checklist
Before committing, verify:
- [ ] Code follows patterns in [CODE_PATTERNS.md](CODE_PATTERNS.md)
- [ ] Functions are < 30 lines
- [ ] Components are < 150 lines
- [ ] All types are defined (no `any`)
- [ ] No console.logs in production code
- [ ] Error handling is implemented
- [ ] Code is readable without excessive comments

---

## Quick Start Commands

### Frontend Development
```bash
cd frontend
npm install          # Install dependencies
npm run dev          # Start dev server (http://localhost:5173)
npm run build        # Build for production
npm run preview      # Preview production build
```

### Backend Development
```bash
cd backend
dotnet restore       # Restore packages
func start          # Start local functions (http://localhost:7071)
func host start     # Alternative start command
```

### Database
```bash
# Connect to Supabase PostgreSQL
psql "postgresql://[connection-string]"
```

---

## Troubleshooting

### Frontend Issues
**Problem:** `Module not found` errors
- Solution: Check path aliases in `vite.config.ts`
- Verify imports use `@/` prefix correctly

**Problem:** CORS errors calling API
- Solution: Check Azure Functions CORS configuration
- Verify API_URL in `.env.local`

### Backend Issues
**Problem:** Function won't start
- Solution: Check `local.settings.json` exists
- Verify .NET 8 SDK is installed: `dotnet --version`

**Problem:** Database connection fails
- Solution: Verify connection string format
- Check Supabase allows connection from your IP

---

## Next Steps

**Current Task:** Phase 1.1 - Frontend Setup

Start with:
1. Initialize Vite project in `frontend/`
2. Set up folder structure per [CODE_PATTERNS.md](CODE_PATTERNS.md#project-structure)
3. Create base configuration files
4. Install dependencies
5. Create first component following patterns

**AI Prompt Suggestion:**
```
Based on CODE_PATTERNS.md, create the frontend folder structure for FoodByTheBook.
Initialize a Vite + React + TypeScript project with:
- Proper folder structure (components/ui, pages, hooks, services, types)
- Path aliases configured in vite.config.ts
- Base TypeScript types from DATA_MODELS.md
- Empty barrel exports (index.ts files)

Follow the exact structure and patterns defined in the documentation.
```

---

## Documentation References

All patterns and specifications are documented:
- **Code Style:** [CODE_PATTERNS.md](CODE_PATTERNS.md)
- **System Design:** [ARCHITECTURE.md](ARCHITECTURE.md)
- **Technology Choices:** [TECH_STACK.md](TECH_STACK.md)
- **Features:** [REQUIREMENTS.md](REQUIREMENTS.md)
- **Database:** [DATA_MODELS.md](DATA_MODELS.md)
- **Deployment:** [INFRASTRUCTURE.md](INFRASTRUCTURE.md)
- **Roadmap:** [PROJECT_PLAN.md](PROJECT_PLAN.md)

**Rule:** Always reference these docs when implementing. They are the source of truth.

---

## Support & Resources

### Official Documentation
- [Vite](https://vitejs.dev/)
- [React](https://react.dev/)
- [Azure Functions (.NET)](https://learn.microsoft.com/en-us/azure/azure-functions/)
- [Supabase](https://supabase.com/docs)
- [Microsoft Identity Platform](https://learn.microsoft.com/en-us/azure/active-directory/develop/)

### Project-Specific Help
- Refer to CODE_PATTERNS.md for implementation examples
- Check ARCHITECTURE.md for system design decisions
- Review TECH_STACK.md for technology rationale

---

**Status:** Ready for AI-assisted implementation! 🚀

Start with Phase 1.1 and follow the implementation order sequentially.
