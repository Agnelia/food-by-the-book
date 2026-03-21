# Frontend - React + TypeScript + Vite

This directory contains the frontend application built with Vite, React 18, TypeScript, and React Router.

## Structure

```
frontend/
├── src/
│   ├── components/
│   │   ├── ui/           # Reusable UI component abstractions (UIButton, UIInput, etc.)
│   │   ├── lib/          # Third-party library adapters
│   │   ├── layout/       # Layout components (Header, Footer, Layout)
│   │   └── common/       # Shared application components (RecipeCard, ImageUpload)
│   ├── pages/            # Page components mapped to routes
│   ├── hooks/            # Custom React hooks (useAuth, useRecipes, useOCR)
│   ├── services/         # API service layer (recipeService, authService)
│   ├── utils/            # Utility functions (formatters, validators, constants)
│   ├── types/            # TypeScript type definitions
│   ├── contexts/         # React contexts for global state
│   └── styles/           # Global styles and CSS
├── public/               # Static assets (favicon, images)
└── package.json
```

## Directory Purposes

### src/components/ui/
Abstraction layer for UI components to prevent vendor lock-in.
- Wrap third-party UI library components
- Consistent API across the application
- Prefix all components with `UI` (e.g., `UIButton`, `UIInput`)
- Export via barrel file (`index.ts`)

### src/components/lib/
Adapters for third-party libraries.
- `digi-adapter.ts` - Wrapper for Arbetsförmedlingens Designsystem

### src/components/layout/
Layout components used across pages.
- `Header.tsx` - Application header
- `Footer.tsx` - Application footer
- `Layout.tsx` - Main layout wrapper

### src/components/common/
Reusable application-specific components.
- `RecipeCard.tsx` - Recipe display card
- `ImageUpload.tsx` - Image upload component
- `Loading.tsx` - Loading indicator

### src/pages/
Page components mapped to routes.
- `Home.tsx` - Landing page
- `Login.tsx` - Login page
- `Recipes.tsx` - Recipe list page
- `RecipeDetail.tsx` - Recipe detail page
- `RecipeCreate.tsx` - Recipe creation page

### src/hooks/
Custom React hooks for reusable logic.
- `useAuth.ts` - Authentication state and methods
- `useRecipes.ts` - Recipe data fetching and management
- `useRecipe.ts` - Single recipe data
- `usePhotoUpload.ts` - Photo upload handling
- `useOCR.ts` - OCR text extraction

### src/services/
API service layer for backend communication.
- `api.ts` - Axios/fetch wrapper
- `recipeService.ts` - Recipe API calls
- `authService.ts` - Authentication API calls
- `imageService.ts` - Image upload API calls

### src/utils/
Utility functions and constants.
- `formatters.ts` - Data formatting functions
- `validators.ts` - Input validation functions
- `constants.ts` - Application constants

### src/types/
TypeScript type definitions.
- `recipe.ts` - Recipe-related types
- `user.ts` - User-related types
- `api.ts` - API response types

### src/contexts/
React contexts for global state management.
- `AuthContext.tsx` - Authentication context and provider

### src/styles/
Global styles and CSS.
- `index.css` - Global styles

## Getting Started

### Prerequisites
- Node.js 18+
- npm or yarn

### Setup
```bash
cd frontend
npm create vite@latest . -- --template react-ts
npm install
npm install react-router-dom @tanstack/react-query axios
```

### Local Development
```bash
npm run dev
# Frontend will run at http://localhost:5173
```

### Build for Production
```bash
npm run build
npm run preview
```

### Configuration
Copy `.env.example` to `.env.local` and fill in your configuration values.

## Reference
See [CODE_PATTERNS.md](../docs/CODE_PATTERNS.md) for coding patterns and best practices.
