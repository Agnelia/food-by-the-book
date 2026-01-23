# System Architecture

## Architecture Overview
**Pattern:** Decoupled Frontend/Backend (Microservices-lite)

**Frontend:** Vite + React 18 + React Router (SPA - Single Page Application)  
**Backend:** Azure Functions (.NET 8 C#) - HTTP-triggered serverless functions  
**Database:** Supabase PostgreSQL  
**Storage:** Supabase Storage (images)  
**Authentication:** Microsoft OAuth 2.0 + JWT  

**Why This Architecture:**
- **Separation of Concerns:** Frontend and backend deploy independently
- **Scalability:** Each layer scales separately
- **Free Hosting:** Vite static build (Vercel/Netlify) + Azure Functions free tier
- **Technology Freedom:** React for UI, C# for business logic
- **Modern Standard:** Industry-standard microservices pattern

### Architecture Pattern
**Selected:** Decoupled SPA + Serverless API

**Characteristics:**
- Frontend is pure static files (HTML, CSS, JS) deployed to CDN
- Backend is serverless functions (Azure Functions) with HTTP triggers
- Communication via REST API over HTTPS
- Stateless backend (JWT for authentication)
- Database-backed persistence (PostgreSQL)

---

## System Diagram
```
┌──────────────────────────────────────────────────────────────┐
│                         User Browser                          │
└───────────────────────────┬──────────────────────────────────┘
                            │
                            │ 1. Load Static Files
                            ▼
┌──────────────────────────────────────────────────────────────┐
│           CDN (Vercel/Netlify/Azure Static Web Apps)         │
│                                                               │
│  ┌────────────────────────────────────────────────────────┐  │
│  │         Vite React App (Static Build)                  │  │
│  │  - React Router for navigation                         │  │
│  │  - React components with hooks                         │  │
│  │  - Tesseract.js for OCR (client-side)                  │  │
│  │  - SWR for data fetching                               │  │
│  │  - @azure/msal-browser for auth                        │  │
│  └────────────────────────────────────────────────────────┘  │
└───────────────────────────┬──────────────────────────────────┘
                            │
                            │ 2. HTTPS API Calls (with JWT)
                            │    https://your-app.azurewebsites.net/api/*
                            ▼
┌──────────────────────────────────────────────────────────────┐
│                    Azure Functions (C#)                       │
│                                                               │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  HTTP-Triggered Functions (.NET 8 Isolated Worker)     │  │
│  │  ┌──────────────────────────────────────────────────┐  │  │
│  │  │  GET  /api/recipes          (List recipes)       │  │  │
│  │  │  POST /api/recipes          (Create recipe)      │  │  │
│  │  │  GET  /api/recipes/{id}     (Get recipe)         │  │  │
│  │  │  PUT  /api/recipes/{id}     (Update recipe)      │  │  │
│  │  │  DELETE /api/recipes/{id}   (Delete recipe)      │  │  │
│  │  │  POST /api/recipes/{id}/images (Upload image)    │  │  │
│  │  │  POST /api/auth/token       (Exchange OAuth code)│  │  │
│  │  └──────────────────────────────────────────────────┘  │  │
│  │                                                         │  │
│  │  Middleware:                                            │  │
│  │  - JWT validation (Microsoft.Identity.Web)             │  │
│  │  - CORS policy (allow frontend domain)                 │  │
│  │  - Error handling                                       │  │
│  │                                                         │  │
│  │  Services:                                              │  │
│  │  - RecipeService (business logic)                      │  │
│  │  - ImageService (upload to Supabase)                   │  │
│  │  - UserService (user management)                       │  │
│  └────────────────────────────────────────────────────────┘  │
└───────────────────────────┬──────────────────────────────────┘
                            │
                            │ 3. SQL Queries
                            ▼
┌──────────────────────────────────────────────────────────────┐
│                    Supabase (PostgreSQL)                      │
│                                                               │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────────────┐    │
│  │   Database   │  │   Storage    │  │   Auth (OAuth)  │    │
│  │ - Users      │  │ - Images     │  │ - Microsoft     │    │
│  │ - Recipes    │  │ - 2GB FREE   │  │   Integration   │    │
│  │ - Ratings    │  └──────────────┘  └─────────────────┘    │
│  │ - 500MB FREE │                                            │
│  └──────────────┘                                            │
└──────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────┐
│                  Microsoft Entra ID (Azure AD)                │
│                  OAuth 2.0 + OpenID Connect                   │
│                        (100% FREE)                            │
└──────────────────────────────────────────────────────────────┘
```

---

## Frontend Architecture (Vite + React)

### Folder Structure
```
/
├── src/
│   ├── components/          # Reusable UI components
│   │   ├── ui/              # Abstraction layer (UIButton, UIInput, etc.)
│   │   │   ├── Button.tsx
│   │   │   ├── Input.tsx
│   │   │   ├── Card.tsx
│   │   │   └── index.ts     # Barrel export
│   │   ├── lib/             # Third-party adapters
│   │   │   └── digi-adapter.ts
│   │   ├── layout/          # Layout components
│   │   │   ├── Header.tsx
│   │   │   ├── Footer.tsx
│   │   │   └── Layout.tsx
│   │   └── common/          # Shared components
│   │       ├── RecipeCard.tsx
│   │       ├── ImageUpload.tsx
│   │       └── Loading.tsx
│   ├── pages/               # Page components (routes)
│   │   ├── Home.tsx
│   │   ├── Login.tsx
│   │   ├── Recipes.tsx
│   │   ├── RecipeDetail.tsx
│   │   ├── RecipeCreate.tsx
│   │   └── NotFound.tsx
│   ├── hooks/               # Custom React hooks
│   │   ├── useAuth.ts
│   │   ├── useRecipes.ts
│   │   ├── usePhotoUpload.ts
│   │   └── useOCR.ts
│   ├── services/            # API services
│   │   ├── api.ts           # Axios/fetch wrapper
│   │   ├── recipeService.ts
│   │   ├── authService.ts
│   │   └── imageService.ts
│   ├── utils/               # Utility functions
│   │   ├── formatters.ts
│   │   ├── validators.ts
│   │   └── constants.ts
│   ├── types/               # TypeScript types
│   │   ├── recipe.ts
│   │   ├── user.ts
│   │   └── api.ts
│   ├── contexts/            # React contexts (if needed)
│   │   └── AuthContext.tsx
│   ├── styles/              # Global styles
│   │   └── index.css
│   ├── App.tsx              # Main app component with router
│   ├── main.tsx             # Entry point
│   └── vite-env.d.ts        # Vite type definitions
├── public/                  # Static assets
│   ├── favicon.ico
│   └── images/
├── index.html               # HTML entry point
├── vite.config.ts           # Vite configuration
├── tsconfig.json            # TypeScript configuration
├── package.json
└── .env.example             # Environment variables template
```

### Routing (React Router)
```tsx
// src/App.tsx
import { BrowserRouter, Routes, Route, Navigate } from 'react-router-dom';
import { useAuth } from './hooks/useAuth';
import Layout from './components/layout/Layout';
import Home from './pages/Home';
import Login from './pages/Login';
import Recipes from './pages/Recipes';
import RecipeDetail from './pages/RecipeDetail';
import RecipeCreate from './pages/RecipeCreate';
import NotFound from './pages/NotFound';

function ProtectedRoute({ children }: { children: React.ReactNode }) {
  const { isAuthenticated } = useAuth();
  return isAuthenticated ? <>{children}</> : <Navigate to="/login" />;
}

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/login" element={<Login />} />
        <Route path="/" element={<Layout />}>
          <Route index element={<Home />} />
          <Route path="recipes" element={
            <ProtectedRoute><Recipes /></ProtectedRoute>
          } />
          <Route path="recipes/:id" element={
            <ProtectedRoute><RecipeDetail /></ProtectedRoute>
          } />
          <Route path="recipes/new" element={
            <ProtectedRoute><RecipeCreate /></ProtectedRoute>
          } />
          <Route path="*" element={<NotFound />} />
        </Route>
      </Routes>
    </BrowserRouter>
  );
}

export default App;
```

### UI Component Abstraction Layer
**Strategy:** Create wrapper components to avoid vendor lock-in and enable easy UI library switching

#### Why Abstract Components?
- **Easy Migration:** Switch UI libraries by changing 10-20 files instead of 100s
- **Consistent API:** Standardize prop names across your app
- **Custom Logic:** Add app-specific behavior without modifying library components
- **Testing:** Mock UI library without importing actual components

#### ⚠️ Components to Abstract vs. Use Directly

**WRAP (create UIComponent):**
- Basic components used everywhere: Button, Input, Card, Select, Modal
- Simple, stable components that justify the wrapping overhead
- Components where you want consistent styling/behavior across the app

**USE DIRECTLY (don't wrap):**
- Very specialized components (e.g., advanced data tables, charts, complex date pickers)
- Components used only once or twice in the entire app
- Third-party components with extensive APIs that would be tedious to wrap
- Feature-rich components where wrapping adds no value

**💡 IMPORTANT:** If you need specialized components NOT available in Arbetsförmedlingens Designsystem (Digi), consider using a more mainstream UI library like:
- **Material-UI (MUI):** Most comprehensive, 2000+ icons, enterprise-ready
- **Ant Design:** Feature-rich, great for data-heavy apps
- **Chakra UI:** Excellent accessibility, simple API
- **Radix UI + Tailwind:** Headless components, full styling control

**Recommended Approach:**
1. Use Digi for basic components (Button, Input, Card) via abstraction layer
2. For specialized needs (DataTable, DateRangePicker, Charts), import directly from mainstream library
3. Don't waste time wrapping components you'll only use once

```tsx
// Example: Mixed approach
import { UIButton, UIInput } from '@/components/ui'; // Wrapped Digi components
import { DataGrid } from '@mui/x-data-grid'; // Specialized MUI component, used directly
import { Calendar } from '@/components/common/Calendar'; // Custom component

function RecipesPage() {
  return (
    <>
      <UIButton>Add Recipe</UIButton>  {/* Wrapped */}
      <DataGrid rows={recipes} />       {/* Direct import - specialized */}
    </>
  );
}
```

#### Folder Structure
```
/src
  /components
    /ui                    # Your abstraction layer (import these everywhere)
      Button.tsx           # Exports UIButton
      Input.tsx            # Exports UIInput
      Card.tsx             # Exports UICard
      Select.tsx           # Exports UISelect
      Modal.tsx            # Exports UIModal
      index.ts             # Barrel exports all UI components
    /lib                   # Third-party library adapters
      digi-adapter.ts      # Arbetsförmedlingens Designsystem wrapper
    /common                # App-specific reusable components
    /layout                # Layout components
```

#### Example: Button Component Abstraction
```tsx
// components/ui/Button.tsx
import { DigiButton } from '@designsystem-se/af';

interface UIButtonProps {
  size?: 'small' | 'medium' | 'large';
  variant?: 'primary' | 'secondary' | 'ghost';
  disabled?: boolean;
  loading?: boolean;
  onClick?: () => void;
  children: React.ReactNode;
  className?: string;
}

export function UIButton({ 
  size = 'medium',
  variant = 'primary',
  disabled,
  loading,
  children,
  ...props 
}: UIButtonProps) {
  return (
    <DigiButton 
      af-size={size}
      af-variation={variant}
      disabled={disabled || loading}
      {...props}
    >
      {loading ? 'Loading...' : children}
    </DigiButton>
  );
}
```

```tsx
// components/ui/index.ts (Barrel Export)
export { UIButton } from './Button';
export { UIInput } from './Input';
export { UICard } from './Card';
export { UISelect } from './Select';
export { UIModal } from './Modal';
// ... export all UI components
```

#### Usage in Application
```tsx
// ✅ GOOD: Import from abstraction layer
import { UIButton, UIInput, UICard } from '@/components/ui';

function MyPage() {
  return (
    <UICard>
      <UIInput placeholder="Enter text" />
      <UIButton size="medium" variant="primary">
        Submit
      </UIButton>
    </UICard>
  );
}

// ❌ BAD: Direct import from UI library
import { DigiButton } from '@designsystem-se/af';

function MyPage() {
  return (
    <DigiButton af-size="medium" af-variation="primary">
      Submit
    </DigiButton>
  );
}
```

#### Switching UI Libraries
When switching from Digi to another library (e.g., Material-UI):

```tsx
// components/ui/Button.tsx
// import { DigiButton } from '@designsystem-se/af';  // Old
import { Button as MuiButton } from '@mui/material';   // New

export function UIButton({ size, variant, children, ...props }: UIButtonProps) {
  // Map your props to new library's API
  return (
    <MuiButton 
      size={size}
      variant={variant}
      {...props}
    >
      {children}
    </MuiButton>
  );
}
```

**Result:** Rest of your app (~100s of files) remains unchanged ✨

#### Naming Convention: UI Prefix

**All wrapped components use the `UI` prefix:**
- `UIButton` instead of `Button`
- `UIInput` instead of `Input`
- `UICard` instead of `Card`

**Why UI Prefix:**
- ✅ Clear distinction from native HTML elements (`<button>` vs `<UIButton>`)
- ✅ Shows ownership - these are YOUR components
- ✅ Industry standard pattern
- ✅ Easy for AI tools to identify and use consistently
- ✅ Prevents naming conflicts

**Import Pattern:**
```tsx
// Always import from barrel export
import { UIButton, UIInput, UICard } from '@/components/ui';

// NOT from individual files
import { UIButton } from '@/components/ui/Button';  // ❌
```

#### Components to Abstract

**AI-Assisted Development Strategy:**
With AI code generation, wrap ALL standard components upfront (Day 1). The cost of wrapping 20+ components is minimal with AI assistance, and provides consistency from the start.

**Wrap Immediately (Generate All Upfront):**
- [ ] UIButton
- [ ] UIInput / UITextField
- [ ] UISelect / UIDropdown
- [ ] UICheckbox
- [ ] UIRadio
- [ ] UIModal / UIDialog
- [ ] UICard
- [ ] UILoading / UISpinner
- [ ] UITable
- [ ] UITabs
- [ ] UIAccordion
- [ ] UITooltip
- [ ] UIBadge
- [ ] UIAlert / UIToast
- [ ] UIAvatar
- [ ] UIChip / UITag
- [ ] UIPagination
- [ ] UIBreadcrumbs

**Don't Wrap (Build Custom):**
- Complex, highly-customized domain components (RecipeCard, CookingTimer)
- Components with heavy business logic
- One-off specialized components

**Benefits of Upfront Wrapping with AI:**
- AI always uses YOUR components, never library directly
- Consistent API from day 1
- No risk of accidentally using library components
- Easy to switch libraries later

#### Guidelines
1. **Import Rule:** Only `/components/ui` may import from UI library
2. **Naming Convention:** All wrapped components use `UI` prefix (`UIButton`, `UIInput`)
3. **Barrel Exports:** Always import from `@/components/ui`, never from individual files
4. **Prop Names:** Use semantic names (`variant` not `af-variation`)

---

### Data Fetching Abstraction Layer
**Strategy:** Create custom hooks to avoid vendor lock-in with data fetching libraries (SWR, React Query)

#### Why Abstract Data Fetching?
- **Easy Migration:** Switch from SWR to React Query by changing hook files only
- **Consistent API:** Standard interface regardless of underlying library
- **Testing:** Mock data fetching without importing actual libraries
- **Business Logic:** Centralize data transformation, error handling, retry logic

#### Folder Structure
```
/src
  /hooks                    # Data fetching abstraction layer
    useRecipes.ts           # Recipe CRUD operations
    useAuth.ts              # Authentication state
    usePhotoUpload.ts       # Image upload with OCR
    useOCR.ts               # OCR processing
    index.ts                # Barrel exports
  /services                 # API layer (used by hooks)
    api.ts                  # Axios/fetch wrapper
    recipeService.ts        # Recipe API calls
    authService.ts          # Auth API calls
```

#### Example: Recipe Data Hook Abstraction

```tsx
// hooks/useRecipes.ts - Abstraction layer
import useSWR from 'swr';
import { recipeService } from '@/services/recipeService';

export function useRecipes() {
  const { data, error, isLoading, mutate } = useSWR(
    '/api/recipes',
    recipeService.getAll
  );
  
  return {
    recipes: data || [],
    isLoading,
    isError: !!error,
    error,
    refresh: mutate  // ← Abstract library-specific names
  };
}

export function useRecipe(id: string) {
  const { data, error, isLoading, mutate } = useSWR(
    id ? `/api/recipes/${id}` : null,
    () => recipeService.getById(id)
  );
  
  return {
    recipe: data,
    isLoading,
    isError: !!error,
    error,
    refresh: mutate
  };
}
```

#### Usage in Components
```tsx
// ✅ GOOD: Import from custom hooks
import { useRecipes, useRecipe } from '@/hooks';

function RecipesList() {
  const { recipes, isLoading, refresh } = useRecipes();
  
  if (isLoading) return <UILoading />;
  
  return (
    <div>
      <UIButton onClick={refresh}>Refresh</UIButton>
      {recipes.map(recipe => <RecipeCard key={recipe.id} recipe={recipe} />)}
    </div>
  );
}

// ❌ BAD: Direct import from data fetching library
import useSWR from 'swr';

function RecipesList() {
  const { data, isLoading, mutate } = useSWR('/api/recipes', fetcher);
  // If you switch to React Query, you need to change this everywhere!
}
```

#### Switching Data Fetching Libraries
When switching from SWR to React Query:

```tsx
// hooks/useRecipes.ts - Only change this file
// import useSWR from 'swr';  // Old
import { useQuery } from '@tanstack/react-query';  // New

export function useRecipes() {
  // const { data, error, isLoading, mutate } = useSWR(...)  // Old
  const { data, error, isLoading, refetch } = useQuery({     // New
    queryKey: ['recipes'],
    queryFn: recipeService.getAll
  });
  
  return {
    recipes: data || [],
    isLoading,
    isError: !!error,
    error,
    refresh: refetch  // ← Same interface, different implementation
  };
}
```

**Result:** All components using `useRecipes()` work without changes! ✨

#### Data Fetching Hooks Pattern

**Standard Hook Interface:**
```tsx
// All data hooks should return consistent structure
interface DataHookReturn<T> {
  data: T | undefined;
  isLoading: boolean;
  isError: boolean;
  error: Error | undefined;
  refresh: () => void;
}

// Example
function useRecipes(): DataHookReturn<Recipe[]> { ... }
function useRecipe(id: string): DataHookReturn<Recipe> { ... }
```

**Mutation Hooks:**
```tsx
// hooks/useRecipeMutations.ts
export function useCreateRecipe() {
  const { recipes, refresh } = useRecipes();
  
  async function createRecipe(recipe: CreateRecipeRequest) {
    // Optimistic update (optional)
    // ...
    
    // Call API
    const newRecipe = await recipeService.create(recipe);
    
    // Refresh list
    refresh();
    
    return newRecipe;
  }
  
  return { createRecipe };
}
```

#### Data Fetching Hooks to Create

**Core Data Hooks:**
- [ ] `useRecipes()` - List all recipes
- [ ] `useRecipe(id)` - Get single recipe
- [ ] `useCreateRecipe()` - Create recipe mutation
- [ ] `useUpdateRecipe()` - Update recipe mutation
- [ ] `useDeleteRecipe()` - Delete recipe mutation
- [ ] `useAuth()` - Authentication state
- [ ] `useUser()` - Current user data

**Feature-Specific Hooks:**
- [ ] `usePhotoUpload()` - Upload image to Supabase
- [ ] `useOCR()` - Process image with Tesseract.js
- [ ] `useRecipeSearch(query)` - Search recipes
- [ ] `useRecipeImages(recipeId)` - Get recipe images

#### Guidelines
1. **Import Rule:** Only `/hooks` may import data fetching libraries (SWR/React Query)
2. **Naming Convention:** Use standard hook pattern (`useRecipes`, `useRecipe`, etc.)
3. **Barrel Exports:** Always import from `@/hooks`, never from individual hook files
4. **Consistent Interface:** All hooks return `{ data, isLoading, isError, error, refresh }`
5. **Service Layer:** Hooks call API services, never fetch directly
6. **No Library in Components:** Components never import SWR/React Query directly

**Benefits:**
- Switch libraries: Change 5-10 hook files vs. 50+ components
- Migration time: 1-2 hours vs. 1-2 days
- Same pattern as UI component abstraction
- Easy to test: Mock hooks instead of fetch library
5. **TypeScript:** Strong typing for all wrapper props (e.g., `UIButtonProps`)
6. **Documentation:** Document prop mapping in comments
7. **Testing:** Test wrappers independently of UI library
8. **AI Instructions:** Instruct AI to ONLY use UI-prefixed components, never library directly

### State Management Strategy
[Describe how you'll manage state]
- **Global State:** [Redux, Zustand, etc.] for [what data]
- **Server State:** [React Query, SWR] for [API data]
- **Local State:** [useState] for [component-specific data]
- **URL State:** [React Router] for [navigation state]

---

## Backend Architecture

### Folder Structure
```
/src
  /controllers      # Request handlers
  /services         # Business logic
  /models           # Data models
  /repositories     # Data access layer
  /middleware       # Express middleware
  /routes           # API routes
  /config           # Configuration
  /utils            # Utility functions
  /types            # TypeScript types
  /validators       # Input validation
  /tests            # Test files
  server.ts
```

### API Design

#### REST API
```
Base URL: https://api.yourapp.com/v1

Authentication:
  POST   /auth/register
  POST   /auth/login
  POST   /auth/logout
  POST   /auth/refresh

Users:
  GET    /users
  GET    /users/:id
  PUT    /users/:id
  DELETE /users/:id

[Add your specific endpoints]
```

#### GraphQL (if applicable)
```graphql
type Query {
  user(id: ID!): User
  users: [User]
}

type Mutation {
  createUser(input: CreateUserInput!): User
  updateUser(id: ID!, input: UpdateUserInput!): User
}
```

### Layered Architecture
```
┌──────────────────────┐
│   Controllers        │  <- HTTP handling
├──────────────────────┤
│   Services           │  <- Business logic
├──────────────────────┤
│   Repositories       │  <- Data access
├──────────────────────┤
│   Models             │  <- Data structures
└──────────────────────┘
```

---

## Database Architecture

### Schema Design
[List your main entities and relationships]

```sql
-- Example User table
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Add your tables here
```

### Entity Relationships
```
[Add ER diagram or description]

User (1) ──── (Many) Posts
User (1) ──── (Many) Comments
Post (1) ──── (Many) Comments
```

### Indexes
[List important indexes for performance]
- `users.email` - Unique index for login lookups
- `posts.user_id` - Foreign key index
- [Add your indexes]

### Migrations Strategy
- [ ] Use migration tool (Prisma, TypeORM, Alembic, etc.)
- [ ] Version control migrations
- [ ] Rollback capability
- [ ] Seed data for development

---

## Backend Architecture (Azure Functions + C#)

### Project Structure
```
/backend
├── Functions/                   # HTTP-triggered function endpoints
│   ├── RecipesFunctions.cs      # Recipe CRUD operations
│   ├── AuthFunctions.cs         # Authentication endpoints
│   └── ImageFunctions.cs        # Image upload operations
├── Services/                    # Business logic layer
│   ├── IRecipeService.cs
│   ├── RecipeService.cs
│   ├── IImageService.cs
│   ├── ImageService.cs
│   ├── IUserService.cs
│   └── UserService.cs
├── Repositories/                # Data access layer
│   ├── IRecipeRepository.cs
│   ├── RecipeRepository.cs
│   ├── IUserRepository.cs
│   └── UserRepository.cs
├── Models/                      # DTOs and domain models
│   ├── Recipe.cs
│   ├── User.cs
│   ├── CreateRecipeRequest.cs
│   └── RecipeResponse.cs
├── Data/                        # Database context
│   └── ApplicationDbContext.cs  # Entity Framework Core context
├── Middleware/                  # Custom middleware
│   ├── JwtMiddleware.cs
│   └── ErrorHandlingMiddleware.cs
├── Extensions/                  # Extension methods
│   └── ServiceCollectionExtensions.cs
├── Program.cs                   # Startup configuration
├── host.json                    # Function app configuration
├── local.settings.json          # Local environment variables
└── backend.csproj              # Project file
```

### Function Examples

#### Recipe CRUD Functions
```csharp
// Functions/RecipesFunctions.cs
using Microsoft.Azure.Functions.Worker;
using Microsoft.Azure.Functions.Worker.Http;
using Microsoft.Extensions.Logging;

public class RecipesFunctions
{
    private readonly IRecipeService _recipeService;
    private readonly ILogger<RecipesFunctions> _logger;

    public RecipesFunctions(IRecipeService recipeService, ILogger<RecipesFunctions> logger)
    {
        _recipeService = recipeService;
        _logger = logger;
    }

    [Function("GetRecipes")]
    public async Task<HttpResponseData> GetRecipes(
        [HttpTrigger(AuthorizationLevel.Anonymous, "get", Route = "recipes")] HttpRequestData req)
    {
        // JWT validation happens in middleware
        var userId = req.GetUserId(); // Extension method
        var recipes = await _recipeService.GetRecipesByUserAsync(userId);
        
        var response = req.CreateResponse(HttpStatusCode.OK);
        await response.WriteAsJsonAsync(recipes);
        return response;
    }

    [Function("CreateRecipe")]
    public async Task<HttpResponseData> CreateRecipe(
        [HttpTrigger(AuthorizationLevel.Anonymous, "post", Route = "recipes")] HttpRequestData req)
    {
        var userId = req.GetUserId();
        var createRequest = await req.ReadFromJsonAsync<CreateRecipeRequest>();
        
        var recipe = await _recipeService.CreateRecipeAsync(userId, createRequest);
        
        var response = req.CreateResponse(HttpStatusCode.Created);
        await response.WriteAsJsonAsync(recipe);
        return response;
    }

    [Function("GetRecipeById")]
    public async Task<HttpResponseData> GetRecipeById(
        [HttpTrigger(AuthorizationLevel.Anonymous, "get", Route = "recipes/{id}")] HttpRequestData req,
        string id)
    {
        var userId = req.GetUserId();
        var recipe = await _recipeService.GetRecipeByIdAsync(userId, id);
        
        if (recipe == null)
        {
            return req.CreateResponse(HttpStatusCode.NotFound);
        }
        
        var response = req.CreateResponse(HttpStatusCode.OK);
        await response.WriteAsJsonAsync(recipe);
        return response;
    }

    // PUT, DELETE endpoints follow similar pattern...
}
```

#### Dependency Injection Setup
```csharp
// Program.cs
using Microsoft.Azure.Functions.Worker;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using Microsoft.EntityFrameworkCore;

var host = new HostBuilder()
    .ConfigureFunctionsWorkerDefaults(builder =>
    {
        builder.UseMiddleware<JwtMiddleware>();
        builder.UseMiddleware<ErrorHandlingMiddleware>();
    })
    .ConfigureServices((context, services) =>
    {
        // Database (Supabase PostgreSQL)
        services.AddDbContext<ApplicationDbContext>(options =>
            options.UseNpgsql(context.Configuration["SupabaseConnectionString"]));
        
        // Services
        services.AddScoped<IRecipeService, RecipeService>();
        services.AddScoped<IImageService, ImageService>();
        services.AddScoped<IUserService, UserService>();
        
        // Repositories
        services.AddScoped<IRecipeRepository, RecipeRepository>();
        services.AddScoped<IUserRepository, UserRepository>();
        
        // CORS
        services.AddCors(options =>
        {
            options.AddPolicy("AllowFrontend", builder =>
            {
                builder.WithOrigins(
                    "http://localhost:5173",  // Vite dev server
                    "https://your-app.vercel.app",
                    "https://your-app.netlify.app"
                )
                .AllowAnyMethod()
                .AllowAnyHeader()
                .AllowCredentials();
            });
        });
        
        // Authentication (JWT)
        services.AddAuthentication()
            .AddJwtBearer(options =>
            {
                options.Authority = "https://login.microsoftonline.com/{tenant-id}";
                options.Audience = context.Configuration["AzureAd:ClientId"];
            });
    })
    .Build();

host.Run();
```

#### JWT Middleware
```csharp
// Middleware/JwtMiddleware.cs
public class JwtMiddleware : IFunctionsWorkerMiddleware
{
    public async Task Invoke(FunctionContext context, FunctionExecutionDelegate next)
    {
        var request = await context.GetHttpRequestDataAsync();
        
        // Extract JWT from Authorization header
        if (request?.Headers.TryGetValues("Authorization", out var authHeaders) == true)
        {
            var token = authHeaders.FirstOrDefault()?.Replace("Bearer ", "");
            
            if (!string.IsNullOrEmpty(token))
            {
                // Validate JWT and extract user ID
                // Store in context for use in functions
                var userId = ValidateTokenAndGetUserId(token);
                context.Items["UserId"] = userId;
            }
        }
        
        await next(context);
    }
}
```

### Database Access (Entity Framework Core)

```csharp
// Data/ApplicationDbContext.cs
using Microsoft.EntityFrameworkCore;

public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }

    public DbSet<User> Users { get; set; }
    public DbSet<Recipe> Recipes { get; set; }
    public DbSet<Rating> Ratings { get; set; }
    public DbSet<FamilyMember> FamilyMembers { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Configure relationships
        modelBuilder.Entity<Recipe>()
            .HasOne(r => r.User)
            .WithMany(u => u.Recipes)
            .HasForeignKey(r => r.UserId);

        // Configure indexes, constraints, etc.
    }
}
```

### Deployment Configuration

**host.json:**
```json
{
  "version": "2.0",
  "logging": {
    "applicationInsights": {
      "samplingSettings": {
        "isEnabled": true,
        "maxTelemetryItemsPerSecond": 20
      }
    }
  },
  "extensions": {
    "http": {
      "routePrefix": "api"
    }
  }
}
```

**local.settings.json (not committed):**
```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "dotnet-isolated",
    "SupabaseConnectionString": "Host=...;Database=...;Username=...;Password=...",
    "SupabaseUrl": "https://your-project.supabase.co",
    "SupabaseKey": "your-supabase-anon-key",
    "AzureAd:TenantId": "your-tenant-id",
    "AzureAd:ClientId": "your-client-id",
    "AzureAd:ClientSecret": "your-client-secret"
  }
}
```

---

## Authentication & Security

### Authentication Flow (Microsoft OAuth + JWT)
```
1. User clicks "Sign in with Microsoft" in Vite React app
2. Frontend redirects to Microsoft OAuth consent page
   URL: https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize
3. User grants permission, Microsoft redirects back with authorization code
4. Frontend calls Azure Function: POST /api/auth/token with code
5. Azure Function exchanges code for Microsoft tokens (access_token, id_token)
6. Azure Function creates/updates user in database
7. Azure Function generates JWT token with user ID
8. Azure Function returns JWT to frontend
9. Frontend stores JWT in memory/localStorage
10. Frontend includes JWT in Authorization header for all API calls
11. Azure Functions validate JWT using middleware on each request
12. If JWT valid, extract user ID and process request
```

### JWT Token Structure
```json
{
  "sub": "user-id-from-database",
  "email": "user@example.com",
  "name": "User Name",
  "iss": "https://your-app.azurewebsites.net",
  "aud": "foodbythebook-frontend",
  "exp": 1640000000,
  "iat": 1639900000
}
```

### Security Measures
- ✅ **Microsoft OAuth:** Industry-standard authentication
- ✅ **JWT Tokens:** Stateless authentication with expiration
- ✅ **HTTPS Only:** All communication encrypted
- ✅ **CORS Configuration:** Only allow frontend domains
- ✅ **Input Validation:** Validate all request data
- ✅ **SQL Injection Prevention:** Entity Framework Core parameterized queries
- ✅ **XSS Prevention:** React automatically escapes output
- ✅ **Rate Limiting:** Azure Functions built-in (free tier: 1M requests/month limit)
- ✅ **Environment Variables:** Secrets stored in Azure App Settings (not in code)

### Authorization Strategy
- **User-Based Access Control:** Users can only access their own recipes
- **Function-level validation:** Each function checks `userId` from JWT matches requested resource
- **No role system needed for MVP:** Only one user type (recipe owner)

---

## Deployment Architecture

### Environments
```
┌─────────────┐
│ Development │  - Local machines
├─────────────┤
│   Staging   │  - Pre-production testing
├─────────────┤
│ Production  │  - Live system
└─────────────┘
```

### Infrastructure
```
┌──────────────────────────────────────┐
│         Load Balancer                │
└────────┬────────────────────┬────────┘
         │                    │
┌────────▼────────┐  ┌────────▼────────┐
│  App Server 1   │  │  App Server 2   │
└────────┬────────┘  └────────┬────────┘
         │                    │
         └──────────┬─────────┘
                    │
         ┌──────────▼──────────┐
         │      Database       │
         │   (with replicas)   │
         └─────────────────────┘
```

### Scalability Strategy
- **Horizontal Scaling:** Add more servers
- **Vertical Scaling:** Increase server resources
- **Database Scaling:** Read replicas, sharding
- **Caching:** Redis for frequently accessed data
- **CDN:** Static asset delivery

---

## Monitoring & Observability

### Logging Strategy
- **Application Logs:** Winston, Pino, or built-in
- **Access Logs:** Nginx, API Gateway
- **Error Logs:** Sentry, Rollbar

### Metrics to Track
- [ ] Response times
- [ ] Error rates
- [ ] Request counts
- [ ] Database query performance
- [ ] Memory usage
- [ ] CPU usage
- [ ] Active users

### Health Checks
```
GET /health       - Basic health check
GET /health/db    - Database connectivity
GET /health/ready - Readiness probe
```

---

## Disaster Recovery

### Backup Strategy
- **Database Backups:**
  - Frequency: [Daily/Hourly]
  - Retention: [30 days]
  - Location: [Cloud storage]

- **Application Backups:**
  - Code: Git repository
  - Configuration: Environment variables in secure storage

### Recovery Plan
1. Identify the issue
2. Assess impact
3. Execute recovery procedure
4. Validate recovery
5. Post-mortem analysis

---

## Performance Optimization

### Frontend
- [ ] Code splitting
- [ ] Lazy loading
- [ ] Image optimization
- [ ] Caching strategies
- [ ] Bundle size optimization
- [ ] Tree shaking

### Backend
- [ ] Database query optimization
- [ ] Caching (Redis)
- [ ] Connection pooling
- [ ] API response compression
- [ ] Rate limiting

### Network
- [ ] CDN for static assets
- [ ] HTTP/2 or HTTP/3
- [ ] Gzip/Brotli compression
- [ ] Minimize redirects

---

## Third-Party Integrations
[List external services and how they integrate]

| Service | Purpose | Integration Point |
|---------|---------|-------------------|
| [Service name] | [What it does] | [How it connects] |

---

## Architecture Decision Records (ADRs)

### ADR-001: [Decision Title]
**Date:** [Date]  
**Status:** Accepted/Rejected/Superseded  
**Context:** [What led to this decision]  
**Decision:** [What was decided]  
**Consequences:** [Impact of this decision]

---

## Future Considerations
[Technologies or patterns to consider for future iterations]

- [ ] Microservices migration
- [ ] GraphQL adoption
- [ ] Event-driven architecture
- [ ] Kubernetes orchestration
- [ ] Service mesh (Istio, Linkerd)
