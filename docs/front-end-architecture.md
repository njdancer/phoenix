# Phoenix Frontend Architecture Document

## Table of Contents

- [Introduction](#introduction)
- [Overall Frontend Philosophy & Patterns](#overall-frontend-philosophy--patterns)
- [Detailed Frontend Directory Structure](#detailed-frontend-directory-structure)
- [Component Breakdown & Implementation Details](#component-breakdown--implementation-details)
  - [Component Naming & Organization](#component-naming--organization)
  - [Template for Component Specification](#template-for-component-specification)
- [State Management In-Depth](#state-management-in-depth)
  - [Guiding Principles](#guiding-principles)
  - [Primary Use Cases for Zustand](#primary-use-cases-for-zustand)
  - [Re-evaluation During Development](#re-evaluation-during-development)
- [API Interaction Layer](#api-interaction-layer)
  - [Client/Service Structure](#clientservice-structure)
  - [Error Handling & Retries (Frontend)](#error-handling--retries-frontend)
- [Routing Strategy](#routing-strategy)
  - [Route Definitions](#route-definitions)
  - [Route Guards / Protection](#route-guards--protection)
- [Build, Bundling, and Deployment](#build-bundling-and-deployment)
  - [Build Process & Scripts](#build-process--scripts)
  - [Key Bundling Optimizations](#key-bundling-optimizations)
  - [Deployment](#deployment)
- [Frontend Testing Strategy](#frontend-testing-strategy)
  - [Component Testing (Unit & Integration)](#component-testing-unit--integration)
  - [UI Integration/Flow Testing (Route Level)](#ui-integrationflow-testing-route-level)
  - [End-to-End (E2E) UI Testing](#end-to-end-e2e-ui-testing)
  - [Storybook for Visual Testing & AX Checks](#storybook-for-visual-testing--ax-checks)
  - [Test Coverage Goals](#test-coverage-goals)
  - [Testing Tools & Libraries (Summary)](#testing-tools--libraries-summary)
- [Accessibility (AX) Implementation Plan](#accessibility-ax-implementation-plan)
- [Performance Considerations](#performance-considerations)
- [Security Considerations (Frontend)](#security-considerations-frontend)
- [Browser Compatibility & Progressive Enhancement](#browser-compatibility--progressive-enhancement)
- [Change Log](#change-log)

## Introduction

This document outlines the frontend architecture for the **Phoenix** project. It details the choices regarding frameworks, patterns, structure, and conventions that will guide the development of the user interface and client-side logic. The goal is to create a maintainable, scalable, performant, and accessible frontend application that aligns with the overall project objectives defined in the main architecture document and UI/UX specifications. The frontend is a Server-Side Rendered (SSR) React Router application running on Cloudflare Workers.

- **Project Name:** Phoenix
- **Link to Main Architecture Document:** `architecture.md` (This document provides the overarching system architecture, including backend services, data models, and technology stack choices that influence this frontend architecture.)
- **Link to UI/UX Specification:** (The content generated based on `front-end-spec-tmpl.txt` serves as this specification, detailing user flows, information architecture, and accessibility requirements.)
- **Link to Primary Design Files (Figma, Sketch, etc.):** Not applicable. Visual design and layout will be developed directly in code using Shadcn/UI and Tailwind CSS, guided by conceptual layouts and functional requirements in the UI/UX Specification.
- **Link to Deployed Storybook / Component Showcase:** {URL will be added here once Storybook (targeting v9) is deployed.}

## Overall Frontend Philosophy & Patterns

The frontend for Phoenix will be a modern, server-side rendered (SSR) React application, primarily serving as an admin and debug interface for the primary user.

- **Framework & Core Libraries:**
  - **React 19:** For building the user interface.
  - **React Router v7:** For routing, data loading, and data mutations, leveraging its SSR capabilities within the Cloudflare Worker environment.
  - **TypeScript:** For static typing, improving code quality and maintainability.
  - **Vite:** As the frontend build tool and development server, utilizing Rollup for production builds.
- **Component Architecture:** A pragmatic approach will be taken:
  - Leverage **Shadcn/UI** components as the primary UI toolkit. These are unstyled, composable components that will be styled with Tailwind CSS.
  - **Organization:**
    - Raw Shadcn/UI components: `apps/phoenix-app/app/lib/components/ui/`.
    - Customized/Extended UI Primitives: `apps/phoenix-app/app/components/ui/`.
    - Common Reusable Components: `apps/phoenix-app/app/components/common/`.
    - Feature-Specific Components: Within feature modules, e.g., `apps/phoenix-app/app/features/{featureName}/components/`.
  - Layouts will be handled contextually and as composable components (see Directory Structure and Routing).
  - Route components (views) will be organized within feature modules.
- **State Management Strategy:**
  - **Primary:** React Router v7 loaders (for data fetching) and actions (for data mutations) will manage server state and its flow into components.
  - **Secondary (Client-Side):** **Zustand** will be used for:
    - Minimal global client-side UI state (e.g., managing notifications, global modal states, UI theme/settings).
    - Complex non-global (local/feature) state that doesn't fit neatly into React Router's data flow or component state, evaluated on a case-by-case basis.
- **Data Flow:** Primarily unidirectional, driven by React Router's data loading and action handling mechanisms, and standard React prop-drilling/context patterns. Zustand state changes also follow a unidirectional flow.
- **Styling Approach:**
  - **Tailwind CSS v4:** For utility-first CSS styling.
  - **Shadcn/UI:** Configured to use CSS variables for theming.
  - Global styles and Tailwind configuration will reside in `apps/phoenix-app/app/styles/` and the project root (`apps/phoenix-app/`) respectively.
- **Key Design Patterns:**
  - **React Hooks:** For encapsulating reusable logic and stateful behavior.
  - **Custom Hooks:** For abstracting complex or shared logic.
  - **Composition:** Favoring component composition over inheritance.
- **Alignment with Overall System Architecture:**
  - The application is part of the `phoenix-app` Cloudflare Worker deployment, structured within an NX monorepo.
  - The worker (defined in `apps/phoenix-app/worker/`) handles SSR, API logic, and serves static assets generated by the frontend build.
  - API interactions are typically with the same worker.

## Detailed Frontend Directory Structure

The Phoenix application is structured within an NX monorepo. The `apps/phoenix-app/` directory contains the SSR React Router application, including its frontend UI code, worker logic, and build configurations.

```plaintext
/phoenix-workspace/                 # Root of your NX monorepo
├── apps/
│   └── phoenix-app/                # The Phoenix SSR React Router application
│       ├── app/                    # Core React application code (SSR and client components)
│       │   ├── components/         # UI components specific to Phoenix
│       │   │   ├── ui/             # Customized/extended UI primitives (wrappers for Shadcn)
│       │   │   │   └── Button.tsx  # Example: Phoenix-specific Button
│       │   │   ├── common/         # Common composite Phoenix components (e.g., BalanceCard)
│       │   │   └── features/       # Top-level directory for feature-specific complex components
│       │   │       └── {featureName}/
│       │   │           └── SomeComplexFeatureComponent.tsx
│       │   │
│       │   ├── features/         # Top-level feature modules (containing routes, views, feature-specific components)
│       │   │   └── transactions/
│       │   │       ├── components/ # Components specifically for the transactions feature
│       │   │       │   └── TransactionFilters.tsx
│       │   │       ├── views/      # Route components / pages for this feature
│       │   │       │   └── TransactionsPage.tsx
│       │   │       ├── transactions.routes.ts # Route definitions for this feature
│       │   │       └── services/   # Optional: feature-specific client-side API logic
│       │   │       └── store/      # Optional: feature-specific Zustand store
│       │   │
│       │   ├── hooks/            # Custom React hooks (e.g., useBreakpoints.ts)
│       │   │
│       │   ├── lib/                # Copied third-party code, primarily Shadcn/UI
│       │   │   ├── components/
│       │   │   │   └── ui/         # Raw Shadcn/UI components (e.g., button.tsx, card.tsx)
│       │   │   └── utils.ts        # Shadcn/UI utils (e.g., cn function)
│       │   │
│       │   ├── services/           # Global API client or utility services (e.g., apiClient.ts)
│       │   │
│       │   ├── store/              # Global Zustand store slices (e.g., uiStore.ts)
│       │   │   └── uiStore.ts
│       │   │
│       │   ├── styles/             # Global styles and main CSS file for Tailwind
│       │   │   └── app.css         # Main CSS file (includes Tailwind directives)
│       │   │
│       │   ├── types/              # App-specific TypeScript definitions (frontend internal)
│       │   │   └── index.ts
│       │   │
│       │   ├── router.config.ts    # Centralized application router configuration (composes feature routes)
│       │   ├── entry.client.tsx    # CLIENT-SIDE entry point (for hydration)
│       │   ├── entry.server.tsx    # SERVER-SIDE entry point (for SSR in worker)
│       │   └── root.tsx            # Root React component (HTML shell, <Outlet />, global providers)
│       │                           # {NOTE: The organization of layouts will be contextual. `root.tsx` is the primary layout. Other shared layout components might live in `app/components/common/layouts/` or feature-specific layouts within feature directories. This will be assessed during development.}
│       │
│       ├── public/                 # Static assets (favicon, images, manifest.json)
│       │   └── favicon.ico
│       │
│       ├── worker/                 # Cloudflare Worker script & API logic
│       │   ├── index.ts            # Main Worker entry point (imports app/entry.server.tsx, handles API & SSR)
│       │   ├── api/                # API route handlers
│       │   │   ├── v1/
│       │   │   │   ├── transactions.ts
│       │   │   │   └── settings.ts
│       │   │   └── health.ts
│       │   └── auth.server.ts      # Server-side authentication middleware/handlers
│       │
│       ├── storybook/              # Storybook v9 configuration and stories
│       │   ├── main.ts
│       │   ├── preview.ts
│       │   └── (stories mirroring app/components structure, or co-located)
│       │
│       ├── .env                    # Environment variables (VITE_ prefix for client, direct for worker via .dev.vars)
│       ├── .dev.vars               # Local development worker variables/secrets (git-ignored)
│       ├── .eslintrc.cjs
│       ├── .gitignore
│       ├── package.json            # Dependencies for phoenix-app (React, Vite, Tailwind, etc.)
│       ├── postcss.config.js
│       ├── tailwind.config.js      # Tailwind configuration for phoenix-app
│       ├── tsconfig.json           # Base TSConfig for phoenix-app client-side code & Vite
│       ├── tsconfig.worker.json    # TSConfig specific for worker code (if different needs)
│       ├── vite.config.ts          # Vite config (handles client & server builds for SSR)
│       ├── worker-configuration.d.ts # For Worker environment types
│       └── wrangler.jsonc          # Wrangler configuration for deployment
│
├── libs/                           # Shared libraries (e.g., types, utils) for the monorepo
│   └── shared-types/
│       ├── src/
│       │   └── index.ts            # e.g., export * from './api-interfaces';
│       ├── package.json
│       └── tsconfig.json
│
├── nx.json                         # NX workspace configuration
├── tsconfig.base.json              # Base TypeScript config for the NX workspace
└── package.json                    # Root package.json (for NX, workspace-level dependencies/scripts)
```

## Component Breakdown & Implementation Details

### Component Naming & Organization

- **Naming Conventions:**
  - Component files and exported components: `PascalCase` (e.g., `TransactionTable.tsx`).
  - Feature directories: `lowercase` (e.g., `transactions`).
- **Organizational Principles:** As detailed in the directory structure, prioritizing feature-based organization and clear separation for Shadcn/UI primitives, custom UI elements, common components, and feature-specific components. Tests and stories will be co-located with components.

### Template for Component Specification

(For complex or critical components, particularly in `app/components/common/` or feature modules. Can be a README or comments.)

```markdown
---
**Component Name:** {ComponentName}
**Purpose:** {Description}
**Source File(s):** `apps/phoenix-app/app/{path}/{ComponentName}.tsx`
**Visual Reference:** {Storybook link, conceptual ASCII art reference}
---

**Props:** {Table: Name | Type | Required | Default | Description}
**Internal State (if significant):** {Table: Variable | Type | Initial | Description}
**Key Behaviors & Interactions:** {List}
**Data Fetching / Side Effects:** {Description or N/A if data from loaders/props}
**Accessibility Notes:** {ARIA attributes, keyboard patterns}

---
```

## State Management In-Depth

Phoenix will primarily use React Router v7's `loader` and `action` system for server state. Zustand will be used for specific client-side global UI state and complex local/feature state on a case-by-case basis.

### Guiding Principles

1.  **React Router's Data Lifecycle First:** `loader`s for data fetching, `action`s for mutations. This includes global data like auth status via a root loader.
2.  **Zustand as a Targeted Tool:** For state not fitting React Router's flow.
3.  **Flexibility and Case-by-Case Evaluation:** Re-evaluate state choices during development.

### Primary Use Cases for Zustand

1.  **Purely Client-Side Global UI State:**
    - Examples: UI theme (persisted to `localStorage`), global notifications/toasts, ephemeral global UI toggles.
    - `app/store/uiStore.ts` will house such state. (Example provided previously with `persist` middleware for theme).
2.  **Complex Non-Global (Local/Feature-Specific) State:**
    - Where `useState`/`useReducer` + Context is unwieldy within a feature or component tree. A co-located Zustand store can be created within the feature's directory.

### Re-evaluation During Development

The team will default to React Router's data handling and introduce Zustand pragmatically, documenting decisions for uses beyond the above.

## API Interaction Layer

Interaction with the Cloudflare Worker backend occurs mainly via React Router `loader`s/`action`s. A custom `apiClient` handles other cases.

### Client/Service Structure

1.  **React Router `Workspace` (Primary):** For `loader` and `action` functions.
2.  **Custom API Client (`apps/phoenix-app/app/services/apiClient.ts`) (Secondary):**
    - Wrapper around `Workspace` for non-router client-side calls (e.g., from Zustand actions, event handlers).
    - Handles common headers, JSON parsing, and error shaping (includes `ApiError` class example provided previously).
3.  **Specific Service Definitions:** Minimal, as React Router actions are preferred for mutations. Feature-specific client-side services in `app/features/{featureName}/services/` if complex non-router API logic is needed.

### Error Handling & Retries (Frontend)

- **React Router `errorElement`:** Primary for loader/action errors.
- **`apiClient.ts` Errors:** Handled with `try/catch`, using `ApiError`. Display errors inline or via global `uiStore` notifications.
- **Retries:** Avoided for mutations. Minimal for `GET`s, on a case-by-case basis. React Router loaders typically require manual re-initiation (e.g., refresh, `router.revalidate()`).
- **Server Error Response Structure:** Worker API/actions should return structured JSON errors.

## Routing Strategy

Utilizes React Router v7 for SSR, with a feature-based organization.

### Route Definitions

1.  **Framework:** React Router v7.
2.  **Routing Convention - Feature-Based Organization:**
    - Routes defined as JavaScript objects.
    - Features (e.g., `app/features/transactions/`) define their own route arrays (e.g., in `transactions.routes.ts`).
    - These are composed into a main application router configuration (e.g., `app/router.config.ts`).
3.  **Layouts as Composable & Abstract Concepts:**
    - `app/root.tsx` is the primary HTML shell.
    - Structural layout components are composable, residing in `app/components/common/` or within feature directories if specific.
4.  **Key Application Features & Corresponding Routes (Conceptual):**
    - Authentication (e.g., `/login`, Passkey routes/actions).
    - Main Dashboard Area (e.g., `/dashboard`).
    - Transactions Management (e.g., `/dashboard/transactions`).
    - Cleared Balance Display (Integrated into relevant views, not a separate "calculator" page).
    - Application Settings (e.g., `/dashboard/settings`).
    - Debugging Utilities (e.g., `/dashboard/debug`).
5.  **`errorElement`:** For route-level error display.

### Route Guards / Protection

1.  **Primary: Server-Side Checks & React Router Middleware (Experimental):**
    - Auth logic in worker (e.g., `worker/auth.server.ts`).
    - Explore React Router's experimental middleware (future flag `future.v7_reactRouterMiddleware`) for guards.
    - Fallback: Auth checks in each `loader`/`action`, throwing `redirect`.
2.  **RBAC:** Future extension of the same mechanisms.
3.  **Client-Side UI Indicators:** UX enhancement only.
4.  **Passkey Authentication Routes:** Handled within React Router `action`s.

## Build, Bundling, and Deployment

### Build Process & Scripts

- **Development Server:** Vite (`vite dev`) with `wrangler dev` for SSR.
- **Production Build Tool:** Vite (using Rollup).
- **Build Output:** Client-side static assets; Cloudflare Worker JS bundle.
- **Key Scripts (Conceptual, orchestrated by NX or in `apps/phoenix-app/package.json`):** `dev`, `build`, `preview`, `lint`, `format`, `test`, `storybook:dev`, `storybook:build`.
- **Environment Variables & Configuration (Cloudflare Workers):**
  - Runtime: `env` object in worker (variables/secrets/bindings from `wrangler.jsonc` or dashboard).
  - Local Dev: `wrangler.jsonc` and `.dev.vars` file.
  - Client-Side: Explicitly passed from server or build-time embedded via Vite (`import.meta.env.VITE_...` for non-sensitive config).

### Key Bundling Optimizations (via Vite/Rollup)

- Code Splitting, Tree Shaking, Minification, Asset Hashing, Lazy Loading (`React.lazy()`, route components).
- Image Optimization: Future consideration.

### Deployment

- **Platform:** Cloudflare Workers (SSR, API, static asset serving from worker or R2).
- **Process:** CI/CD (GitHub Actions) using `wrangler deploy`.
- **Environment Management:** Wrangler environments in `wrangler.jsonc`.
- **Cache Control:** Optimal HTTP cache headers for static assets.

## Frontend Testing Strategy

Vitest is the primary framework.

- **Link to Main Overall Testing Strategy:** Refer to `architecture.md` and PRD NFRs.

### 1\. Component Testing (Unit & Integration)

- **Scope:** Individual UI components, small groups of components.
- **Tools:** Vitest, React Testing Library.
- **Focus:** Rendering, props, interactions, conditional logic, basic AX, events. Sparingly use snapshots.
- **Location:** Co-located test files (`*.test.tsx`).
- **Mocking:** Vitest mocks, Zustand store mocking, React Router context mocking.

### 2\. UI Integration/Flow Testing (Route Level)

- **Scope:** Client-side user flows across components/routes.
- **Tools:** Vitest, React Testing Library, MSW for API mocking.
- **Focus:** Navigation, form submissions (mocked actions), data display (mocked loaders).

### 3\. End-to-End (E2E) UI Testing

- **Scope (MVP Focus):** Minimal (1-2 critical flows).
- **Tools:** Vitest Browser Mode (MVP). Playwright/Cypress (Future).
- **Test Data Management:** Mocked APIs or test accounts.

### 4\. Storybook for Visual Testing & AX Checks

- **Visual Testing:** Storybook for manual visual regression (future automation).
- **Accessibility Testing:** `@storybook/addon-a11y` (Axe-core).

### 5\. Test Coverage Goals

- No strict % for MVP. Focus on critical components, core logic, key flows.

### 6\. Testing Tools & Libraries (Summary)

- Vitest, React Testing Library, MSW, Storybook.

## Accessibility (AX) Implementation Plan

Goal: WCAG 2.1 Level AA where feasible.

1.  **Semantic HTML.**
2.  **ARIA Implementation:** Leverage Shadcn/UI; custom ARIA via WAI-ARIA APG.
3.  **Keyboard Navigation & Focus Management:** All interactive elements keyboard accessible; logical focus order; visible focus; focus trapping in modals.
4.  **Forms & Inputs:** Associated labels; errors linked via `aria-describedby`/`aria-invalid`.
5.  **Color Contrast:** Meet WCAG AA.
6.  **Content & Structure:** Correct heading use; text alternatives for icons.
7.  **Testing & Validation:** `eslint-plugin-jsx-a11y`, Storybook/Axe, browser dev tools, manual keyboard/screen reader checks.

## Performance Considerations

1.  **Bundle Size & Code Splitting:** Vite/Rollup; route splitting; `React.lazy()`.
2.  **Minimizing Re-renders:** `React.memo`, `useCallback`, `useMemo`; optimized Zustand selectors.
3.  **Data Fetching:** React Router loaders.
4.  **List Virtualization:** Consider for very long lists (pagination primary).
5.  **Debouncing/Throttling:** For rapid-fire events if performance issues arise.
6.  **Styling Performance:** Tailwind CSS (purged).
7.  **Image Optimization:** Future strategy.
8.  **Monitoring & Profiling:** Browser DevTools, React DevTools, Lighthouse.

## Security Considerations (Frontend)

1.  **XSS Prevention:** React auto-escaping; avoid `dangerouslySetInnerHTML`.
2.  **Data Handling:** Treat API data as data. Sanitize any user-HTML input (unlikely).
3.  **Content Security Policy (CSP):** Via HTTP headers from Worker.
4.  **Authentication Token Management:** Primary: HTTP-only cookies via Worker.
5.  **API Interaction Security:** HTTPS. Client-side validation for UX; server-side authoritative.
6.  **Third-Party Libraries:** Reputable; keep updated (`npm audit`).
7.  **Secure Headers (from Worker):** HSTS, X-Content-Type-Options, X-Frame-Options, etc.
8.  **Development Practices:** Least privilege; avoid leaking sensitive info in client errors.

## Browser Compatibility & Progressive Enhancement

1.  **Target Browsers:** Latest 2 stable desktop (Chrome, Firefox, Safari, Edge); latest mobile (iOS Safari, Android Chrome). No IE.
2.  **JavaScript Requirement & Progressive Enhancement:**
    - SSR ensures base content/navigation works without JS.
    - Client-side JS enhances interactivity, state, forms.
    - Consider `<noscript>` content for JS-critical features. Goal: graceful degradation.
3.  **CSS Compatibility:** Vite/PostCSS/Autoprefixer. Tailwind uses modern CSS.
4.  **Polyfill Strategy:** Vite targets modern browsers; minimal polyfills expected.
5.  **Responsive Design:** Via Tailwind CSS for desktop/mobile.

## Change Log

| Date       | Version | Description                                                       | Author  |
| ---------- | ------- | ----------------------------------------------------------------- | ------- |
| 2025-05-17 | 1.0.0   | Initial comprehensive draft of architecture.                      | AI/User |
| 2025-05-17 | 1.1.0   | Revisions to directory structure, routing, state management, etc. | AI/User |
