# RFC-001: Next.js Project Structure and Standards

## ⚠️ IMPORTANT: LLM COMPLIANCE REQUIREMENT

This document serves as the authoritative source of truth for project structure and standards. All LLM-generated code and suggestions MUST strictly conform to these guidelines. Any deviations or exceptions must be explicitly approved and documented.

### Compliance Checklist

- [ ] Follow directory structure exactly as specified
- [ ] Adhere to all naming conventions
- [ ] Implement all security best practices
- [ ] Follow testing requirements
- [ ] Maintain performance standards
- [ ] Use absolute imports with '@' alias
- [ ] Follow component organization rules
- [ ] Implement proper error handling
- [ ] Follow version control guidelines

## Status

- Status: Accepted
- Date: 2025-03-26
- Version: 1.0.0
- Last Updated: 2025-03-26

## Abstract

This RFC defines the standard project structure, coding conventions, and best practices for Next.js applications using the App Router architecture. It establishes a consistent, scalable, and maintainable project structure that leverages Next.js App Router features while maintaining good separation of concerns.

## Motivation

The need for a standardized project structure arises from:

- Ensuring consistency across the codebase
- Improving maintainability and scalability
- Optimizing performance and security
- Facilitating team collaboration
- Enabling efficient code generation by LLMs

## Design

### 1. Directory Structure

```
project
├── app/                     # Next.js App Router root
│   ├── components/          # Components specific to root layout - Components that are only used in the root layout
│   ├── page.tsx             # Root page component - The main landing page of the application
│   ├── layout.tsx           # Root layout component - The main layout wrapper for all pages
│   ├── [page]/              # Route groups (e.g., tools/, auth/) - Dynamic route segments for feature-based organization
│   │   ├── components/      # Page-specific components - Components that are only used within this specific route
│   │   ├── actions.ts       # Server actions for this route - Server-side functions for handling form submissions and data mutations
│   │   ├── page.tsx         # Page component - The main component for this route
│   │   └── layout.tsx       # Optional layout for this route - Route-specific layout wrapper
├── components/              # Shared components - Reusable components used across multiple pages
│   ├── ui/                  # UI primitives and design system - Base UI components like buttons, inputs, cards
│   └── shared/              # Reusable feature components - Complex components that combine multiple UI primitives
├── docs/                    # Project documentation - Technical documentation, guides, and architecture decisions
│   ├── architecture/        # Architecture documentation - System design, patterns, and technical decisions
│   ├── guides/             # Development guides - Setup, deployment, and development workflows
│   └── api/                # API documentation - API endpoints, schemas, and integration guides
├── lib/                     # Utility libraries - Shared utilities and helper functions
│   ├── hooks/               # React hooks - Custom React hooks for shared stateful logic
│   └── utils/               # Utility functions - Pure functions for data manipulation, formatting, etc.
├── middleware/              # Modular middleware components - Request/response interceptors
│   ├── auth.ts              # Authentication middleware - Handles user authentication and authorization
│   ├── logging.ts           # Logging middleware - Request/response logging and monitoring
│   ├── metrics.ts           # Metrics collection middleware - Performance and usage metrics
│   └── index.ts             # Re-exports middleware components - Central export point for middleware
├── middleware.ts            # Main Next.js middleware entry point - Orchestrates all middleware execution
├── services/                # External service integrations - Third-party service clients and configurations
│   └── supabase/            # Database and API integrations - Supabase client setup and database operations
├── types/                   # TypeScript type definitions - Shared TypeScript interfaces and types
├── tests/                   # Test files and test utilities
│   ├── unit/               # Unit tests for individual components and functions
│   ├── integration/        # Integration tests for component interactions
│   ├── e2e/               # End-to-end tests for complete user flows
│   └── utils/              # Test utilities, helpers, and mocks
└── public/                  # Static assets - Images, fonts, and other static files served directly
```

### 2. Path Configuration

Configure absolute imports in `tsconfig.json`:

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./*"]
    }
  }
}
```

### 3. Naming Conventions

1. **Directories**: Use kebab-case (e.g., `user-settings/`)
2. **Files**:
   - Regular files: Use kebab-case (e.g., `button.tsx`, `user-profile.tsx`)
   - Next.js special files: Use standard naming (e.g., `page.tsx`, `layout.tsx`)

### 4. Version Control

1. **Branch Naming**:

   - Feature: `feature/description`
   - Bug fix: `fix/description`
   - Hotfix: `hotfix/description`
   - Release: `release/v1.0.0`

2. **Commit Messages**:

   - Format: `type(scope): description`
   - Types: feat, fix, docs, style, refactor, test, chore

3. **Pull Requests**:
   - Max 400 lines changed
   - Include screenshots for UI changes
   - Link related issues
   - Add test coverage info

### 5. Component Organization

1. **Placement Rules**:

   - Page-specific: `/app/[page]/components/`
   - Layout: `/app/components/`
   - Shared: `/components/shared/`
   - UI primitives: `/components/ui/`

2. **Complex Component Structure**:

```
components/shared/data-table/
├── data-table.tsx         # Main component
├── data-table-header.tsx  # Sub-component
├── data-table-row.tsx     # Sub-component
└── index.ts               # Re-export the main component
```

### 6. State Management

1. **Client-Side State (Zustand)**:

   - Use Zustand for global client-side state management
   - Store structure:

   ```
   stores/
   ├── useAuthStore.ts     # Authentication state
   ├── useUIStore.ts       # UI state (modals, themes, etc.)
   └── index.ts            # Re-exports all stores
   ```

   - Store implementation pattern:

   ```typescript
   import { create } from "zustand";

   interface AuthState {
     user: User | null;
     isAuthenticated: boolean;
     login: (user: User) => void;
     logout: () => void;
   }

   export const useAuthStore = create<AuthState>((set) => ({
     user: null,
     isAuthenticated: false,
     login: (user) => set({ user, isAuthenticated: true }),
     logout: () => set({ user: null, isAuthenticated: false }),
   }));
   ```

2. **Server State (TanStack Query)**:

   - Use TanStack Query for all server state management
   - Query organization:

   ```
   lib/
   ├── queries/
   │   ├── users.ts        # User-related queries
   │   ├── products.ts     # Product-related queries
   │   └── index.ts        # Re-exports all queries
   ```

   - Query implementation pattern:

   ```typescript
   import {
     useQuery,
     useMutation,
     useQueryClient,
   } from "@tanstack/react-query";

   // Query hooks
   export const useUser = (userId: string) => {
     return useQuery({
       queryKey: ["user", userId],
       queryFn: () => fetchUser(userId),
     });
   };

   // Mutation hooks
   export const useUpdateUser = () => {
     const queryClient = useQueryClient();

     return useMutation({
       mutationFn: updateUser,
       onSuccess: (data) => {
         queryClient.invalidateQueries({ queryKey: ["user", data.id] });
       },
     });
   };
   ```

3. **State Management Rules**:

   - Use Zustand for:
     - UI state (modals, themes, etc.)
     - Form state (when complex)
     - User preferences
     - Navigation state
   - Use TanStack Query for:
     - API data fetching
     - Server state synchronization
     - Optimistic updates
     - Infinite scrolling
     - Real-time updates
   - Use React Context for:
     - Theme providers
     - Auth providers
     - Feature flags
   - Use Local State (useState) for:
     - Component-specific UI state
     - Simple form state
     - Temporary state

4. **Performance Considerations**:
   - Implement proper query keys for efficient caching
   - Use suspense mode for loading states
   - Implement proper error boundaries
   - Use optimistic updates for better UX
   - Implement proper prefetching strategies

### 7. Testing Strategy

1. **Organization**:

   - Unit: `__tests__/unit/`
   - Integration: `__tests__/integration/`
   - E2E: `__tests__/e2e/`
   - Utilities: `__tests__/utils/`

2. **Requirements**:
   - 80% minimum test coverage
   - React Testing Library for components
   - Playwright for E2E
   - Mock external services

### 8. Performance Standards

1. **Image Optimization**:

   - Use Next.js Image component
   - Implement responsive sizing
   - Use WebP with fallbacks
   - Lazy load below-fold images

2. **Code Splitting**:

   - Dynamic imports for large components
   - Route-based splitting
   - Lazy loading non-critical components
   - React.lazy() for components

3. **Caching**:

   - Proper cache headers
   - SWR/React Query for data
   - Stale-while-revalidate pattern
   - Service workers for offline

4. **Bundle Optimization**:
   - Monitor with @next/bundle-analyzer
   - Remove unused dependencies
   - Tree-shaking friendly imports
   - CSS optimization

### 9. Security Requirements

1. **Authentication**:

   - Secure session management
   - HTTP-only cookies
   - Rate limiting
   - CORS configuration

2. **Data Protection**:

   - Input sanitization
   - SQL injection prevention
   - Parameterized queries
   - Data encryption

3. **Dependencies**:
   - Regular updates
   - Security audits
   - CI/CD scanning
   - Lockfile usage

## Implementation

### Example Imports

```typescript
// ✅ Correct
import { Button } from "@/components/ui/button";
import { getUserById } from "@/lib/utils/users";
import { authService } from "@/services/supabase/auth";

// ❌ Incorrect
import { Button } from "../../../components/ui/button";
import { getUserById } from "../../lib/utils/users";
```

### Middleware Example

```typescript
// middleware/auth.ts
export function authMiddleware(req) {
  // Authentication logic
}

// middleware/index.ts
export { authMiddleware } from "./auth";
export { loggingMiddleware } from "./logging";
export { metricsMiddleware } from "./metrics";

// Root middleware.ts
import { NextResponse } from "next/server";
import {
  authMiddleware,
  loggingMiddleware,
  metricsMiddleware,
} from "./middleware";

export function middleware(request) {
  authMiddleware(request);
  loggingMiddleware(request);
  metricsMiddleware(request);
  return NextResponse.next();
}

export const config = {
  matcher: ["/dashboard/:path*"],
};
```
