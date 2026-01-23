# Code Templates & Patterns

This document provides concrete code patterns and templates for AI-assisted development. Follow these patterns exactly to ensure consistency.

---

## Table of Contents
1. [Project Structure](#project-structure)
2. [Naming Conventions](#naming-conventions)
3. [TypeScript Types](#typescript-types)
4. [UI Component Pattern](#ui-component-pattern)
5. [Page Component Pattern](#page-component-pattern)
6. [Custom Hook Pattern](#custom-hook-pattern)
7. [API Service Pattern](#api-service-pattern)
8. [Error Handling](#error-handling)
9. [Import Order](#import-order)
10. [File Templates](#file-templates)

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

## Custom Hook Pattern

### Data Fetching Hook Template

```typescript
// hooks/use[Resource].ts
import useSWR from 'swr';
import { [Resource], [Resource]Response } from '@/types';
import { [resource]Service } from '@/services/[resource]Service';

export function use[Resources]() {
  const { data, error, isLoading, mutate } = useSWR<[Resource]Response[]>(
    '/api/[resources]',
    [resource]Service.getAll
  );

  return {
    [resources]: data || [],
    isLoading,
    isError: !!error,
    error,
    refresh: mutate
  };
}

export function use[Resource](id: string | undefined) {
  const { data, error, isLoading, mutate } = useSWR<[Resource]Response>(
    id ? `/api/[resources]/${id}` : null,
    id ? () => [resource]Service.getById(id) : null
  );

  return {
    [resource]: data,
    isLoading,
    isError: !!error,
    error,
    refresh: mutate
  };
}
```

### Example: useRecipes

```typescript
// hooks/useRecipes.ts
import useSWR from 'swr';
import { Recipe, RecipesResponse, RecipeResponse } from '@/types';
import { recipeService } from '@/services/recipeService';

export function useRecipes() {
  const { data, error, isLoading, mutate } = useSWR<RecipesResponse>(
    '/api/recipes',
    recipeService.getAll
  );

  return {
    recipes: data || [],
    isLoading,
    isError: !!error,
    error,
    refresh: mutate
  };
}

export function useRecipe(id: string | undefined) {
  const { data, error, isLoading, mutate } = useSWR<RecipeResponse>(
    id ? `/api/recipes/${id}` : null,
    id ? () => recipeService.getById(id) : null
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

### Mutation Hook Template

```typescript
// hooks/use[Resource]Mutations.ts
import { useState } from 'react';
import { use[Resources] } from './use[Resources]';
import { [resource]Service } from '@/services/[resource]Service';
import { Create[Resource]Request, Update[Resource]Request } from '@/types';

export function useCreate[Resource]() {
  const [isCreating, setIsCreating] = useState(false);
  const [error, setError] = useState<Error | null>(null);
  const { refresh } = use[Resources]();

  const create[Resource] = async (data: Create[Resource]Request) => {
    setIsCreating(true);
    setError(null);
    
    try {
      const new[Resource] = await [resource]Service.create(data);
      await refresh(); // Refresh list
      return new[Resource];
    } catch (err) {
      const error = err as Error;
      setError(error);
      throw error;
    } finally {
      setIsCreating(false);
    }
  };

  return {
    create[Resource],
    isCreating,
    error
  };
}

export function useUpdate[Resource]() {
  const [isUpdating, setIsUpdating] = useState(false);
  const [error, setError] = useState<Error | null>(null);

  const update[Resource] = async (id: string, data: Update[Resource]Request) => {
    setIsUpdating(true);
    setError(null);
    
    try {
      const updated[Resource] = await [resource]Service.update(id, data);
      return updated[Resource];
    } catch (err) {
      const error = err as Error;
      setError(error);
      throw error;
    } finally {
      setIsUpdating(false);
    }
  };

  return {
    update[Resource],
    isUpdating,
    error
  };
}

export function useDelete[Resource]() {
  const [isDeleting, setIsDeleting] = useState(false);
  const [error, setError] = useState<Error | null>(null);
  const { refresh } = use[Resources]();

  const delete[Resource] = async (id: string) => {
    setIsDeleting(true);
    setError(null);
    
    try {
      await [resource]Service.delete(id);
      await refresh(); // Refresh list
    } catch (err) {
      const error = err as Error;
      setError(error);
      throw error;
    } finally {
      setIsDeleting(false);
    }
  };

  return {
    delete[Resource],
    isDeleting,
    error
  };
}
```

---

## API Service Pattern

### Service Template

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

---

## Next Steps

With these patterns defined:
1. AI can generate consistent code following exact patterns
2. All components, hooks, services follow same structure
3. Easy to review - everything looks familiar
4. Easy to maintain - patterns are documented

**Ready to use these patterns to:**
- Generate UI component wrappers
- Create page components
- Build custom hooks
- Implement API services
- Set up project structure

**What would you like to generate first using these patterns?**
