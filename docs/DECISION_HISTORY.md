# Technology Decision History

This document captures the alternatives considered and rationale behind technology choices for the FoodByTheBook project. For the current tech stack, see [TECH_STACK.md](TECH_STACK.md).

---

## Frontend Framework Decision

### Options Considered

| Technology | Pros | Cons | Decision |
|------------|------|------|----------|
| **React** | - Huge ecosystem<br>- Component reusability<br>- Strong community<br>- Great tooling | - Steeper learning curve<br>- Needs additional libraries for routing, state | ✅ **Selected** |
| **Vue.js** | - Easier learning curve<br>- Good documentation<br>- Progressive framework<br>- Single-file components | - Smaller ecosystem than React<br>- Less job market demand | ❌ |
| **Angular** | - Complete framework<br>- TypeScript by default<br>- Enterprise-ready<br>- Opinionated structure | - Steepest learning curve<br>- More verbose<br>- Heavier bundle size | ❌ |
| **Svelte** | - Best performance<br>- Less boilerplate<br>- Smaller bundle size<br>- Easy to learn | - Smaller ecosystem<br>- Fewer job opportunities<br>- Less mature | ❌ |
| **Next.js** | - React with SSR/SSG<br>- Great for SEO<br>- Built-in routing<br>- API routes | - Learning curve<br>- Vendor lock-in | ❌ |

### Why Not Next.js (Anymore)?
- Backend moved to Azure Functions (don't need Next.js API routes)
- Private app for 1 user (don't need SSR/SEO)
- Prefer simplicity and speed for hobby project
- Vite's HMR is significantly faster than Next.js

### Final Decision: Vite + React 18 + React Router
**Rationale:**
- Pure React hooks-based development
- No server-side rendering overhead
- Blazing fast development with Vite
- Lightweight and perfect fit for separated backend (Azure Functions)
- Simple static deployment

---

## UI Component Library Decision

### Options Considered
- Material-UI / MUI
- Ant Design
- Chakra UI
- Tailwind CSS + Headless UI
- Bootstrap
- Shadcn/ui
- Arbetsförmedlingens Designsystem (Digi)

### Final Decision: Arbetsförmedlingens Designsystem (Digi)
**Rationale:**
- Accessibility-focused (WCAG compliant)
- Design tokens for customization
- Swedish government design patterns
- Open source with active development
- Framework-agnostic Web Components

---

## Data Fetching & State Management Decision

### Options Considered & Comparison

#### React Query (TanStack Query)
**Pros:**
- Popular library (40k+ GitHub stars)
- Powerful caching and mutations
- Excellent DevTools
- TypeScript first

**Cons:**
- ❌ Single maintainer (Tanner Linsley / TanStack)
- Organizational backing concern
- Additional library separate from state management

#### SWR (Vercel)
**Pros:**
- Lightweight (~5KB)
- Corporate backing (Vercel)
- Simple API

**Cons:**
- Fewer features than alternatives
- Less comprehensive mutation handling
- Manual cache invalidation

#### RTK Query (Redux Toolkit)
**Pros:**
- ✅ **Organizational backing** (Redux core team)
- ✅ **Enterprise adoption** (Microsoft, Spotify, Amazon)
- ✅ Integrated state + data fetching solution
- Auto-generated hooks from API definitions
- Tag-based cache invalidation
- 10+ years of Redux stability

**Cons:**
- Slightly more setup than SWR
- Larger bundle size (~20KB)

### Final Decision: Redux Toolkit + RTK Query
**Rationale:**
- **Organizational backing requirement** - Redux team vs single maintainer
- Industry standard with proven track record
- Integrated solution (state management + data fetching)
- TypeScript-first with excellent type inference
- Long-term support and enterprise adoption
- **Trade-off accepted:** Slightly more setup for better stability and features

---

## Backend Framework Decision

### Options Considered

| Framework | Pros | Cons | Decision |
|-----------|------|------|----------|
| **Node.js (Express)** | - JavaScript everywhere<br>- Fast development<br>- Large ecosystem<br>- Great for real-time | - Callback hell (if not using async/await)<br>- Less structured | ❌ |
| **Node.js (NestJS)** | - TypeScript first<br>- Angular-like structure<br>- Built-in modules<br>- Great for enterprise | - Learning curve<br>- More opinionated | ❌ |
| **.NET (C#)** | - Type-safe<br>- Excellent tooling<br>- Great performance<br>- Enterprise-ready | - Steeper learning curve<br>- Windows-centric (changing) | ✅ **Selected** |
| **Python (FastAPI)** | - Easy to learn<br>- Great for ML/AI<br>- Fast development<br>- Automatic docs | - Slower than compiled languages<br>- Async still maturing | ❌ |
| **Python (Django)** | - Batteries included<br>- Admin panel<br>- ORM built-in<br>- Mature | - Heavier framework<br>- Less flexible | ❌ |
| **Go** | - Excellent performance<br>- Simple concurrency<br>- Static typing<br>- Fast compilation | - Less web frameworks<br>- Smaller ecosystem | ❌ |

### Final Decision: Azure Functions (.NET 8 C#)
**Rationale:**
- Free tier (1 million executions/month)
- Fast cold starts (~500ms)
- Strong typing matches TypeScript frontend
- Native Microsoft OAuth integration
- Serverless auto-scaling
- Industry-standard microservices pattern

---

## Database Decision

### Options Considered

| Database | Use Case | Pros | Cons | Decision |
|----------|----------|------|------|----------|
| **PostgreSQL** | Relational, complex queries | - Robust<br>- JSON support<br>- ACID compliant<br>- Open source | - Vertical scaling limits<br>- More complex setup | ✅ **Selected** |
| **MySQL/MariaDB** | Relational, traditional | - Mature<br>- Fast reads<br>- Wide support | - Less features than PostgreSQL<br>- Licensing (MySQL) | ❌ |
| **MongoDB** | Document store, flexible schema | - Schema flexibility<br>- Easy to start<br>- Horizontal scaling | - Less structured<br>- Memory intensive | ❌ |
| **SQLite** | Lightweight, embedded | - Serverless<br>- Zero config<br>- Great for dev | - Not for production at scale<br>- Limited concurrency | ❌ |

### Hosting Provider Comparison

#### Supabase (Selected)
- ✅ 500MB database FREE
- ✅ 2GB storage FREE
- ✅ PostgreSQL + authentication + storage in one
- ✅ Realtime subscriptions included
- ✅ No server management

#### Alternatives Considered
- **Railway:** 500MB free, then $5/month
- **Neon:** 512MB free, serverless PostgreSQL
- **Vercel Postgres:** $0.25/month minimum

### Final Decision: PostgreSQL on Supabase
**Rationale:**
- Free tier sufficient for MVP (500MB)
- Hosted PostgreSQL with no management overhead
- Built-in authentication option
- Storage included for images
- Easy migration path if needed

---

## Authentication Decision

### Options Considered
- **Email/Password:** More work, security concerns, password reset complexity
- **Auth0:** $23+/month for features not needed
- **Google OAuth:** Good, but Microsoft fits better with Azure
- **GitHub OAuth:** Works for developers only
- **Microsoft OAuth:** FREE, native Azure integration

### Final Decision: Microsoft OAuth 2.0 + JWT
**Rationale:**
- 100% FREE (no per-user costs)
- No password management burden
- OAuth 2.0 industry standard
- Native C# library integration with Azure Functions
- Users likely already have Microsoft accounts

---

## OCR Service Decision

### Comparison: Tesseract.js vs Cloud APIs

| Feature | Tesseract.js | Azure Vision | Google Vision |
|---------|-------------|--------------|---------------|
| **Cost** | FREE | ~$1.50/1k images | ~$1.50/1k images |
| **Accuracy** | 70-85% | 95%+ | 95%+ |
| **Speed** | 5-10 seconds | 1-2 seconds | 1-2 seconds |
| **Privacy** | Client-side (images never leave device) | Cloud processing | Cloud processing |
| **Setup** | Install npm package | Azure account + API keys | GCP account + API keys |

### Final Decision: Tesseract.js (Client-Side)
**Rationale:**
- ✅ 100% FREE for hobby project
- ✅ Privacy - images never leave user's device
- ✅ No server load
- ⚠️ **Trade-off accepted:** Lower accuracy (70-85% vs 95%)
- ⚠️ **Trade-off accepted:** Slower processing (5-10s vs 1-2s)
- **Mitigation:** User manually corrects mistakes (acceptable for 1 user MVP)

### Future Upgrade Path (if budget allows)
- OCR.space API (free tier: 500 requests/day)
- Google Cloud Vision (free tier: 1000 requests/month)
- Azure Computer Vision

---

## Frontend Hosting Decision

### Options Compared

| Feature | Azure Static Web Apps | Vercel | Netlify |
|---------|----------------------|--------|---------|
| **Free Bandwidth** | 100GB | 100GB | 100GB |
| **Build Minutes** | Unlimited | 6000 | 300 |
| **Deploy Speed** | ~2-3 min | ~1-2 min | ~2-3 min |
| **Azure Integration** | ⭐ Native | Via CORS | Via CORS |
| **Custom Domains** | ✅ Free SSL | ✅ Free SSL | ✅ Free SSL |
| **Preview Deployments** | ✅ Per PR | ✅ Per PR | ✅ Per PR |
| **DX (Developer Experience)** | Good | ⭐ Excellent | Good |
| **Analytics** | ❌ | ✅ Free | ❌ (Paid) |
| **Pro Plan Cost** | $9/mo | $20/mo | $19/mo |

### Vercel Analysis
**Pros:**
- Best developer experience
- Created Vite (perfect optimization)
- Fastest deployments
- Built-in Web Vitals analytics

**Cons:**
- Requires CORS configuration in Azure Functions
- Managing two separate platforms
- $20/month Pro plan

### Netlify Analysis
**Pros:**
- Easiest setup (drag-and-drop)
- Built-in form handling
- A/B testing built-in

**Cons:**
- Requires CORS configuration
- Only 300 build minutes/month free
- $19/month Pro plan

### Azure Static Web Apps Analysis
**Pros:**
- ✅ Same ecosystem as backend (Azure)
- ✅ No CORS complexity
- ✅ Unified billing and management
- ✅ Cheapest Pro plan ($9/month)
- ✅ Managed identity between services

**Cons:**
- Slightly less polished DX than Vercel
- No built-in analytics

### Final Decision: Azure Static Web Apps
**Rationale:**
1. **Unified platform** - Frontend and backend both in Azure
2. **CORS simplicity** - Same domain possible with routing
3. **Future growth** - Easy to add more Azure services
4. **Cost efficiency** - Cheaper Pro plan than alternatives
5. **Infrastructure as Code** - Terraform manages all Azure resources consistently

---

## Infrastructure as Code Decision

### Why Terraform over Azure CLI/ARM Templates?
- **Cloud-agnostic:** Works with Azure, AWS, GCP (future-proof)
- **Declarative:** Describe desired state, Terraform handles changes
- **Version control:** Infrastructure changes tracked in Git
- **Easy migration:** Change provider to switch hosting platforms
- **State management:** Terraform tracks deployed resources
- **Large community:** More examples and modules than ARM templates

### Final Decision: Terraform
**Rationale:**
- Industry standard IaC tool
- Free and open source
- Future-proof (not locked to Azure)
- Better for multi-cloud scenarios
- Easier to learn than ARM templates

---

## State Management Decision (Historical)

### Options Considered
- **React Hooks (useState, useContext):** Built-in, simple
- **Zustand:** Lightweight, modern
- **Redux Toolkit:** Industry standard, more boilerplate
- **Recoil:** Experimental, by Facebook
- **MobX:** Observable-based, less popular

### Original Decision: React Hooks
Initially selected React hooks for simplicity.

### Revised Decision: Redux Toolkit
Changed to Redux Toolkit when selecting RTK Query to get:
- Unified state management + data fetching
- Single source of truth
- Better DevTools
- Organizational backing

---

## Forms & Validation Decision

### Why React Hook Form + Zod?
- **React Hook Form:** Performance-focused, minimal re-renders
- **Zod:** TypeScript-first schema validation
- **Integration:** Works seamlessly together
- **Bundle size:** ~8KB gzipped
- **Free:** Open source

### Alternatives Not Chosen
- **Formik:** Older, more re-renders
- **React Final Form:** Less TypeScript support
- **Yup:** Similar to Zod but less TypeScript-native

---

## Decision Principles

Throughout this project, technology choices were guided by:

1. **Budget Constraint:** Free tiers and open-source solutions prioritized
2. **Organizational Backing:** Preference for team-maintained over single-developer libraries
3. **TypeScript First:** Strong typing throughout the stack
4. **Separation of Concerns:** Decoupled frontend/backend architecture
5. **Industry Standards:** Proven technologies over experimental ones
6. **Long-term Viability:** Active maintenance and large communities
7. **Developer Experience:** Modern tooling and good documentation
8. **Acceptable Trade-offs:** Lower accuracy/speed accepted for zero cost (OCR)

---

**Last Updated:** February 2026  
**For Current Stack:** See [TECH_STACK.md](TECH_STACK.md)
