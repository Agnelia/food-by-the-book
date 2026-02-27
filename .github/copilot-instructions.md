# GitHub Copilot Instructions for FoodByTheBook

## 🎯 Project Context
This is a **meal planning & recipe management application** with AI-powered OCR for digitizing paper recipes.

**Tech Stack:**
- Frontend: Vite + React 18 + TypeScript + Redux Toolkit + RTK Query
- Backend: Azure Functions (.NET 8 C#)
- Database: Supabase PostgreSQL
- Storage: Supabase Storage
- Auth: Microsoft OAuth 2.0 + JWT

---

## 📚 Required Reading Before Code Generation

**ALWAYS reference these files before generating code:**

1. **docs/CODE_PATTERNS.md** - ⭐ PRIMARY REFERENCE
   - Code templates and patterns
   - Clean code principles
   - Naming conventions
   - Component patterns (UI, Pages, Hooks, Services)
   - Backend C# patterns (Functions, Services, Repositories)

2. **docs/ARCHITECTURE.md**
   - System design and architecture patterns
   - API endpoint structure
   - Authentication flow
   - Database connection patterns

3. **docs/DATA_MODELS.md**
   - Database schema and table structures
   - DTO definitions
   - Type definitions

4. **docs/TECH_STACK.md**
   - Technology decisions and rationale
   - Package versions and compatibility

---

## 🎨 Frontend Code Generation Rules

### React Components
- Use functional components with TypeScript
- Follow the component patterns in CODE_PATTERNS.md
- Use hooks (useState, useEffect, custom hooks)
- Keep components under 200 lines (split if larger)
- Props interfaces should be named `{ComponentName}Props`

### File Naming
- React components: PascalCase (e.g., `RecipeCard.tsx`)
- RTK Query API slices: camelCase + `Api` suffix (e.g., `recipeApi.ts`)
- Redux slices: camelCase + `Slice` suffix (e.g., `authSlice.ts`)
- Pages: PascalCase (e.g., `RecipeListPage.tsx`)

### Import Order (follow exactly)
```typescript
// 1. External libraries
import { useState, useEffect } from 'react';
import { useNavigate } from 'react-router-dom';

// 2. RTK Query hooks
import { useGetRecipesQuery, useCreateRecipeMutation } from '@/store/api/recipeApi';

// 3. Components
import { RecipeCard } from '@/components/ui/RecipeCard';

// 4. Types
import type { Recipe, CreateRecipeDto } from '@/types';

// 5. Styles (if separate CSS file)
import './RecipeList.css';
```

### API Calls with RTK Query
- Use **Redux Toolkit Query (RTK Query)** for all data fetching and mutations
- Define API endpoints in RTK Query slices (`store/api/`)
- Use auto-generated hooks from API slices
- Always handle loading and error states
- Follow the API Slice Pattern in CODE_PATTERNS.md

### RTK Query Patterns
- **API Slices**: Define endpoints with `createApi` and `fetchBaseQuery`
- **Auto-generated Hooks**: Use hooks like `useGetRecipesQuery`, `useCreateRecipeMutation`
- **Tag-based Caching**: Use `providesTags` and `invalidatesTags` for cache management
- **Store Setup**: Configure Redux store with RTK Query middleware
- **TypeScript**: Define types for all endpoints
- **Base Query**: Centralize auth headers in `prepareHeaders`

#### API Slice Structure
```typescript
// store/api/recipeApi.ts
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';

export const recipeApi = createApi({
  reducerPath: 'recipeApi',
  baseQuery: fetchBaseQuery({
    baseUrl: import.meta.env.VITE_API_URL,
    prepareHeaders: (headers) => {
      const token = localStorage.getItem('auth_token');
      if (token) headers.set('Authorization', `Bearer ${token}`);
      return headers;
    },
  }),
  tagTypes: ['Recipe'],
  endpoints: (builder) => ({
    getRecipes: builder.query<Recipe[], void>({
      query: () => '/recipes',
      providesTags: ['Recipe'],
    }),
    createRecipe: builder.mutation<Recipe, CreateRecipeDto>({
      query: (data) => ({ url: '/recipes', method: 'POST', body: data }),
      invalidatesTags: ['Recipe'],
    }),
  }),
});

export const { useGetRecipesQuery, useCreateRecipeMutation } = recipeApi;
```

#### Component Usage
```typescript
import { useGetRecipesQuery, useCreateRecipeMutation } from '@/store/api/recipeApi';

function RecipeList() {
  const { data: recipes, isLoading } = useGetRecipesQuery();
  const [createRecipe] = useCreateRecipeMutation();
  
  // Component logic
}
```

---

## ⚙️ Backend Code Generation Rules

### Azure Functions (.NET 8 C#)
- Use HTTP-triggered functions with isolated worker model
- Follow the Function template in CODE_PATTERNS.md
- Always validate JWT tokens via middleware
- Return consistent response structure

### File Organization
```
Functions/
  RecipesFunctions.cs          // API endpoints
Services/
  RecipeService.cs             // Business logic
Repositories/
  RecipeRepository.cs          // Database access
DTOs/
  RecipeDto.cs                 // Data transfer objects
Models/
  Recipe.cs                    // Database models
```

### Naming Conventions
- Functions: `{Entity}{Action}` (e.g., `RecipeCreate`, `RecipeGetById`)
- Services: `{Entity}Service` (e.g., `RecipeService`)
- Repositories: `{Entity}Repository` (e.g., `RecipeRepository`)
- DTOs: `{Entity}Dto` or `{Entity}CreateDto` (e.g., `RecipeDto`, `RecipeCreateDto`)

### Dependency Injection
Always use constructor injection:
```csharp
public class RecipeService
{
    private readonly IRecipeRepository _repository;
    
    public RecipeService(IRecipeRepository repository)
    {
        _repository = repository;
    }
}
```

---

## 🗄️ Database Patterns

### Supabase PostgreSQL
- Use parameterized queries (NEVER string concatenation)
- Connection via Npgsql package
- Follow the repository pattern in CODE_PATTERNS.md
- Always use async/await

### Query Example
```csharp
var recipe = await _connection.QueryFirstOrDefaultAsync<Recipe>(
    "SELECT * FROM recipes WHERE id = @Id",
    new { Id = recipeId }
);
```

---

## 🔒 Security Rules

### Authentication
- All API endpoints require JWT validation (except auth endpoints)
- Extract userId from JWT claims
- Never trust client-provided user IDs

### Authorization
- Users can only access their own data
- Always check ownership before update/delete operations

### Input Validation
- Validate all inputs (required fields, max lengths, formats)
- Sanitize user inputs before database operations
- Return meaningful error messages

---

## ✅ Code Quality Standards

### Clean Code Principles (from CODE_PATTERNS.md)
1. **Single Responsibility Principle** - Each function does ONE thing
2. **KISS** - Keep it simple, avoid over-engineering
3. **Early Returns** - Reduce nesting with guard clauses
4. **Descriptive Names** - Variables and functions should be self-documenting
5. **No Magic Numbers** - Use named constants

### File Length Limits
- Components: 200 lines max
- Functions: 50 lines max
- Services: 300 lines max
- Utilities: Split if over 200 lines

### Comments
- Use comments to explain WHY, not WHAT
- Add JSDoc/XML comments for public APIs
- Avoid obvious comments

---

## 🚀 Development Workflow

### When Adding New Features
1. Check if similar patterns exist in CODE_PATTERNS.md
2. Follow the established file structure
3. Add types/interfaces first (TypeScript) or DTOs (C#)
4. Implement service layer before UI
5. Add error handling
6. Keep it consistent with existing code

### Testing (when implemented)
- Unit tests for services and utilities
- Integration tests for API endpoints
- Follow test patterns in respective test directories

---

## 🛑 Anti-Patterns to Avoid

### Frontend
- ❌ Don't use class components (use functional)
- ❌ Don't put business logic in components (use Redux slices or services)
- ❌ Don't use inline styles (use CSS modules or styled-components)
- ❌ Don't forget error boundaries
- ❌ Don't fetch data directly in components (use RTK Query auto-generated hooks)
- ❌ Don't forget to add `invalidatesTags` in mutations
- ❌ Don't define endpoints outside RTK Query API slices
- ❌ Don't forget to add RTK Query middleware to store configuration

### Backend
- ❌ Don't put business logic in Functions (use Services)
- ❌ Don't use raw SQL strings (use parameterized queries)
- ❌ Don't catch exceptions without handling them
- ❌ Don't return database models directly (use DTOs)

---

## 💡 AI Assistant Tips

**When I ask you to create code:**
1. First, read the relevant section in docs/CODE_PATTERNS.md
2. Check docs/ARCHITECTURE.md for system design context
3. Follow existing patterns exactly
4. Ask clarifying questions if requirements are unclear
5. Suggest improvements if you spot issues

**When I ask you to debug:**
1. Check error messages against common patterns
2. Verify authentication/authorization flow
3. Check database connection and query syntax
4. Review environment variable configuration

**When I ask you to refactor:**
1. Follow Clean Code principles from CODE_PATTERNS.md
2. Maintain existing API contracts
3. Don't change functionality unless requested
4. Split large files following our size limits

---

## 📦 Package Usage

### Frontend (npm)
- React Router v6 for routing
- **@reduxjs/toolkit** (Redux Toolkit) for state management and RTK Query
- **react-redux** for React bindings
- @azure/msal-browser for authentication
- Tesseract.js for OCR
- TypeScript for type safety

### Backend (NuGet)
- Microsoft.Azure.Functions.Worker
- Npgsql for PostgreSQL
- Microsoft.Identity.Web for JWT
- Dapper for data mapping (if needed)

---

## 🎯 Project Goals

**Remember:** This is an MVP focused on:
1. Microsoft OAuth authentication
2. Recipe CRUD operations
3. Photo upload with OCR extraction

**Success Metric:** 1 real user + deployed to production

**Budget Constraint:** Free tier services only (Azure Functions free tier, Supabase free tier)

---

**Last Updated:** February 2026
