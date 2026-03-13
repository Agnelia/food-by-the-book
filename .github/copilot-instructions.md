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

1. **[docs/CODE_PATTERNS.md](../docs/CODE_PATTERNS.md)** - ⭐ PRIMARY REFERENCE
   - Contains all code examples, templates, and patterns
   - This document links to specific sections for each task
   - See "Code Examples" section below for quick navigation

2. **[docs/ARCHITECTURE.md](../docs/ARCHITECTURE.md)**
   - System design and architecture patterns
   - API endpoint structure
   - Authentication flow
   - Database connection patterns

3. **[docs/DATA_MODELS.md](../docs/DATA_MODELS.md)**
   - Database schema and table structures
   - DTO definitions
   - Type definitions

4. **[docs/TECH_STACK.md](../docs/TECH_STACK.md)**
   - Technology decisions and rationale
   - Package versions and compatibility

---

## 🎨 Frontend Code Generation - Quick Links to CODE_PATTERNS.md

### Code Style & Principles
- **Clean Code Principles**: See [CODE_PATTERNS - Clean Code Principles](../docs/CODE_PATTERNS.md#clean-code-principles)
  - Single Responsibility Principle (SRP)
  - KISS (Keep It Simple, Stupid)
  - Early Returns to reduce nesting
  - Extract complex conditions
  
- **Code Length Guidelines**: See [CODE_PATTERNS - Code Length Guidelines](../docs/CODE_PATTERNS.md#code-length-guidelines)
  - Functions: max 30 lines
  - React Components: max 150 lines
  - Files: max 300 lines
  - Parameters: max 4

### File Organization & Naming
- **Project Structure**: See [CODE_PATTERNS - Project Structure](../docs/CODE_PATTERNS.md#project-structure)
  - Frontend folder organization for `src/`
  - File naming conventions (PascalCase components, camelCase utilities)

- **Naming Conventions**: See [CODE_PATTERNS - Naming Conventions](../docs/CODE_PATTERNS.md#naming-conventions)
  - Files: Components (PascalCase), Hooks (camelCase + "use"), Services (camelCase + "Service")
  - Variables & Functions: camelCase
  - Constants: SCREAMING_SNAKE_CASE
  - Types/Interfaces: PascalCase

### React Components
- **UI Component Pattern**: See [CODE_PATTERNS - UI Component Pattern](../docs/CODE_PATTERNS.md#ui-component-pattern)
  - Wrapper pattern for design system components
  - Props interface naming and structure
  - Barrel exports for component libraries

- **Page Component Pattern**: See [CODE_PATTERNS - Page Component Pattern](../docs/CODE_PATTERNS.md#page-component-pattern)
  - Template structure with hooks, effects, handlers
  - Error handling and loading states
  - Early returns for UX

- **Custom Hook Pattern**: See [CODE_PATTERNS - Custom Hook Pattern](../docs/CODE_PATTERNS.md#custom-hook-pattern)
  - Hook naming (`use` prefix)
  - State management and side effects

### API Data Fetching (Redux Toolkit Query)
**⭐ USE THESE PATTERNS - RTK Query is our data fetching solution**

- **RTK Query API Slice Pattern**: See [CODE_PATTERNS - RTK Query API Slice Pattern](../docs/CODE_PATTERNS.md#rtk-query-api-slice-pattern)
  - API slice creation with `createApi` and `fetchBaseQuery`
  - Auto-generated hooks (queries and mutations)
  - Tag-based cache invalidation with `providesTags` and `invalidatesTags`
  - Redux store setup with RTK Query middleware
  - Setup listeners for refetch on focus/reconnect

- **RTK Query Anti-Patterns**: See [CODE_PATTERNS - RTK Query Anti-Patterns](../docs/CODE_PATTERNS.md#rtk-query-anti-patterns)
  - ❌ Don't fetch data directly in components
  - ❌ Don't forget `invalidatesTags` in mutations
  - ❌ Don't define endpoints outside API slices
  - ❌ Don't forget middleware in store setup

**Why RTK Query?**
- Single source of truth for async state (data, loading, error)
- Automatic caching and deduplication
- Built-in refetching on focus/reconnect
- Less boilerplate than Redux + Thunk
- Better TypeScript support with auto-generated hooks

### TypeScript Types
- **Type Definition Pattern**: See [CODE_PATTERNS - TypeScript Types](../docs/CODE_PATTERNS.md#typescript-types)
  - Entity interfaces (with id, createdAt, updatedAt)
  - Request/Response types
  - API response types and error handling
  - Reusable type exports

### Other Frontend Tools
- **API Service Pattern** (Optional with RTK Query): See [CODE_PATTERNS - API Service Pattern](../docs/CODE_PATTERNS.md#api-service-pattern)
  - Only needed for shared utility functions
  - Base API client with interceptors

- **Comment Guidelines**: See [CODE_PATTERNS - Comment Guidelines](../docs/CODE_PATTERNS.md#comment-guidelines)
  - When to comment (WHY, not WHAT)
  - Comment style (JSDoc, block comments)
  - Self-documenting code preference

- **Import Order**: See [CODE_PATTERNS - Import Order](../docs/CODE_PATTERNS.md#import-order)
  - 7-step import order for consistency
  - External → Types → Components → Hooks → Services → Utils → Styles

---

## ⚙️ Backend Code Generation - Quick Links to CODE_PATTERNS.md

### Code Style & Principles
- **Clean Code Principles**: See [CODE_PATTERNS - Clean Code Principles](../docs/CODE_PATTERNS.md#clean-code-principles)
  - Applies to C# as well
  - Single Responsibility for functions/classes
  - Early returns to reduce nesting
  - Extract complex conditions

- **Code Length Guidelines**: See [CODE_PATTERNS - Code Length Guidelines](../docs/CODE_PATTERNS.md#code-length-guidelines)
  - Methods: max 30 lines
  - Code blocks (if/for): max 10 lines
  - Parameters: max 4 (use options object)

### C# Naming Conventions
- **Naming Conventions (C# Section)**: See [CODE_PATTERNS - Naming Conventions (C# Section)](../docs/CODE_PATTERNS.md#c-naming-conventions)
  - Classes & Interfaces: PascalCase (Interfaces start with `I`)
  - Methods: PascalCase with `Async` suffix for async methods
  - Variables & Parameters: camelCase
  - Private Fields: _camelCase with underscore
  - Constants: PascalCase
  - Properties: PascalCase

### Azure Functions & C# Patterns
- **Azure Functions Structure**: See [CODE_PATTERNS - Backend C# Patterns](../docs/CODE_PATTERNS.md#backend-c-patterns)
  - Folder organization (Functions, Services, Repositories, Models)
  - File naming patterns

- **Azure Function Pattern (HTTP Trigger)**: See [CODE_PATTERNS - Azure Function Pattern](../docs/CODE_PATTERNS.md#azure-function-pattern-http-trigger)
  - Constructor injection for dependencies
  - HTTP-triggered function structure
  - Error handling with try-catch
  - Helper methods to keep functions short

**Why this pattern?**
- Clean separation of concerns
- Dependency injection makes testing easier
- Short functions are easier to test and debug
- Consistent error handling across all endpoints

- **Service Pattern (Business Logic)**: See [CODE_PATTERNS - Service Pattern](../docs/CODE_PATTERNS.md#service-pattern-business-logic)
  - Interface-first design (IRecipeService)
  - Constructor injection in services
  - Business logic isolated from HTTP concerns
  - Logging for debugging

- **Repository Pattern (Data Access)**: See [CODE_PATTERNS - Repository Pattern](../docs/CODE_PATTERNS.md#repository-pattern-data-access)
  - Interface-first design (IRecipeRepository)
  - Parameterized queries (NEVER string concatenation)
  - Async/await for all database operations
  - Npgsql for PostgreSQL access

**Why this layered architecture?**
- **Functions**: Just handle HTTP (routing, validation)
- **Services**: Handle business logic (authorization, transformations)
- **Repositories**: Handle data access (queries, transactions)
- **Result**: Easy to test, understand, and maintain

- **Model/DTO Pattern**: See [CODE_PATTERNS - Model/DTO Pattern](../docs/CODE_PATTERNS.md#modelto-pattern)
  - Domain Models (class) vs DTOs (record)
  - When to use class vs record
  - Data annotations for validation
  - Request vs Response types

- **Dependency Injection Setup**: See [CODE_PATTERNS - Dependency Injection Setup](../docs/CODE_PATTERNS.md#dependency-injection-setup)
  - Program.cs configuration
  - Service registration with scope (Singleton, Scoped, Transient)
  - Connection string management

### Error Handling
- **Error Handling (C#)**: See [CODE_PATTERNS - Error Handling](../docs/CODE_PATTERNS.md#error-handling)
  - Try-catch pattern
  - Exception types and when to throw
  - Logging errors for debugging

---

## 🗄️ Database & Security Patterns

### Database Security
- **Parameterized Queries** (See [CODE_PATTERNS - Repository Pattern](../docs/CODE_PATTERNS.md#repository-pattern-data-access))
  - ✅ ALWAYS use parameterized queries with `@Parameter` syntax
  - ❌ NEVER concatenate user input into SQL strings
  - Why: Prevents SQL injection attacks

### Authentication & Authorization
- **Authentication**: Microsoft OAuth 2.0 + JWT tokens
  - Tokens stored in `localStorage` (frontend)
  - Tokens sent in `Authorization: Bearer <token>` header
  - Always extracted from JWT claims (never trust client user ID)

- **Authorization**: User-scoped data access
  - Users can only access their own recipes
  - Always validate ownership before update/delete
  - Check `recipe.userId === currentUserId` before operations

### Input Validation
- See [CODE_PATTERNS - Model/DTO Pattern](../docs/CODE_PATTERNS.md#modelto-pattern) for validation attributes
- Required fields validation
- String length limits
- Range validation for numeric values
- Return meaningful error messages

---

## ✅ Code Quality Standards

### Clean Code Principles (Reference)
See [CODE_PATTERNS - Clean Code Principles](../docs/CODE_PATTERNS.md#clean-code-principles):

1. **Single Responsibility Principle** - Each function does ONE thing
   - A function that validates should only validate
   - A function that saves should only save
   - Extract helpers for complex operations

2. **KISS** - Keep it simple, avoid over-engineering
   - Simple solutions are better
   - Don't optimize prematurely
   - Use straightforward approaches

3. **Early Returns** - Reduce nesting with guard clauses
   - Check preconditions first, return early
   - Reduces indentation and complexity
   - Makes code easier to read

4. **Descriptive Names** - Variables and functions should be self-documenting
   - `recipeOwnerId` is better than `oId`
   - `validateRecipeTitle()` is better than `check()`
   - Long clear names beat short confusing ones

5. **No Magic Numbers** - Use named constants
   - `const MAX_RECIPE_LENGTH = 200` instead of using `200` directly
   - Easier to maintain when values need to change

### Comment Guidelines
See [CODE_PATTERNS - Comment Guidelines](../docs/CODE_PATTERNS.md#comment-guidelines):
- Comment to explain **WHY**, not **WHAT**
- The code should explain what it does
- Comments should explain business decisions

### File Organization
- **Frontend Structure**: See [CODE_PATTERNS - Project Structure (Frontend)](../docs/CODE_PATTERNS.md#frontend-vite--react)
- **Backend Structure**: See [CODE_PATTERNS - Project Structure (Backend)](../docs/CODE_PATTERNS.md#backend-azure-functions-c)

---

## 🚀 Development Workflow

### When Adding New Features
1. Read the relevant section in [CODE_PATTERNS.md](../docs/CODE_PATTERNS.md)
2. Check [ARCHITECTURE.md](../docs/ARCHITECTURE.md) for system design
3. Follow the established patterns EXACTLY (copy from templates if needed)
4. Add types/interfaces first (TypeScript) or DTOs (C#)
5. Implement service/business logic before UI
6. Add error handling and validation
7. Keep it consistent with existing code

### Updating TypeScript Types
1. Check [CODE_PATTERNS - TypeScript Types](../docs/CODE_PATTERNS.md#typescript-types)
2. Add type alongside the entity (in same file)
3. Export both interfaces and response types
4. Update API slices if data structure changes

### Updating Backend Services
1. Check [CODE_PATTERNS - Service Pattern](../docs/CODE_PATTERNS.md#service-pattern-business-logic)
2. Update repository interface first
3. Update repository implementation
4. Update service interface
5. Update service implementation
6. Update or create Azure Function
7. Ensure error handling is present

---

## 🛑 Anti-Patterns to Avoid

See [CODE_PATTERNS - RTK Query Anti-Patterns](../docs/CODE_PATTERNS.md#rtk-query-anti-patterns) and [CODE_PATTERNS - Error Handling](../docs/CODE_PATTERNS.md#error-handling) for detailed examples.

### Frontend Anti-Patterns
- ❌ Using class components (use functional with hooks)
- ❌ Putting business logic in components (use Redux slices or custom hooks)
- ❌ Inline styles (use CSS modules)
- ❌ Fetching data directly in components (use RTK Query hooks)
- ❌ Forgetting to add `invalidatesTags` in mutations (cache won't update)
- ❌ Defining endpoints outside RTK Query API slices (won't be consistent)
- ❌ Forgetting middleware in store (RTK Query won't work)

### Backend Anti-Patterns
- ❌ Business logic in Azure Functions (use Services)
- ❌ String concatenation in SQL (use parameterized queries)
- ❌ Catching exceptions without handling (log and return meaningful error)
- ❌ Returning database models directly (use DTOs/Response types)
- ❌ Trusting client-provided user IDs (always extract from JWT)
- ❌ Skipping ownership checks (always verify user owns the data)

---

## 💡 How to Use These Instructions

### For New Components
1. Determine component type: UI, Page, or Common
2. Go to [CODE_PATTERNS - UI Component Pattern](../docs/CODE_PATTERNS.md#ui-component-pattern), [Page Component Pattern](../docs/CODE_PATTERNS.md#page-component-pattern), or similar
3. Copy the template structure
4. Fill in your specific logic

### For New API Endpoints
1. Create RTK Query API slice following [CODE_PATTERNS - RTK Query API Slice Pattern](../docs/CODE_PATTERNS.md#rtk-query-api-slice-pattern)
2. Create corresponding Azure Function following [CODE_PATTERNS - Azure Function Pattern](../docs/CODE_PATTERNS.md#azure-function-pattern-http-trigger)
3. Create Service following [CODE_PATTERNS - Service Pattern](../docs/CODE_PATTERNS.md#service-pattern-business-logic)
4. Create Repository following [CODE_PATTERNS - Repository Pattern](../docs/CODE_PATTERNS.md#repository-pattern-data-access)

### For Debugging Issues
1. Check if code follows the patterns in [CODE_PATTERNS.md](../docs/CODE_PATTERNS.md)
2. Review error handling section: [CODE_PATTERNS - Error Handling](../docs/CODE_PATTERNS.md#error-handling)
3. Verify RTK Query setup if data isn't updating
4. Check authentication/authorization if getting 401/403 errors
5. Check database queries are parameterized

### For Refactoring Code
1. Reference [CODE_PATTERNS - Clean Code Principles](../docs/CODE_PATTERNS.md#clean-code-principles)
2. Check file length limits in [CODE_PATTERNS - Code Length Guidelines](../docs/CODE_PATTERNS.md#code-length-guidelines)
3. Ensure proper separation of concerns (Functions → Services → Repositories)
4. Don't change functionality unless requested
5. Run tests after refactoring

---

## 📦 Package Usage

### Frontend (npm)
- **React Router v6** - routing
- **@reduxjs/toolkit** - state management + RTK Query for data fetching
- **react-redux** - React bindings for Redux
- **@azure/msal-browser** - Microsoft OAuth authentication
- **Tesseract.js** - OCR for recipe photos
- **TypeScript** - type safety

### Backend (NuGet)
- **Microsoft.Azure.Functions.Worker** - Azure Functions runtime
- **Npgsql** - PostgreSQL database driver
- **Microsoft.Identity.Web** - JWT token validation
- **Dapper** - data mapper (optional, can use Npgsql directly)

---

## 🎯 Project Goals

This is an MVP focused on:
1. Microsoft OAuth authentication
2. Recipe CRUD operations
3. Photo upload with OCR extraction

**Success Metric:** 1 real user + deployed to production

**Budget Constraint:** Free tier services only (Azure Functions free tier, Supabase free tier)

---

## 🔗 Cross-Reference Guide

**Need to write code? Follow this path:**

1. **Creating a new page** → [CODE_PATTERNS - Page Component Pattern](../docs/CODE_PATTERNS.md#page-component-pattern)
2. **Creating a new API endpoint** → [CODE_PATTERNS - RTK Query API Slice Pattern](../docs/CODE_PATTERNS.md#rtk-query-api-slice-pattern) + [CODE_PATTERNS - Azure Function Pattern](../docs/CODE_PATTERNS.md#azure-function-pattern-http-trigger)
3. **Creating a reusable component** → [CODE_PATTERNS - UI Component Pattern](../docs/CODE_PATTERNS.md#ui-component-pattern)
4. **Creating a custom hook** → [CODE_PATTERNS - Custom Hook Pattern](../docs/CODE_PATTERNS.md#custom-hook-pattern)
5. **Creating a backend service** → [CODE_PATTERNS - Service Pattern](../docs/CODE_PATTERNS.md#service-pattern-business-logic)
6. **Creating database access** → [CODE_PATTERNS - Repository Pattern](../docs/CODE_PATTERNS.md#repository-pattern-data-access)
7. **Stuck on naming** → [CODE_PATTERNS - Naming Conventions](../docs/CODE_PATTERNS.md#naming-conventions)
8. **File too long** → [CODE_PATTERNS - Code Length Guidelines](../docs/CODE_PATTERNS.md#code-length-guidelines)
9. **Need to handle errors** → [CODE_PATTERNS - Error Handling](../docs/CODE_PATTERNS.md#error-handling)
10. **Code feels messy** → [CODE_PATTERNS - Clean Code Principles](../docs/CODE_PATTERNS.md#clean-code-principles)
