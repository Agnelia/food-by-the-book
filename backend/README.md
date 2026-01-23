# Backend - Azure Functions (.NET 8)

This directory contains the backend API built with Azure Functions and C# .NET 8.

## Structure

```
backend/
├── Functions/          # HTTP-triggered Azure Functions (API endpoints)
├── Services/           # Business logic layer
├── Repositories/       # Data access layer
├── Models/            # Domain models and DTOs
├── Data/              # Database context and configurations
├── Middleware/        # Custom middleware (JWT validation, error handling)
├── Extensions/        # Extension methods and dependency injection setup
└── Validators/        # Input validation logic
```

## Directory Purposes

### Functions/
HTTP-triggered Azure Functions that serve as API endpoints.
- `RecipesFunctions.cs` - CRUD operations for recipes
- `AuthFunctions.cs` - Authentication and token exchange
- `ImageFunctions.cs` - Image upload and processing

### Services/
Business logic layer containing service interfaces and implementations.
- Interface pattern: `IRecipeService.cs`
- Implementation: `RecipeService.cs`

### Repositories/
Data access layer for database operations.
- Interface pattern: `IRecipeRepository.cs`
- Implementation: `RecipeRepository.cs`

### Models/
Domain models and Data Transfer Objects (DTOs).
- Domain models: `Recipe.cs`, `User.cs`
- Request DTOs: `CreateRecipeRequest.cs`
- Response DTOs: `RecipeResponse.cs`

### Data/
Database context and entity configurations.
- `ApplicationDbContext.cs` - EF Core DbContext

### Middleware/
Custom middleware for request processing.
- `JwtMiddleware.cs` - JWT token validation
- `ErrorHandlingMiddleware.cs` - Global error handling

### Extensions/
Extension methods and dependency injection configuration.
- `ServiceCollectionExtensions.cs` - DI setup

### Validators/
Input validation logic.
- `RecipeValidator.cs` - Recipe validation rules

## Getting Started

### Prerequisites
- .NET 8 SDK
- Azure Functions Core Tools v4

### Setup
```bash
cd backend
func init . --worker-runtime dotnet-isolated --target-framework net8.0
dotnet restore
```

### Local Development
```bash
func start
# Backend will run at http://localhost:7071
```

### Configuration
Copy `local.settings.example.json` to `local.settings.json` and fill in your configuration values.

## Reference
See [CODE_PATTERNS.md](../docs/CODE_PATTERNS.md) for coding patterns and best practices.
