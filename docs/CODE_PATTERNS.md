# Code Templates & Patterns

This document provides concrete code patterns and templates for AI-assisted development. Follow these patterns exactly to ensure consistency.

**📌 This is the single source of truth for all code generation.** All code examples, templates, and patterns are defined here. Other documentation files (like `.github/copilot-instructions.md`) link to specific sections of this document rather than duplicating examples.

**🔗 Cross-references:** Each pattern includes links back to why it exists and where it's used:
- References to [ARCHITECTURE.md](ARCHITECTURE.md) for system design rationale
- References to [DATA_MODELS.md](DATA_MODELS.md) for type definitions
- References to [TECH_STACK.md](TECH_STACK.md) for technology choices
- References to `.github/copilot-instructions.md` for quick navigation

---

## Table of Contents
1. [Clean Code Principles](#clean-code-principles)
2. [Code Length Guidelines](#code-length-guidelines)
3. [Project Structure](#project-structure)
4. [Naming Conventions](#naming-conventions)
5. [TypeScript Types](#typescript-types)
6. [RTK Query API Slice Pattern](#rtk-query-api-slice-pattern)
7. [RTK Query Anti-Patterns](#rtk-query-anti-patterns)
8. [UI Component Pattern](#ui-component-pattern)
9. [Page Component Pattern](#page-component-pattern)
10. [Custom Hook Pattern](#custom-hook-pattern)
11. [API Service Pattern](#api-service-pattern)
12. [Backend C# Patterns](#backend-c-patterns)
13. [Error Handling](#error-handling)
14. [Import Order](#import-order)
15. [Comment Guidelines](#comment-guidelines)
16. [File Templates](#file-templates)

---

## Clean Code Principles

### Single Responsibility Principle (SRP)
**Rule:** Each function, component, or class should do ONE thing well

**Good Example:**
```typescript
// ✅ GOOD: Each function has a single purpose
function validateEmail(email: string): boolean {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
}

function saveUserEmail(userId: string, email: string): Promise<void> {
  if (!validateEmail(email)) {
    throw new Error('Invalid email format');
  }
  return userRepository.updateEmail(userId, email);
}
```

**Bad Example:**
```typescript
// ❌ BAD: Function does too many things
function processUser(email: string, userId: string): Promise<void> {
  // Validation
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!emailRegex.test(email)) {
    throw new Error('Invalid email');
  }
  
  // Formatting
  const formatted = email.toLowerCase().trim();
  
  // Logging
  console.log('Updating user', userId);
  
  // Database update
  return userRepository.updateEmail(userId, formatted);
}
```

### KISS (Keep It Simple, Stupid)
**Rule:** Prefer simple, straightforward solutions

**Good Example:**
```typescript
// ✅ GOOD: Simple and clear
function isRecipeOwner(recipe: Recipe, userId: string): boolean {
  return recipe.userId === userId;
}
```

**Bad Example:**
```typescript
// ❌ BAD: Unnecessarily complex
function isRecipeOwner(recipe: Recipe, userId: string): boolean {
  const ownershipMatrix = new Map<string, string>();
  ownershipMatrix.set(recipe.id, recipe.userId);
  return ownershipMatrix.get(recipe.id) === userId ? true : false;
}
```

### Early Returns
**Rule:** Return early to reduce nesting

**Good Example:**
```typescript
// ✅ GOOD: Early returns reduce nesting
function processRecipe(recipe: Recipe | null): string {
  if (!recipe) {
    return 'No recipe found';
  }
  
  if (!recipe.title) {
    return 'Recipe has no title';
  }
  
  if (recipe.ingredients.length === 0) {
    return 'Recipe has no ingredients';
  }
  
  return `Processing: ${recipe.title}`;
}
```

**Bad Example:**
```typescript
// ❌ BAD: Deep nesting is hard to read
function processRecipe(recipe: Recipe | null): string {
  if (recipe) {
    if (recipe.title) {
      if (recipe.ingredients.length > 0) {
        return `Processing: ${recipe.title}`;
      } else {
        return 'Recipe has no ingredients';
      }
    } else {
      return 'Recipe has no title';
    }
  } else {
    return 'No recipe found';
  }
}
```

### Extract Complex Conditions
**Rule:** Name complex conditions for clarity

**Good Example:**
```typescript
// ✅ GOOD: Named conditions are self-documenting
function canEditRecipe(recipe: Recipe, user: User): boolean {
  const isOwner = recipe.userId === user.id;
  const isAdmin = user.role === 'admin';
  const isNotDeleted = !recipe.deletedAt;
  
  return (isOwner || isAdmin) && isNotDeleted;
}
```

**Bad Example:**
```typescript
// ❌ BAD: Complex inline conditions are hard to understand
function canEditRecipe(recipe: Recipe, user: User): boolean {
  return (recipe.userId === user.id || user.role === 'admin') && !recipe.deletedAt;
}
```

---

## Code Length Guidelines

### Strict Limits for Readability

#### Functions/Methods
- **Maximum:** 30 lines (excluding comments and blank lines)
- **Ideal:** 10-15 lines
- **If longer:** Extract sub-functions

**Example:**
```typescript
// ✅ GOOD: Short, focused function
async function createRecipe(data: CreateRecipeRequest): Promise<Recipe> {
  validateRecipeData(data);
  const sanitized = sanitizeRecipeInputs(data);
  const recipe = await recipeRepository.create(sanitized);
  await sendRecipeCreatedEvent(recipe);
  return recipe;
}

// Each helper is also short and focused
function validateRecipeData(data: CreateRecipeRequest): void {
  if (!data.title) throw new Error('Title required');
  if (data.ingredients.length === 0) throw new Error('Ingredients required');
}

function sanitizeRecipeInputs(data: CreateRecipeRequest): CreateRecipeRequest {
  return {
    ...data,
    title: data.title.trim(),
    ingredients: data.ingredients.map(i => i.trim())
  };
}
```

#### React Components
- **Maximum:** 150 lines per component
- **Ideal:** 50-100 lines
- **If longer:** Split into smaller components or extract custom hooks

**Example:**
```typescript
// ✅ GOOD: Split large components
function RecipeDetailPage() {
  return (
    <div>
      <RecipeHeader recipe={recipe} />
      <RecipeIngredients ingredients={recipe.ingredients} />
      <RecipeInstructions instructions={recipe.instructions} />
      <RecipeMetadata recipe={recipe} />
    </div>
  );
}

// Each sub-component is focused and manageable
function RecipeHeader({ recipe }: { recipe: Recipe }) {
  return (
    <header>
      <h1>{recipe.title}</h1>
      <RecipeImage src={recipe.imageUrl} />
    </header>
  );
}
```

#### Files
- **Maximum:** 300 lines
- **Ideal:** 100-200 lines
- **If longer:** Split into multiple files

#### Code Blocks (if/for/while)
- **Maximum:** 10 lines per block
- **If longer:** Extract to separate function

**Example:**
```typescript
// ✅ GOOD: Extracted complex logic
function processRecipes(recipes: Recipe[]): ProcessedRecipe[] {
  return recipes.map(recipe => processRecipe(recipe));
}

function processRecipe(recipe: Recipe): ProcessedRecipe {
  const formatted = formatRecipe(recipe);
  const validated = validateRecipe(formatted);
  return enrichRecipe(validated);
}
```

#### Parameter Count
- **Maximum:** 4 parameters
- **Ideal:** 1-2 parameters
- **If more:** Use an options object

**Example:**
```typescript
// ✅ GOOD: Options object for many parameters
interface CreateRecipeOptions {
  title: string;
  ingredients: string[];
  instructions: string[];
  servings?: number;
  prepTime?: number;
  cookTime?: number;
}

function createRecipe(options: CreateRecipeOptions): Promise<Recipe> {
  // Implementation
}

// ❌ BAD: Too many parameters
function createRecipe(
  title: string,
  ingredients: string[],
  instructions: string[],
  servings: number,
  prepTime: number,
  cookTime: number
): Promise<Recipe> {
  // Implementation
}
```

---

## Project Structure

### Frontend (Vite + React)
```
frontend/
├── src/
│   ├── components/
│   │   ├── ui/                      # UI abstraction layer
│   │   │   ├── Button.tsx           # UIButton component
│   │   │   ├── Input.tsx            # UIInput component
│   │   │   ├── Card.tsx             # UICard component
│   │   │   └── index.ts             # Barrel export
│   │   ├── lib/
│   │   │   └── digi-adapter.ts      # Third-party adapter
│   │   ├── layout/
│   │   │   ├── Header.tsx
│   │   │   ├── Footer.tsx
│   │   │   └── Layout.tsx
│   │   └── common/
│   │       ├── RecipeCard.tsx
│   │       ├── ImageUpload.tsx
│   │       └── Loading.tsx
│   ├── pages/
│   │   ├── Home.tsx
│   │   ├── Login.tsx
│   │   ├── Recipes.tsx
│   │   ├── RecipeDetail.tsx
│   │   ├── RecipeCreate.tsx
│   │   └── NotFound.tsx
│   ├── hooks/
│   │   ├── useAuth.ts
│   │   ├── useRecipes.ts
│   │   ├── useRecipe.ts
│   │   ├── usePhotoUpload.ts
│   │   ├── useOCR.ts
│   │   └── index.ts                 # Barrel export
│   ├── services/
│   │   ├── api.ts                   # Base API client
│   │   ├── recipeService.ts
│   │   ├── authService.ts
│   │   └── imageService.ts
│   ├── types/
│   │   ├── recipe.ts
│   │   ├── user.ts
│   │   ├── api.ts
│   │   └── index.ts                 # Barrel export
│   ├── utils/
│   │   ├── formatters.ts
│   │   ├── validators.ts
│   │   └── constants.ts
│   ├── contexts/
│   │   └── AuthContext.tsx
│   ├── styles/
│   │   └── index.css
│   ├── App.tsx
│   ├── main.tsx
│   └── vite-env.d.ts
├── public/
│   └── favicon.ico
├── index.html
├── vite.config.ts
├── tsconfig.json
├── package.json
├── .env.example
├── .env.local                       # gitignored
└── .gitignore
```

### Backend (Azure Functions C#)
```
backend/
├── Functions/
│   ├── RecipesFunctions.cs
│   ├── AuthFunctions.cs
│   └── ImageFunctions.cs
├── Services/
│   ├── IRecipeService.cs
│   ├── RecipeService.cs
│   ├── IImageService.cs
│   └── ImageService.cs
├── Repositories/
│   ├── IRecipeRepository.cs
│   └── RecipeRepository.cs
├── Models/
│   ├── Recipe.cs
│   ├── User.cs
│   ├── CreateRecipeRequest.cs
│   └── RecipeResponse.cs
├── Data/
│   └── ApplicationDbContext.cs
├── Middleware/
│   ├── JwtMiddleware.cs
│   └── ErrorHandlingMiddleware.cs
├── Extensions/
│   └── ServiceCollectionExtensions.cs
├── Program.cs
├── host.json
├── local.settings.json              # gitignored
├── local.settings.example.json
└── backend.csproj
```

---

## Naming Conventions

### Files
- **Components:** PascalCase - `RecipeCard.tsx`, `ImageUpload.tsx`
- **Hooks:** camelCase with "use" prefix - `useRecipes.ts`, `useAuth.ts`
- **Services:** camelCase with "Service" suffix - `recipeService.ts`, `authService.ts`
- **Types:** camelCase - `recipe.ts`, `user.ts`, `api.ts`
- **Utils:** camelCase - `formatters.ts`, `validators.ts`

### Variables & Functions
- **Variables:** camelCase - `recipeList`, `isLoading`, `userName`
- **Functions:** camelCase - `createRecipe()`, `formatDate()`, `validateEmail()`
- **Constants:** SCREAMING_SNAKE_CASE - `API_BASE_URL`, `MAX_FILE_SIZE`
- **Types/Interfaces:** PascalCase - `Recipe`, `User`, `ApiResponse<T>`
- **Enums:** PascalCase - `RecipeStatus`, `UserRole`

### Components
- **UI Components:** PascalCase with "UI" prefix - `UIButton`, `UIInput`, `UICard`
- **Page Components:** PascalCase - `Home`, `Recipes`, `RecipeDetail`
- **Common Components:** PascalCase - `RecipeCard`, `ImageUpload`, `Loading`
- **Layout Components:** PascalCase - `Header`, `Footer`, `Layout`

### Hooks
- **Data Hooks:** camelCase with "use" prefix - `useRecipes`, `useRecipe`, `useAuth`
- **Effect Hooks:** camelCase with "use" prefix - `useDebounce`, `useLocalStorage`

---

## TypeScript Types

### Type Definition Pattern

```typescript
// types/recipe.ts
export interface Recipe {
  id: string;
  userId: string;
  title: string;
  description: string | null;
  ingredients: string[];
  instructions: string[];
  servings: number | null;
  prepTime: number | null;
  cookTime: number | null;
  imageUrl: string | null;
  ocrText: string | null;
  createdAt: string;
  updatedAt: string;
}

export interface CreateRecipeRequest {
  title: string;
  description?: string;
  ingredients: string[];
  instructions: string[];
  servings?: number;
  prepTime?: number;
  cookTime?: number;
}

export interface UpdateRecipeRequest extends Partial<CreateRecipeRequest> {
  id: string;
}

export type RecipeResponse = Recipe;
export type RecipesResponse = Recipe[];
```

### API Response Types

```typescript
// types/api.ts
export interface ApiResponse<T> {
  data: T;
  message?: string;
}

export interface ApiError {
  error: string;
  message: string;
  statusCode: number;
  details?: Record<string, string[]>;
}

export interface PaginatedResponse<T> {
  data: T[];
  total: number;
  page: number;
  pageSize: number;
  hasMore: boolean;
}
```

### User Types

```typescript
// types/user.ts
export interface User {
  id: string;
  email: string;
  name: string;
  microsoftId: string;
  createdAt: string;
  updatedAt: string;
}

export interface AuthToken {
  token: string;
  expiresAt: string;
}

export interface AuthResponse {
  user: User;
  token: AuthToken;
}
```

---

## UI Component Pattern

### Template
```tsx
// components/ui/[ComponentName].tsx
import React from 'react';
import { DigiComponent } from '@designsystem-se/af'; // or other library

export interface UI[ComponentName]Props {
  // Required props
  children: React.ReactNode;
  
  // Optional props with defaults
  variant?: 'primary' | 'secondary' | 'ghost';
  size?: 'small' | 'medium' | 'large';
  disabled?: boolean;
  
  // Event handlers
  onClick?: () => void;
  
  // Style override
  className?: string;
}

export function UI[ComponentName]({
  children,
  variant = 'primary',
  size = 'medium',
  disabled = false,
  onClick,
  className,
  ...props
}: UI[ComponentName]Props) {
  return (
    <DigiComponent
      // Map props to library's API
      af-variation={variant}
      af-size={size}
      disabled={disabled}
      onClick={onClick}
      className={className}
      {...props}
    >
      {children}
    </DigiComponent>
  );
}
```

### Example: UIButton

```tsx
// components/ui/Button.tsx
import React from 'react';
import { DigiButton } from '@designsystem-se/af';

export interface UIButtonProps {
  children: React.ReactNode;
  variant?: 'primary' | 'secondary' | 'ghost';
  size?: 'small' | 'medium' | 'large';
  disabled?: boolean;
  loading?: boolean;
  type?: 'button' | 'submit' | 'reset';
  onClick?: () => void;
  className?: string;
}

export function UIButton({
  children,
  variant = 'primary',
  size = 'medium',
  disabled = false,
  loading = false,
  type = 'button',
  onClick,
  className,
  ...props
}: UIButtonProps) {
  return (
    <DigiButton
      af-variation={variant}
      af-size={size}
      disabled={disabled || loading}
      type={type}
      onClick={onClick}
      className={className}
      {...props}
    >
      {loading ? 'Loading...' : children}
    </DigiButton>
  );
}
```

### Barrel Export

```typescript
// components/ui/index.ts
export { UIButton } from './Button';
export type { UIButtonProps } from './Button';

export { UIInput } from './Input';
export type { UIInputProps } from './Input';

export { UICard } from './Card';
export type { UICardProps } from './Card';

// ... export all UI components and their types
```

---

## Page Component Pattern

### Template
```tsx
// pages/[PageName].tsx
import React from 'react';
import { useNavigate } from 'react-router-dom';
import { UIButton, UICard } from '@/components/ui';
import { useCustomHook } from '@/hooks';

export function [PageName]() {
  const navigate = useNavigate();
  const { data, isLoading, isError, error } = useCustomHook();

  // Early returns for loading/error states
  if (isLoading) {
    return (
      <div className="loading-container">
        <UILoading />
      </div>
    );
  }

  if (isError) {
    return (
      <div className="error-container">
        <UIAlert variant="error">{error?.message || 'An error occurred'}</UIAlert>
      </div>
    );
  }

  // Event handlers
  const handleAction = () => {
    // Handle action
  };

  // Main render
  return (
    <div className="page-container">
      <header className="page-header">
        <h1>[Page Title]</h1>
      </header>
      
      <main className="page-content">
        {/* Page content */}
      </main>
    </div>
  );
}
```

### Example: Recipes Page

```tsx
// pages/Recipes.tsx
import React from 'react';
import { useNavigate } from 'react-router-dom';
import { UIButton, UICard, UILoading, UIAlert } from '@/components/ui';
import { RecipeCard } from '@/components/common/RecipeCard';
import { useRecipes } from '@/hooks';

export function Recipes() {
  const navigate = useNavigate();
  const { recipes, isLoading, isError, error, refresh } = useRecipes();

  if (isLoading) {
    return (
      <div className="flex justify-center items-center min-h-screen">
        <UILoading />
      </div>
    );
  }

  if (isError) {
    return (
      <div className="container mx-auto p-4">
        <UIAlert variant="error">
          {error?.message || 'Failed to load recipes'}
        </UIAlert>
        <UIButton onClick={refresh} className="mt-4">
          Try Again
        </UIButton>
      </div>
    );
  }

  const handleCreateNew = () => {
    navigate('/recipes/new');
  };

  return (
    <div className="container mx-auto p-4">
      <header className="flex justify-between items-center mb-6">
        <h1 className="text-3xl font-bold">My Recipes</h1>
        <UIButton onClick={handleCreateNew} variant="primary">
          Add Recipe
        </UIButton>
      </header>

      <main>
        {recipes.length === 0 ? (
          <div className="text-center py-12">
            <p className="text-gray-500 mb-4">No recipes yet</p>
            <UIButton onClick={handleCreateNew}>
              Create Your First Recipe
            </UIButton>
          </div>
        ) : (
          <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
            {recipes.map((recipe) => (
              <RecipeCard key={recipe.id} recipe={recipe} />
            ))}
          </div>
        )}
      </main>
    </div>
  );
}
```

---

## RTK Query API Slice Pattern

### API Slice Template

```typescript
// store/api/[resource]Api.ts - RTK Query API slice
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';
import type {
  [Resource],
  Create[Resource]Dto,
  Update[Resource]Dto,
} from '@/types';

// Create API slice
export const [resource]Api = createApi({
  reducerPath: '[resource]Api',
  baseQuery: fetchBaseQuery({
    baseUrl: import.meta.env.VITE_API_URL,
    prepareHeaders: (headers) => {
      const token = localStorage.getItem('auth_token');
      if (token) {
        headers.set('Authorization', `Bearer ${token}`);
      }
      return headers;
    },
  }),
  tagTypes: ['[Resource]'],
  endpoints: (builder) => ({
    // GET /api/[resources]
    get[Resources]: builder.query<[Resource][], void>({
      query: () => '/[resources]',
      providesTags: ['[Resource]'],
    }),
    
    // GET /api/[resources]/:id
    get[Resource]: builder.query<[Resource], string>({
      query: (id) => `/[resources]/${id}`,
      providesTags: (result, error, id) => [{ type: '[Resource]', id }],
    }),
    
    // POST /api/[resources]
    create[Resource]: builder.mutation<[Resource], Create[Resource]Dto>({
      query: (data) => ({
        url: '/[resources]',
        method: 'POST',
        body: data,
      }),
      invalidatesTags: ['[Resource]'],
    }),
    
    // PUT /api/[resources]/:id
    update[Resource]: builder.mutation<[Resource], { id: string; data: Update[Resource]Dto }>({
      query: ({ id, data }) => ({
        url: `/[resources]/${id}`,
        method: 'PUT',
        body: data,
      }),
      invalidatesTags: (result, error, { id }) => [{ type: '[Resource]', id }, '[Resource]'],
    }),
    
    // DELETE /api/[resources]/:id
    delete[Resource]: builder.mutation<void, string>({
      query: (id) => ({
        url: `/[resources]/${id}`,
        method: 'DELETE',
      }),
      invalidatesTags: ['[Resource]'],
    }),
  }),
});

// Export auto-generated hooks
export const {
  useGet[Resources]Query,
  useGet[Resource]Query,
  useCreate[Resource]Mutation,
  useUpdate[Resource]Mutation,
  useDelete[Resource]Mutation,
} = [resource]Api;
```

### Example: Recipe API Slice

```typescript
// store/api/recipeApi.ts
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';
import type { Recipe, CreateRecipeDto, UpdateRecipeDto } from '@/types';

export const recipeApi = createApi({
  reducerPath: 'recipeApi',
  baseQuery: fetchBaseQuery({
    baseUrl: import.meta.env.VITE_API_URL,
    prepareHeaders: (headers) => {
      const token = localStorage.getItem('auth_token');
      if (token) {
        headers.set('Authorization', `Bearer ${token}`);
      }
      return headers;
    },
  }),
  tagTypes: ['Recipe'],
  endpoints: (builder) => ({
    getRecipes: builder.query<Recipe[], void>({
      query: () => '/recipes',
      providesTags: ['Recipe'],
    }),
    
    getRecipe: builder.query<Recipe, string>({
      query: (id) => `/recipes/${id}`,
      providesTags: (result, error, id) => [{ type: 'Recipe', id }],
    }),
    
    createRecipe: builder.mutation<Recipe, CreateRecipeDto>({
      query: (data) => ({ url: '/recipes', method: 'POST', body: data }),
      invalidatesTags: ['Recipe'],
    }),
    
    updateRecipe: builder.mutation<Recipe, { id: string; data: UpdateRecipeDto }>({
      query: ({ id, data }) => ({ url: `/recipes/${id}`, method: 'PUT', body: data }),
      invalidatesTags: (result, error, { id }) => [{ type: 'Recipe', id }, 'Recipe'],
    }),
    
    deleteRecipe: builder.mutation<void, string>({
      query: (id) => ({ url: `/recipes/${id}`, method: 'DELETE' }),
      invalidatesTags: ['Recipe'],
    }),
  }),
});

export const {
  useGetRecipesQuery,
  useGetRecipeQuery,
  useCreateRecipeMutation,
  useUpdateRecipeMutation,
  useDeleteRecipeMutation,
} = recipeApi;
```

### Redux Store Setup

```typescript
// store/store.ts
import { configureStore } from '@reduxjs/toolkit';
import { setupListeners } from '@reduxjs/toolkit/query';
import { recipeApi } from './api/recipeApi';

export const store = configureStore({
  reducer: {
    // Add RTK Query reducer
    [recipeApi.reducerPath]: recipeApi.reducer,
    // Add other slices here if needed
  },
  // Add RTK Query middleware
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(recipeApi.middleware),
});

// Enable refetch on focus/reconnect
setupListeners(store.dispatch);

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

### App Setup with Provider

```typescript
// main.tsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import { Provider } from 'react-redux';
import { store } from './store/store';
import App from './App';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <Provider store={store}>
      <App />
    </Provider>
  </React.StrictMode>
);
```

### Component Usage Pattern

```typescript
// pages/RecipeListPage.tsx
import { useGetRecipesQuery, useDeleteRecipeMutation } from '@/store/api/recipeApi';
import { RecipeCard } from '@/components/recipes/RecipeCard';

export function RecipeListPage() {
  // Auto-generated query hook
  const { data: recipes, isLoading, error, refetch } = useGetRecipesQuery();
  
  // Auto-generated mutation hook
  const [deleteRecipe, { isLoading: isDeleting }] = useDeleteRecipeMutation();
  
  const handleDelete = async (id: string) => {
    try {
      await deleteRecipe(id).unwrap();
      // Recipes list automatically refetches due to invalidatesTags
    } catch (err) {
      console.error('Failed to delete recipe:', err);
    }
  };
  
  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error loading recipes</div>;
  
  return (
    <div>
      <h1>Recipes</h1>
      <button onClick={() => refetch()}>Refresh</button>
      {recipes?.map(recipe => (
        <RecipeCard
          key={recipe.id}
          recipe={recipe}
          onDelete={() => handleDelete(recipe.id)}
        />
      ))}
    </div>
  );
}
```

### Advanced RTK Query Patterns

#### Optimistic Updates

```typescript
// Optimistic update for better UX
updateRecipe: builder.mutation<Recipe, { id: string; data: UpdateRecipeDto }>({
  query: ({ id, data }) => ({
    url: `/recipes/${id}`,
    method: 'PUT',
    body: data,
  }),
  
  // Optimistic update
  async onQueryStarted({ id, data }, { dispatch, queryFulfilled }) {
    // Update cache optimistically
    const patchResult = dispatch(
      recipeApi.util.updateQueryData('getRecipe', id, (draft) => {
        Object.assign(draft, data);
      })
    );
    
    try {
      await queryFulfilled;
    } catch {
      // Rollback on error
      patchResult.undo();
    }
  },
  
  invalidatesTags: (result, error, { id }) => [{ type: 'Recipe', id }],
}),
```

#### Conditional Queries

```typescript
// Only fetch when id is available
export function RecipeDetailPage({ recipeId }: { recipeId?: string }) {
  const { data: recipe, isLoading } = useGetRecipeQuery(recipeId!, {
    skip: !recipeId, // Skip query if no ID
  });
  
  if (!recipeId) return <div>No recipe selected</div>;
  if (isLoading) return <div>Loading...</div>;
  
  return <div>{recipe?.title}</div>;
}
```

#### Polling and Refetching

```typescript
// Automatic polling
const { data } = useGetRecipesQuery(undefined, {
  pollingInterval: 30000, // Refetch every 30 seconds
});

// Refetch on focus/reconnect (enabled by setupListeners)
setupListeners(store.dispatch);
```

---

## RTK Query Anti-Patterns

### ❌ DON'T: Fetch data directly in components

```typescript
// ❌ BAD: Manual fetch without RTK Query
function RecipesList() {
  const [recipes, setRecipes] = useState([]);
  
  useEffect(() => {
    fetch('/api/recipes')
      .then(res => res.json())
      .then(setRecipes);
  }, []);
  
  // No caching, no refetching, manual error handling
}
```

```typescript
// ✅ GOOD: Use RTK Query hook
function RecipesList() {
  const { data: recipes, isLoading, error } = useGetRecipesQuery();
  
  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  
  return <div>{recipes.map(r => <RecipeCard key={r.id} recipe={r} />)}</div>;
}
```

### ❌ DON'T: Forget invalidatesTags in mutations

```typescript
// ❌ BAD: No cache invalidation
createRecipe: builder.mutation<Recipe, CreateRecipeDto>({
  query: (data) => ({ url: '/recipes', method: 'POST', body: data }),
  // Missing invalidatesTags - list won't update!
}),
```

```typescript
// ✅ GOOD: Invalidate tags to refetch
createRecipe: builder.mutation<Recipe, CreateRecipeDto>({
  query: (data) => ({ url: '/recipes', method: 'POST', body: data }),
  invalidatesTags: ['Recipe'], // Auto-refetch recipe list
}),
```

### ❌ DON'T: Define endpoints outside API slices

```typescript
// ❌ BAD: Defining fetch logic in components or services
const fetchRecipes = async () => {
  const response = await fetch('/api/recipes');
  return response.json();
};
```

```typescript
// ✅ GOOD: Define in RTK Query API slice
export const recipeApi = createApi({
  endpoints: (builder) => ({
    getRecipes: builder.query<Recipe[], void>({
      query: () => '/recipes',
    }),
  }),
});
```

### ❌ DON'T: Forget to add middleware

```typescript
// ❌ BAD: Missing RTK Query middleware
export const store = configureStore({
  reducer: {
    [recipeApi.reducerPath]: recipeApi.reducer,
  },
  // Missing middleware - queries won't work!
});
```

```typescript
// ✅ GOOD: Include RTK Query middleware
export const store = configureStore({
  reducer: {
    [recipeApi.reducerPath]: recipeApi.reducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(recipeApi.middleware),
});
```

### ❌ DON'T: Use wrong hook type

```typescript
// ❌ BAD: Using query hook for mutations
const { data } = useGetRecipesQuery(); // This is a query hook
// How do I create a recipe?
```

```typescript
// ✅ GOOD: Use mutation hooks for mutations
const { data: recipes } = useGetRecipesQuery(); // For fetching
const [createRecipe] = useCreateRecipeMutation(); // For creating
```

### ❌ DON'T: Ignore error handling

```typescript
// ❌ BAD: No error handling
const handleCreate = async (data: CreateRecipeDto) => {
  await createRecipe(data);
  // What if it fails?
};
```

```typescript
// ✅ GOOD: Handle errors properly
const handleCreate = async (data: CreateRecipeDto) => {
  try {
    await createRecipe(data).unwrap();
    toast.success('Recipe created!');
  } catch (err) {
    toast.error('Failed to create recipe');
    console.error(err);
  }
};
```

---

## API Service Pattern

**Note:** With RTK Query, you may not need separate service files. However, if you need shared utility functions for data transformation, keep them in services.

### Service Template (Optional with RTK Query)

```typescript
// services/[resource]Service.ts
import { api } from './api';
import type {
  [Resource],
  Create[Resource]Request,
  Update[Resource]Request,
  [Resource]Response,
  [Resources]Response
} from '@/types';

class [Resource]Service {
  private readonly basePath = '/[resources]';

  async getAll(): Promise<[Resources]Response> {
    const response = await api.get<[Resources]Response>(this.basePath);
    return response.data;
  }

  async getById(id: string): Promise<[Resource]Response> {
    const response = await api.get<[Resource]Response>(`${this.basePath}/${id}`);
    return response.data;
  }

  async create(data: Create[Resource]Request): Promise<[Resource]Response> {
    const response = await api.post<[Resource]Response>(this.basePath, data);
    return response.data;
  }

  async update(id: string, data: Update[Resource]Request): Promise<[Resource]Response> {
    const response = await api.put<[Resource]Response>(`${this.basePath}/${id}`, data);
    return response.data;
  }

  async delete(id: string): Promise<void> {
    await api.delete(`${this.basePath}/${id}`);
  }
}

export const [resource]Service = new [Resource]Service();
```

### Base API Client

```typescript
// services/api.ts
import axios, { AxiosInstance, AxiosError } from 'axios';
import type { ApiError } from '@/types';

class ApiClient {
  private client: AxiosInstance;

  constructor() {
    this.client = axios.create({
      baseURL: import.meta.env.VITE_API_URL,
      headers: {
        'Content-Type': 'application/json'
      }
    });

    // Request interceptor - add auth token
    this.client.interceptors.request.use(
      (config) => {
        const token = localStorage.getItem('auth_token');
        if (token) {
          config.headers.Authorization = `Bearer ${token}`;
        }
        return config;
      },
      (error) => Promise.reject(error)
    );

    // Response interceptor - handle errors
    this.client.interceptors.response.use(
      (response) => response,
      (error: AxiosError<ApiError>) => {
        if (error.response?.status === 401) {
          // Handle unauthorized - redirect to login
          localStorage.removeItem('auth_token');
          window.location.href = '/login';
        }
        return Promise.reject(error);
      }
    );
  }

  get client() {
    return this.client;
  }
}

export const api = new ApiClient().client;
```

---

## Backend C# Patterns

### Azure Functions Structure

```
backend/
├── Functions/
│   ├── RecipesFunctions.cs         # HTTP endpoints
│   ├── AuthFunctions.cs
│   └── ImageFunctions.cs
├── Services/
│   ├── IRecipeService.cs           # Interface
│   ├── RecipeService.cs            # Business logic
│   ├── IImageService.cs
│   └── ImageService.cs
├── Repositories/
│   ├── IRecipeRepository.cs        # Data access interface
│   └── RecipeRepository.cs         # Database queries
├── Models/
│   ├── Recipe.cs                   # Domain models
│   ├── CreateRecipeRequest.cs      # DTOs
│   └── RecipeResponse.cs
├── Validators/
│   └── RecipeValidator.cs
├── Extensions/
│   └── ServiceCollectionExtensions.cs
└── Program.cs                      # DI configuration
```

### Azure Function Pattern (HTTP Trigger)

```csharp
// Functions/RecipesFunctions.cs
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.Functions.Worker;
using Microsoft.Extensions.Logging;
using System.Threading.Tasks;

namespace FoodByTheBook.Functions;

public class RecipesFunctions
{
    private readonly IRecipeService _recipeService;
    private readonly ILogger<RecipesFunctions> _logger;

    // Constructor injection
    public RecipesFunctions(
        IRecipeService recipeService,
        ILogger<RecipesFunctions> logger)
    {
        _recipeService = recipeService;
        _logger = logger;
    }

    [Function("GetRecipes")]
    public async Task<IActionResult> GetRecipes(
        [HttpTrigger(AuthorizationLevel.Anonymous, "get", Route = "recipes")]
        HttpRequest req)
    {
        try
        {
            var userId = GetUserIdFromToken(req);
            var recipes = await _recipeService.GetUserRecipesAsync(userId);
            return new OkObjectResult(recipes);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to get recipes");
            return new StatusCodeResult(StatusCodes.Status500InternalServerError);
        }
    }

    [Function("GetRecipe")]
    public async Task<IActionResult> GetRecipe(
        [HttpTrigger(AuthorizationLevel.Anonymous, "get", Route = "recipes/{id}")]
        HttpRequest req,
        string id)
    {
        try
        {
            var userId = GetUserIdFromToken(req);
            var recipe = await _recipeService.GetRecipeByIdAsync(id, userId);
            
            if (recipe == null)
            {
                return new NotFoundResult();
            }
            
            return new OkObjectResult(recipe);
        }
        catch (UnauthorizedAccessException)
        {
            return new UnauthorizedResult();
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to get recipe {RecipeId}", id);
            return new StatusCodeResult(StatusCodes.Status500InternalServerError);
        }
    }

    [Function("CreateRecipe")]
    public async Task<IActionResult> CreateRecipe(
        [HttpTrigger(AuthorizationLevel.Anonymous, "post", Route = "recipes")]
        HttpRequest req)
    {
        try
        {
            var userId = GetUserIdFromToken(req);
            var request = await DeserializeRequestAsync<CreateRecipeRequest>(req);
            
            var validation = ValidateRequest(request);
            if (!validation.IsValid)
            {
                return new BadRequestObjectResult(validation.Errors);
            }
            
            var recipe = await _recipeService.CreateRecipeAsync(request, userId);
            return new CreatedResult($"/api/recipes/{recipe.Id}", recipe);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to create recipe");
            return new StatusCodeResult(StatusCodes.Status500InternalServerError);
        }
    }

    // Private helper methods - keep functions short
    private string GetUserIdFromToken(HttpRequest req)
    {
        // Extract user ID from JWT token
        var authHeader = req.Headers["Authorization"].ToString();
        // Implementation...
        return userId;
    }

    private async Task<T> DeserializeRequestAsync<T>(HttpRequest req)
    {
        // Deserialize request body
        using var reader = new StreamReader(req.Body);
        var body = await reader.ReadToEndAsync();
        return JsonSerializer.Deserialize<T>(body);
    }

    private ValidationResult ValidateRequest(object request)
    {
        // Validation logic
        return new ValidationResult { IsValid = true };
    }
}
```

### Service Pattern (Business Logic)

```csharp
// Services/IRecipeService.cs
using System.Collections.Generic;
using System.Threading.Tasks;

namespace FoodByTheBook.Services;

public interface IRecipeService
{
    Task<IEnumerable<RecipeResponse>> GetUserRecipesAsync(string userId);
    Task<RecipeResponse?> GetRecipeByIdAsync(string id, string userId);
    Task<RecipeResponse> CreateRecipeAsync(CreateRecipeRequest request, string userId);
    Task<RecipeResponse> UpdateRecipeAsync(string id, UpdateRecipeRequest request, string userId);
    Task DeleteRecipeAsync(string id, string userId);
}
```

```csharp
// Services/RecipeService.cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.Extensions.Logging;

namespace FoodByTheBook.Services;

public class RecipeService : IRecipeService
{
    private readonly IRecipeRepository _repository;
    private readonly ILogger<RecipeService> _logger;

    public RecipeService(
        IRecipeRepository repository,
        ILogger<RecipeService> logger)
    {
        _repository = repository;
        _logger = logger;
    }

    public async Task<IEnumerable<RecipeResponse>> GetUserRecipesAsync(string userId)
    {
        _logger.LogInformation("Getting recipes for user {UserId}", userId);
        
        var recipes = await _repository.GetByUserIdAsync(userId);
        return recipes.Select(MapToResponse);
    }

    public async Task<RecipeResponse?> GetRecipeByIdAsync(string id, string userId)
    {
        var recipe = await _repository.GetByIdAsync(id);
        
        if (recipe == null)
        {
            return null;
        }
        
        if (recipe.UserId != userId)
        {
            throw new UnauthorizedAccessException("User does not own this recipe");
        }
        
        return MapToResponse(recipe);
    }

    public async Task<RecipeResponse> CreateRecipeAsync(
        CreateRecipeRequest request,
        string userId)
    {
        var recipe = new Recipe
        {
            Id = Guid.NewGuid().ToString(),
            UserId = userId,
            Title = request.Title.Trim(),
            Description = request.Description?.Trim(),
            Ingredients = SanitizeList(request.Ingredients),
            Instructions = SanitizeList(request.Instructions),
            Servings = request.Servings,
            PrepTime = request.PrepTime,
            CookTime = request.CookTime,
            CreatedAt = DateTime.UtcNow,
            UpdatedAt = DateTime.UtcNow
        };

        await _repository.CreateAsync(recipe);
        _logger.LogInformation("Created recipe {RecipeId} for user {UserId}", recipe.Id, userId);
        
        return MapToResponse(recipe);
    }

    public async Task<RecipeResponse> UpdateRecipeAsync(
        string id,
        UpdateRecipeRequest request,
        string userId)
    {
        var recipe = await _repository.GetByIdAsync(id);
        
        if (recipe == null)
        {
            throw new NotFoundException($"Recipe {id} not found");
        }
        
        if (recipe.UserId != userId)
        {
            throw new UnauthorizedAccessException("User does not own this recipe");
        }

        UpdateRecipeProperties(recipe, request);
        recipe.UpdatedAt = DateTime.UtcNow;
        
        await _repository.UpdateAsync(recipe);
        _logger.LogInformation("Updated recipe {RecipeId}", id);
        
        return MapToResponse(recipe);
    }

    public async Task DeleteRecipeAsync(string id, string userId)
    {
        var recipe = await _repository.GetByIdAsync(id);
        
        if (recipe == null)
        {
            throw new NotFoundException($"Recipe {id} not found");
        }
        
        if (recipe.UserId != userId)
        {
            throw new UnauthorizedAccessException("User does not own this recipe");
        }

        await _repository.DeleteAsync(id);
        _logger.LogInformation("Deleted recipe {RecipeId}", id);
    }

    // Private helper methods - extract for readability
    private static RecipeResponse MapToResponse(Recipe recipe)
    {
        return new RecipeResponse
        {
            Id = recipe.Id,
            UserId = recipe.UserId,
            Title = recipe.Title,
            Description = recipe.Description,
            Ingredients = recipe.Ingredients,
            Instructions = recipe.Instructions,
            Servings = recipe.Servings,
            PrepTime = recipe.PrepTime,
            CookTime = recipe.CookTime,
            ImageUrl = recipe.ImageUrl,
            OcrText = recipe.OcrText,
            CreatedAt = recipe.CreatedAt.ToString("o"),
            UpdatedAt = recipe.UpdatedAt.ToString("o")
        };
    }

    private static List<string> SanitizeList(List<string> items)
    {
        return items
            .Where(item => !string.IsNullOrWhiteSpace(item))
            .Select(item => item.Trim())
            .ToList();
    }

    private static void UpdateRecipeProperties(Recipe recipe, UpdateRecipeRequest request)
    {
        if (request.Title != null) recipe.Title = request.Title.Trim();
        if (request.Description != null) recipe.Description = request.Description.Trim();
        if (request.Ingredients != null) recipe.Ingredients = SanitizeList(request.Ingredients);
        if (request.Instructions != null) recipe.Instructions = SanitizeList(request.Instructions);
        if (request.Servings.HasValue) recipe.Servings = request.Servings;
        if (request.PrepTime.HasValue) recipe.PrepTime = request.PrepTime;
        if (request.CookTime.HasValue) recipe.CookTime = request.CookTime;
    }
}
```

### Repository Pattern (Data Access)

```csharp
// Repositories/IRecipeRepository.cs
using System.Collections.Generic;
using System.Threading.Tasks;

namespace FoodByTheBook.Repositories;

public interface IRecipeRepository
{
    Task<IEnumerable<Recipe>> GetByUserIdAsync(string userId);
    Task<Recipe?> GetByIdAsync(string id);
    Task<Recipe> CreateAsync(Recipe recipe);
    Task UpdateAsync(Recipe recipe);
    Task DeleteAsync(string id);
}
```

```csharp
// Repositories/RecipeRepository.cs
using System.Collections.Generic;
using System.Threading.Tasks;
using Npgsql;
using Dapper;

namespace FoodByTheBook.Repositories;

public class RecipeRepository : IRecipeRepository
{
    private readonly string _connectionString;

    public RecipeRepository(string connectionString)
    {
        _connectionString = connectionString;
    }

    public async Task<IEnumerable<Recipe>> GetByUserIdAsync(string userId)
    {
        const string sql = @"
            SELECT * FROM recipes 
            WHERE user_id = @UserId 
            ORDER BY created_at DESC";

        using var connection = new NpgsqlConnection(_connectionString);
        return await connection.QueryAsync<Recipe>(sql, new { UserId = userId });
    }

    public async Task<Recipe?> GetByIdAsync(string id)
    {
        const string sql = "SELECT * FROM recipes WHERE id = @Id";

        using var connection = new NpgsqlConnection(_connectionString);
        return await connection.QuerySingleOrDefaultAsync<Recipe>(sql, new { Id = id });
    }

    public async Task<Recipe> CreateAsync(Recipe recipe)
    {
        const string sql = @"
            INSERT INTO recipes (
                id, user_id, title, description, 
                ingredients, instructions, servings, 
                prep_time, cook_time, created_at, updated_at
            ) VALUES (
                @Id, @UserId, @Title, @Description, 
                @Ingredients, @Instructions, @Servings, 
                @PrepTime, @CookTime, @CreatedAt, @UpdatedAt
            )";

        using var connection = new NpgsqlConnection(_connectionString);
        await connection.ExecuteAsync(sql, recipe);
        return recipe;
    }

    public async Task UpdateAsync(Recipe recipe)
    {
        const string sql = @"
            UPDATE recipes SET 
                title = @Title,
                description = @Description,
                ingredients = @Ingredients,
                instructions = @Instructions,
                servings = @Servings,
                prep_time = @PrepTime,
                cook_time = @CookTime,
                updated_at = @UpdatedAt
            WHERE id = @Id";

        using var connection = new NpgsqlConnection(_connectionString);
        await connection.ExecuteAsync(sql, recipe);
    }

    public async Task DeleteAsync(string id)
    {
        const string sql = "DELETE FROM recipes WHERE id = @Id";

        using var connection = new NpgsqlConnection(_connectionString);
        await connection.ExecuteAsync(sql, new { Id = id });
    }
}
```

### Model/DTO Pattern

**Use `record` for:**
- DTOs (Data Transfer Objects) - immutable request/response objects
- Value objects - compared by value, not identity
- Configuration objects

**Use `class` for:**
- Domain models with behavior and mutable state
- Entities with identity (database models)
- Objects requiring inheritance

```csharp
// Models/Recipe.cs (Domain Model - uses class for mutable state)
using System;
using System.Collections.Generic;

namespace FoodByTheBook.Models;

public class Recipe
{
    public string Id { get; set; } = string.Empty;
    public string UserId { get; set; } = string.Empty;
    public string Title { get; set; } = string.Empty;
    public string? Description { get; set; }
    public List<string> Ingredients { get; set; } = new();
    public List<string> Instructions { get; set; } = new();
    public int? Servings { get; set; }
    public int? PrepTime { get; set; }
    public int? CookTime { get; set; }
    public string? ImageUrl { get; set; }
    public string? OcrText { get; set; }
    public DateTime CreatedAt { get; set; }
    public DateTime UpdatedAt { get; set; }
}
```

```csharp
// Models/CreateRecipeRequest.cs (DTO - Data Transfer Object)
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;

namespace FoodByTheBook.Models;

public record CreateRecipeRequest
{
    [Required(ErrorMessage = "Title is required")]
    [StringLength(200, ErrorMessage = "Title must be less than 200 characters")]
    public required string Title { get; init; }

    [StringLength(1000, ErrorMessage = "Description must be less than 1000 characters")]
    public string? Description { get; init; }

    [Required(ErrorMessage = "At least one ingredient is required")]
    [MinLength(1, ErrorMessage = "At least one ingredient is required")]
    public required List<string> Ingredients { get; init; } = new();

    [Required(ErrorMessage = "At least one instruction is required")]
    [MinLength(1, ErrorMessage = "At least one instruction is required")]
    public required List<string> Instructions { get; init; } = new();

    [Range(1, 100, ErrorMessage = "Servings must be between 1 and 100")]
    public int? Servings { get; init; }

    [Range(0, 1440, ErrorMessage = "Prep time must be between 0 and 1440 minutes")]
    public int? PrepTime { get; init; }

    [Range(0, 1440, ErrorMessage = "Cook time must be between 0 and 1440 minutes")]
    public int? CookTime { get; init; }
}
```

```csharp
// Models/RecipeResponse.cs (Response DTO)
using System.Collections.Generic;

namespace FoodByTheBook.Models;

public record RecipeResponse
{
    public required string Id { get; init; }
    public required string UserId { get; init; }
    public required string Title { get; init; }
    public string? Description { get; init; }
    public required List<string> Ingredients { get; init; }
    public required List<string> Instructions { get; init; }
    public int? Servings { get; init; }
    public int? PrepTime { get; init; }
    public int? CookTime { get; init; }
    public string? ImageUrl { get; init; }
    public string? OcrText { get; init; }
    public required string CreatedAt { get; init; }
    public required string UpdatedAt { get; init; }
}
```

### Dependency Injection Setup

```csharp
// Program.cs
using Microsoft.Azure.Functions.Worker;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using FoodByTheBook.Services;
using FoodByTheBook.Repositories;

var host = new HostBuilder()
    .ConfigureFunctionsWorkerDefaults()
    .ConfigureServices(services =>
    {
        // Register services with dependency injection
        services.AddScoped<IRecipeService, RecipeService>();
        services.AddScoped<IImageService, ImageService>();
        
        // Register repositories
        services.AddScoped<IRecipeRepository>(sp =>
        {
            var connectionString = Environment.GetEnvironmentVariable("SUPABASE_CONNECTION_STRING");
            return new RecipeRepository(connectionString);
        });
        
        // Add logging
        services.AddApplicationInsightsTelemetryWorkerService();
        services.ConfigureFunctionsApplicationInsights();
    })
    .Build();

host.Run();
```

### C# Naming Conventions

**Classes & Interfaces:**
- PascalCase: `RecipeService`, `IRecipeRepository`
- Interfaces start with `I`: `IRecipeService`

**Methods:**
- PascalCase: `GetRecipesAsync()`, `CreateRecipeAsync()`
- Async methods end with `Async`

**Variables & Parameters:**
- camelCase: `userId`, `recipeList`, `connectionString`

**Private Fields:**
- _camelCase with underscore: `_repository`, `_logger`, `_connectionString`

**Constants:**
- PascalCase: `MaxRecipeLength`, `DefaultPageSize`

**Properties:**
- PascalCase: `Title`, `CreatedAt`, `UserId`

---

## Error Handling

### Error Boundary Component

```tsx
// components/common/ErrorBoundary.tsx
import React, { Component, ErrorInfo, ReactNode } from 'react';
import { UIButton, UIAlert } from '@/components/ui';

interface Props {
  children: ReactNode;
}

interface State {
  hasError: boolean;
  error: Error | null;
}

export class ErrorBoundary extends Component<Props, State> {
  public state: State = {
    hasError: false,
    error: null
  };

  public static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  public componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('Uncaught error:', error, errorInfo);
  }

  private handleReset = () => {
    this.setState({ hasError: false, error: null });
  };

  public render() {
    if (this.state.hasError) {
      return (
        <div className="min-h-screen flex items-center justify-center p-4">
          <div className="max-w-md w-full">
            <UIAlert variant="error" className="mb-4">
              <h2 className="text-lg font-semibold mb-2">Something went wrong</h2>
              <p className="text-sm mb-4">
                {this.state.error?.message || 'An unexpected error occurred'}
              </p>
            </UIAlert>
            <UIButton onClick={this.handleReset} className="w-full">
              Try Again
            </UIButton>
          </div>
        </div>
      );
    }

    return this.props.children;
  }
}
```

### Try-Catch Pattern

```typescript
// Standard error handling pattern for async operations
async function handleAsyncOperation() {
  try {
    const result = await someAsyncFunction();
    return result;
  } catch (error) {
    if (error instanceof AxiosError) {
      // Handle API errors
      console.error('API Error:', error.response?.data);
      throw new Error(error.response?.data?.message || 'API request failed');
    } else {
      // Handle other errors
      console.error('Unexpected error:', error);
      throw error;
    }
  }
}
```

---

## Import Order

### Standard Import Order

```typescript
// 1. React and third-party libraries
import React, { useState, useEffect } from 'react';
import { useNavigate } from 'react-router-dom';
import axios from 'axios';

// 2. Types
import type { Recipe, User } from '@/types';

// 3. Components (external, then internal)
import { UIButton, UICard, UIInput } from '@/components/ui';
import { RecipeCard } from '@/components/common/RecipeCard';

// 4. Hooks
import { useRecipes, useAuth } from '@/hooks';

// 5. Services
import { recipeService } from '@/services/recipeService';

// 6. Utils
import { formatDate, validateEmail } from '@/utils';

// 7. Styles (if needed)
import './ComponentName.css';
```

---

## Comment Guidelines

### When to Comment

**DO Comment:**
- Complex business logic that isn't obvious
- Why something is done (not what is being done)
- Temporary workarounds or TODOs
- Public API documentation (JSDoc/XML comments)
- Non-obvious algorithm choices

**DON'T Comment:**
- Obvious code (let the code speak for itself)
- What the code does (use good names instead)
- Outdated or incorrect information
- Commented-out code (use version control)

### Good vs Bad Comments

**Good Examples:**
```typescript
// ✅ GOOD: Explains WHY, not what
// Using client-side OCR to avoid API costs (Phase 1 constraint)
const text = await Tesseract.recognize(image, 'eng+swe');

// ✅ GOOD: Documents public API
/**
 * Creates a new recipe for the specified user.
 * @param request - Recipe data including title, ingredients, and instructions
 * @param userId - ID of the user creating the recipe
 * @returns The created recipe with generated ID and timestamps
 * @throws ValidationError if required fields are missing
 */
async function createRecipe(
  request: CreateRecipeRequest,
  userId: string
): Promise<RecipeResponse> {
  // Implementation
}

// ✅ GOOD: Explains non-obvious business logic
// Servings are optional, but default to 4 per Swedish cooking standards
const servings = request.servings ?? 4;

// ✅ GOOD: Temporary workaround
// TODO: Replace with proper rate limiting middleware once deployed
// Currently using simple timestamp check for MVP
if (lastRequestTime && Date.now() - lastRequestTime < 1000) {
  throw new Error('Rate limit exceeded');
}
```

**Bad Examples:**
```typescript
// ❌ BAD: States the obvious
// Get the recipe by ID
const recipe = await getRecipeById(id);

// ❌ BAD: Commented-out code (use git instead)
// const oldMethod = () => {
//   // old implementation
// };

// ❌ BAD: Explains WHAT instead of WHY
// Loop through all recipes
recipes.forEach(recipe => {
  // Process the recipe
  processRecipe(recipe);
});

// ❌ BAD: Outdated comment
// This uses MongoDB (WRONG - we use PostgreSQL now)
const recipe = await db.query('SELECT * FROM recipes');
```

### Comment Style

**TypeScript/JavaScript:**
```typescript
// Single-line comment

/**
 * Multi-line JSDoc comment for functions/classes
 * @param paramName - Description
 * @returns Description
 */

/* 
 * Block comment for complex sections
 * (rarely needed if code is well-structured)
 */
```

**C#:**
```csharp
// Single-line comment

/// <summary>
/// XML documentation comment for public APIs
/// </summary>
/// <param name="paramName">Description</param>
/// <returns>Description</returns>

/* 
 * Block comment for complex sections
 * (rarely needed if code is well-structured)
 */
```

### Self-Documenting Code

**Prefer good names over comments:**

```typescript
// ✅ GOOD: Self-explanatory
function isRecipeOwnedByUser(recipe: Recipe, userId: string): boolean {
  return recipe.userId === userId;
}

// ❌ BAD: Needs comment to explain
function check(r: Recipe, u: string): boolean {
  // Check if user owns the recipe
  return r.userId === u;
}
```

```typescript
// ✅ GOOD: Extract to named function
const hasRequiredIngredients = recipe.ingredients.length > 0;
const hasRequiredInstructions = recipe.instructions.length > 0;

if (!hasRequiredIngredients || !hasRequiredInstructions) {
  throw new ValidationError('Recipe missing required fields');
}

// ❌ BAD: Complex condition needs comment
// Check if recipe has ingredients and instructions
if (recipe.ingredients.length === 0 || recipe.instructions.length === 0) {
  throw new ValidationError('Recipe missing required fields');
}
```

---

## File Templates

### React Component File

```tsx
// components/[category]/[ComponentName].tsx
import React from 'react';
import type { PropsType } from '@/types';

export interface [ComponentName]Props {
  // Define props here
}

export function [ComponentName]({ /* destructure props */ }: [ComponentName]Props) {
  // State
  const [state, setState] = useState(initialValue);
  
  // Effects
  useEffect(() => {
    // Effect logic
  }, [dependencies]);
  
  // Event handlers
  const handleEvent = () => {
    // Handler logic
  };
  
  // Early returns
  if (someCondition) {
    return <div>Loading...</div>;
  }
  
  // Main render
  return (
    <div>
      {/* Component JSX */}
    </div>
  );
}
```

### Custom Hook File

```typescript
// hooks/use[HookName].ts
import { useState, useEffect } from 'react';
import type { ReturnType } from '@/types';

export function use[HookName](params: ParamsType): ReturnType {
  const [state, setState] = useState(initialValue);
  
  useEffect(() => {
    // Effect logic
  }, [dependencies]);
  
  return {
    // Return values
  };
}
```

### Service File

```typescript
// services/[resource]Service.ts
import { api } from './api';
import type { ResourceType } from '@/types';

class [Resource]Service {
  private readonly basePath = '/[resources]';

  async methodName(params): Promise<ReturnType> {
    // Implementation
  }
}

export const [resource]Service = new [Resource]Service();
```

### Type Definition File

```typescript
// types/[resource].ts

// Main entity interface
export interface [Resource] {
  id: string;
  // ... fields
  createdAt: string;
  updatedAt: string;
}

// Request/Response types
export interface Create[Resource]Request {
  // ... fields without id, createdAt, updatedAt
}

export interface Update[Resource]Request extends Partial<Create[Resource]Request> {
  id: string;
}

export type [Resource]Response = [Resource];
export type [Resources]Response = [Resource][];
```

---

## Code Style Guidelines

### General Rules
- Use `const` for variables that don't change, `let` for those that do
- Never use `var`
- Prefer arrow functions for callbacks and short functions
- Use template literals for string interpolation
- Use optional chaining (`?.`) and nullish coalescing (`??`)
- Prefer `async/await` over `.then()` chains
- Always handle errors with try-catch
- Use meaningful variable names (no single letters except in loops)

### TypeScript Rules
- Always define types for function parameters and return values
- Use interfaces for object shapes
- Use type aliases for unions, primitives, and complex types
- Prefer `interface` over `type` for object definitions
- Export types alongside their related code
- Use generics when appropriate
- Avoid `any` - use `unknown` if type is truly unknown

### React Rules
- Use functional components only (no class components)
- Use hooks for state and side effects
- Extract complex logic into custom hooks
- Keep components small and focused (< 200 lines)
- Use TypeScript for all props
- Prefer named exports over default exports
- Use fragments (`<>...</>`) instead of unnecessary divs

### Formatting
- 2 spaces for indentation
- Single quotes for strings
- Semicolons at end of statements
- Trailing commas in multiline arrays/objects
- Max line length: 100 characters
- Use Prettier for auto-formatting

### C# Rules
- Use PascalCase for public members, _camelCase for private fields
- Always use `async`/`await` for I/O operations
- Implement interfaces for all services and repositories
- Use dependency injection for all dependencies
- Keep methods short (< 30 lines)
- Use `using` statements for IDisposable resources
- Prefer LINQ for collections over loops
- Use nullable reference types (`string?` for nullables)

---

## Code Review Checklist

### Before Committing Code

**Readability:**
- [ ] Functions are < 30 lines
- [ ] Components are < 150 lines
- [ ] Files are < 300 lines
- [ ] No deep nesting (max 3 levels)
- [ ] Variable names are descriptive
- [ ] Complex conditions are extracted and named

**Structure:**
- [ ] Single Responsibility Principle followed
- [ ] Early returns used to reduce nesting
- [ ] Related code is grouped together
- [ ] Imports are properly ordered

**TypeScript/C# Specific:**
- [ ] All functions have type annotations
- [ ] No `any` types (use `unknown` if needed)
- [ ] Interfaces used for object shapes
- [ ] Async functions return `Promise<T>`
- [ ] Error handling is present

**React Specific:**
- [ ] Components are functional (no classes)
- [ ] Hooks follow rules of hooks
- [ ] Props are typed with interfaces
- [ ] No prop drilling (use context if needed)
- [ ] Effects have dependency arrays

**Testing (Future Phase):**
- [ ] Unit tests for business logic
- [ ] Integration tests for API endpoints
- [ ] Component tests for UI

---

## Quick Reference

### Function Length Targets
- **Ideal:** 10-15 lines
- **Maximum:** 30 lines
- **If longer:** Extract sub-functions

### Component Length Targets
- **Ideal:** 50-100 lines
- **Maximum:** 150 lines
- **If longer:** Split into smaller components

### File Length Targets
- **Ideal:** 100-200 lines
- **Maximum:** 300 lines
- **If longer:** Split into multiple files

### Nesting Depth
- **Maximum:** 3 levels
- **If deeper:** Use early returns or extract functions

### Parameters
- **Maximum:** 4 parameters
- **If more:** Use options object

---

## Next Steps

With these patterns defined:
1. ✅ AI can generate consistent code following exact patterns
2. ✅ All components, hooks, services follow same structure
3. ✅ Easy to review - everything looks familiar
4. ✅ Easy to maintain - patterns are documented
5. ✅ Clean code principles enforced with specific guidelines

**Ready to use these patterns to:**
- Generate UI component wrappers for Arbetsförmedlingens Designsystem
- Create page components with proper structure
- Build custom hooks for data fetching
- Implement API services with proper error handling
- Set up backend Azure Functions with C#
- Write clean, readable, maintainable code

**Key Takeaways:**
- **Keep it short:** Functions < 30 lines, components < 150 lines
- **Keep it simple:** One responsibility per function/component
- **Keep it readable:** Good names > comments
- **Keep it structured:** Follow consistent patterns
