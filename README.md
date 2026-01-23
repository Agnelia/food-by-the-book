# FoodByTheBook

**Meal Planning & Recipe Management with AI-Powered OCR**

Transform your family's paper recipe collection into a digital, searchable cookbook. Upload photos of handwritten or printed recipes, and let AI extract the text automatically.

## 🎯 Vision
Make meal planning effortless by digitizing paper recipes with AI, enabling families to rediscover forgotten recipes and plan meals together.

## 🏗️ Tech Stack
- **Frontend:** Vite + React 18 + React Router + TypeScript
- **Backend:** Azure Functions (.NET 8 C#)
- **Database:** Supabase PostgreSQL (FREE 500MB)
- **Storage:** Supabase Storage (FREE 2GB)
- **OCR:** Tesseract.js (client-side, FREE)
- **Auth:** Microsoft OAuth 2.0 + JWT
- **Hosting:** Vercel/Netlify (frontend) + Azure Functions (backend) - **$0/month**

## 🎯 MVP Features (Phase 1)
1. ✅ Microsoft OAuth authentication
2. ✅ Recipe CRUD operations (Create, Read, Update, Delete)
3. ✅ Photo upload with OCR text extraction (Tesseract.js)

**Success Metric:** 1 real user + deployed to production

---

## 📚 Documentation

This project follows a structured planning approach. Start here:

1. **[Project Plan & Roadmap](docs/PROJECT_PLAN.md)** - Start here! Overall project plan and checklist
2. **[Requirements](docs/REQUIREMENTS.md)** - Features, user stories, and requirements
3. **[Tech Stack](docs/TECH_STACK.md)** - Technology decisions and comparisons
4. **[Architecture](docs/ARCHITECTURE.md)** - System design and architecture
5. **[Data Models](docs/DATA_MODELS.md)** - Database schema and data structures
6. **[Infrastructure](docs/INFRASTRUCTURE.md)** - Azure setup with Terraform (IaC)
7. **[Code Patterns](docs/CODE_PATTERNS.md)** - ⭐ Templates & patterns for AI-assisted development

---

## Quick Start

### Prerequisites
**Frontend:**
- Node.js v18+ (for Vite)
- npm or yarn

**Backend:**
- .NET 8 SDK
- Azure Functions Core Tools v4
- Visual Studio Code with C# extension

**Services:**
- Supabase account (free tier)
- Microsoft Entra ID app registration (free)
- Azure account (free tier)

### Installation
```bash
# Clone repository
git clone <repo-url>
cd FoodByTheBook

# Frontend setup
cd frontend
npm install
cp .env.example .env
# Edit .env with your Azure Functions URL and Microsoft OAuth client ID
npm run dev

# Backend setup (separate terminal)
cd backend
dotnet restore
cp local.settings.example.json local.settings.json
# Edit local.settings.json with your Supabase connection string and Microsoft OAuth credentials
func start
```

### Development
```bash
# Frontend (Vite dev server)
cd frontend
npm run dev
# Opens at http://localhost:5173

# Backend (Azure Functions local runtime)
cd backend
func start
# Runs at http://localhost:7071
```

### Deployment
```bash
# Frontend (Vercel)
cd frontend
npm run build
vercel --prod

# Backend (Azure Functions)
cd backend
func azure functionapp publish <your-function-app-name>
```

---

## Project Status
🎯 **Current Phase:** Planning & Design

See [PROJECT_PLAN.md](docs/PROJECT_PLAN.md) for detailed status and milestones.

---

## Contributing
[Add contribution guidelines if applicable]

---

## License
[Add license information]

---

## Contact
[Add contact information]
